# Auto-update — updater DESACOPLADO, versão coerente, e a armadilha do git-dep

> O software local **tem de se atualizar sozinho** (o usuário leigo não vai baixar release à mão —
> `prever macacos`). Mas o update é a operação mais perigosa que um app faz: **troca o próprio
> binário em uso**. Por isso dois pisos: **o updater é DESACOPLADO do app**, e **a versão que o
> binário reporta bate com o que ele É** — a armadilha do git-dep pinado quebra exatamente isso.

## 1. Updater DESACOPLADO do app (piso)

O updater é um **componente separado** — crate/binário próprio, como o **`schematize-updater`** da
casa (repo `schematizeme/schematize-updater`, bootstrapper cross-OS híbrido binário|fonte). Motivo:

- **App quebrado não pode travar o update.** Se a versão instalada tem um bug que crasha no boot, o
  updater (processo separado) ainda roda e **conserta puxando a próxima versão**. Se o updater
  vivesse dentro do app quebrado, o usuário ficaria preso numa versão que nem abre.
- **Update quebrado não pode derrubar o app.** Uma falha no download/troca não corrompe a instalação
  boa: o updater valida **antes** de substituir e faz **rollback** se a nova versão não sobe.
- **Fronteira clara:** o app pede "verifica update" e delega; o updater baixa, **verifica assinatura/
  hash**, troca o binário, mata o antigo e re-executa. O app não implementa a mecânica de troca no
  próprio processo que está sendo trocado.

## 2. A ARMADILHA REAL — git-dep pinado no `Cargo.lock` embute versão VELHA

Anti-padrão concreto que a casa **acabou de corrigir**. Quando o app depende de um crate por **git**
(ex.: o updater ou uma lib compartilhada apontada por `git = "..."` no `Cargo.toml`), o
**`Cargo.lock` pina o commit exato**. Resultado: **por mais que a branch remota avance, o build
recompila contra o commit VELHO travado no lock** — e o binário **embute versão/comportamento
antigo**, mesmo você "tendo atualizado o código". O `--version` mente, o comportamento é o de ontem,
e ninguém entende por quê.

**Regra dura:** **antes de compilar um release, o build tem de `cargo update -p <dep>`** (atualizar
o lock pro commit novo do git-dep). No pipeline:

```bash
# ANTES de `cargo build --release`, atualize os git-deps pinados no lock:
cargo update -p schematize-updater   # e cada git-dep cujo commit avançou
cargo build --release
```

- **Piso:** **a versão embutida no binário SEMPRE coerente** com o código que ele deveria ser. Se há
  git-dep, o `cargo update -p` é **parte do build de release**, não um passo opcional que se esquece.
- **Prova:** o CI roda `cargo update -p <dep>` e depois um teste que compara `binário --version` /
  hash do git-dep com o commit esperado da branch. Divergiu → build falha (não publica versão que
  mente).
- **Generalização cross-linguagem:** o mesmo vale pra qualquer lockfile que pina fonte VCS
  (`package-lock`/`go.sum` com replace de git, etc.) — lock que pina git reproduz *demais*.

## 3. Trocar o binário em uso + MATAR o processo antigo + re-exec

Trocar o binário que está **rodando** tem mecânica por SO — e um erro clássico: **só fechar a janela
não mata o processo**.

- **Linux/macOS:** dá pra substituir o arquivo do binário enquanto o processo antigo roda (o inode
  aberto continua válido); o **novo** só vale no próximo start. Estratégia: baixa pra arquivo
  temporário → valida → `rename` atômico por cima → **re-exec** (`execv`) pro novo binário.
- **Windows:** o executável em uso fica **travado** — não dá pra sobrescrever direto. Padrão: baixa
  pro lado, agenda a troca via processo auxiliar/`MoveFileEx` no reboot, ou o updater (processo
  separado) troca **depois** que o app saiu.
- **MATAR o processo antigo, não só fechar a janela.** Bug real: fechar a janela deixa o **processo/
  tray/daemon** vivo, segurando o binário antigo e a porta/lock de single-instance
  (`privilegio-fs.md`). Pós-update, o updater **encerra o processo antigo de verdade** (sinal +
  espera + kill se preciso) e só então **re-exec** a versão nova. Se o app tem tray/serviço, encerre
  os dois.
- **Re-exec limpo:** a nova instância sobe reaproveitando o estado necessário (arquivo, não memória
  do processo morto), e o single-instance lock passa do processo velho pro novo sem deixar órfão.

## 4. Versionamento, pin e rollback

- **SemVer + canal:** versão em SemVer; canais (`stable`/`beta`) se houver. O updater consulta a
  **Release/tag** do repo (a casa publica em `skill-<slug>`/repos de release) e compara.
- **Pin:** o usuário/admin pode **fixar** uma versão (não auto-subir) — respeite o pin; útil em
  ambiente controlado.
- **Rollback:** guarde a **versão anterior** (ou saiba baixá-la) pra **reverter** se a nova falhar o
  self-check pós-update. Rollback é caminho previsto, não heroísmo manual.
- **Self-check pós-update:** depois de trocar+re-exec, a nova versão roda um **smoke** (abre? conecta
  o mínimo? `--version` coerente?). Falhou → **rollback automático** pra anterior + sinal, nunca
  deixa o usuário numa versão que não abre.

## 5. Segurança do canal de update

- **Verifique assinatura/hash do artefato** antes de instalar (senão o update é vetor de RCE — um
  MITM entrega binário malicioso). Chave pública embutida pra verificar; **chave privada em cofre**.
- **HTTPS + pin do host** de release; nada de update por canal não autenticado.
- Casa com `privacidade-seguranca.md` (assinatura de binário) e com a `schematize-pentest` (update
  sequestrável é achado de 1ª linha).

## 6. DoD de auto-update

- [ ] updater é **processo/binário separado** do app (app quebrado ainda atualiza);
- [ ] build de release roda **`cargo update -p <git-dep>`** antes de compilar; CI **prova** que a
      versão embutida bate com o commit da branch (sem git-dep mentiroso);
- [ ] troca de binário testada **nos 3 SOs** (incl. Windows travado); processo antigo é **morto**
      (não só janela fechada) e há **re-exec**;
- [ ] **pin** respeitado, **rollback** automático em falha de self-check pós-update;
- [ ] artefato de update **verificado (assinatura/hash)** antes de instalar;
- [ ] **teste de update ponta-a-ponta** (vN → vN+1 → rollback) no CI (`testes-locais.md`).
