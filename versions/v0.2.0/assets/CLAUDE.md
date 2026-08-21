<!-- cross-skill: offline-sync.md -> schematize-mobile -->
# CLAUDE.md — Engenharia de Software Local/Desktop da Casa (sempre on)

> Copie para a **raiz do repositório** do app e ajuste `<project>`. Fica pinado no contexto de toda
> tarefa e garante o piso mesmo quando a skill `schematize-desktop` não dispara sozinha. Em repo
> multi-skill (ex.: monorepo GUI + backend), use **junto** com os `CLAUDE.md` das skills de
> engenharia e da linguagem (rode `/desktop-claude` que mescla, sem sobrescrever os outros blocos).

## Regra mestre

App desktop/local é software da casa como qualquer outro: **o piso — segurança, IAM, testes, ops,
DoD (§35), archive (§28), índice (§39) — é o MESMO**. O desktop só muda o **como**, sob três
verdades: **não há servidor pra segurar a barra** (o binário roda na máquina do adversário), **cada
SO é diferente**, e **o usuário é leigo e sem suporte**. Em conflito entre "é só um app local,
relaxa" e este piso, **o piso vence**. Consulte o reference antes de agir — não trabalhe de memória.

## Pisos inegociáveis (VETADO — sem exceção)

1. **UX de massa — "prever o usuário leigo" ("prever macacos").** O software **se ADAPTA e NUNCA
   culpa o usuário**. Rodou como **root/sudo** → descobre o **usuário real** e grava **pra ele**
   (nunca em `/root`, com dono correto). Aberto pelo menu com **PATH mínimo** → **resolve** binários
   por fallback dirs + caminho absoluto. **Toolchain/dependência ausente** → instala/configura.
   Fechou/reordenou inesperado → previsto. Edge case que um leigo atinge = **BUG do software**.
   Mensagem **acionável e sem culpa** ("faço X pra você" > "você fez errado").
2. **UM só toolkit de GUI por produto.** Misturar dois (ex.: egui + Slint) é a causa-raiz do
   **"fantasma de layout"** e do launcher que abre o binário errado. Viés da casa: **Slint**; **egui
   não como principal**. Segundo toolkit só atrás de fronteira de processo + **ADR de exceção**.
3. **Updater DESACOPLADO + versão embutida sempre coerente.** O updater é **componente separado**
   (app quebrado não trava o update, e vice-versa). Armadilha real: **git-dep pinado no `Cargo.lock`
   faz o binário embutir versão VELHA** → o build roda **`cargo update -p <dep>` antes de compilar**,
   e o CI **prova** que a versão embutida bate com o commit. Pós-update: **troca o binário em uso,
   MATA o processo antigo** (só fechar a janela não basta) e **re-exec**. Suporta **pin/rollback**.
4. **Launcher com path ABSOLUTO.** O `.desktop` `Exec=` (e atalho macOS/Windows) aponta pro binário
   por **caminho absoluto** — o `PATH` do DE não tem `~/.cargo/bin`/`~/.local/bin`. Ao spawnar
   subprocessos, resolva por **fallback dirs + caminho absoluto**, nunca só o nome no PATH.
5. **NUNCA segredo embutido no cliente.** API key privada, `client_secret`, chave de assinatura,
   credencial — nada no binário/recurso/config distribuído/string compilada. Extrair o binário é
   trivial. Segredo mora no **servidor** (BFF); o app é **public client**. Chave de assinatura de
   release em **cofre/CI**. Segredo do usuário em **keychain do SO**.
6. **Privilégio mínimo + privacidade por padrão.** Eleve **só onde precisa** (nunca a GUI inteira
   como root); grave nos **dirs certos por SO** (XDG/`~/Library`/`%APPDATA%`) do usuário real. **Sem
   telemetria por padrão** — se houver, **opt-in explícito**, sem PII, local-first.
7. **Cross-OS é requisito, não "depois".** Path/processo/launcher/sinais por API do SO; **Wayland E
   X11** no Linux; **fontes cobrem CJK/árabe/emoji** (senão tofu). Um SO sem **smoke de
   empacotamento** naquele SO **não** está suportado.
8. **Estado do usuário é sagrado.** Schema local **versionado**, migração **reversível**, **backup
   antes de operação destrutiva**, escrita **atômica**. Perder o dado do usuário num update é o pior
   pecado. Se há sync, herda `offline-sync.md` (outbox durável, conflito explícito, servidor
   autoritativo).

## Como se decide aqui

- **Toolkit (`/desktop-toolkit`, `toolkits-gui.md`):** um só por produto, Slint default, egui não
  principal, web-in-native com justificativa; domínio testável sem abrir janela.
- **Empacotamento (`/desktop-package`, `empacotamento.md`):** formato por SO, assinatura/
  notarization, binário pré-compilado vs fonte (híbrido), smoke por SO.
- **Auto-update (`/desktop-update`, `auto-update.md`):** desacoplado, git-dep (`cargo update -p`),
  matar processo antigo + re-exec, pin/rollback, canal verificado.
- **Privilégio (`/desktop-privilege`, `privilegio-fs.md`):** root→usuário real, elevar só o pontual,
  launcher absoluto, single-instance, IPC validado.

## Relação com as outras skills

- **schematize-engineering** — a base: UX de massa (prever macacos), segurança, IAM (`iam.md`), DoD
  (§35), archive (§28), índice (§39).
- **schematize-rust** — Slint/Tauri em Rust, o `schematize-updater`, pisos de código.
- **schematize-mobile / schematize-web** — as irmãs: mesmo piso, cliente diferente; "segredo nunca
  no cliente" nos três.
- **schematize-pentest** — o app local é superfície de ataque (segredo no binário, update
  sequestrável, escalada por privilégio, path traversal em IPC/deep-link).

## Gestão de contexto (sessões longas)

Ao se aproximar do teto de contexto: **PARE e** gere o handoff em `<project>_archive/context/`
(estado + FEITO vs EM ABERTO) **antes** de compactar (`/desktop-cc`).
