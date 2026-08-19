# Empacotamento e distribuição cross-OS

> O binário É o produto: chega na máquina do usuário como **um pacote instalável do SO**, assinado
> onde a plataforma exige, e **abre no primeiro clique**. Um SO "suportado" sem pacote testado
> naquele SO **não** está suportado (`testes-locais.md`, smoke de empacotamento). Toda decisão de
> formato/assinatura vira **ADR (§27)** e o pipeline mora no **`<projeto>_ops`** (`ops.md`) — build
> assinado no **CI**, nunca à mão.

## 1. Formato por SO

| SO | Formatos sancionados | Notas |
|---|---|---|
| **Linux** | **AppImage** (portátil, roda em qualquer distro), **Flatpak** (sandbox + Flathub), **.deb/.rpm** (integração nativa por distro) | AppImage é o "baixou e rodou"; Flatpak dá sandbox e loja; .deb/.rpm pra quem quer no gerenciador de pacotes. Ofereça **≥1 portátil (AppImage) + ≥1 nativo** conforme o público. |
| **macOS** | **.dmg** (ou .pkg) com app **assinado (Developer ID) + notarizado + stapled** | Sem notarization o Gatekeeper **bloqueia** ("app está danificado/não pode ser aberto"). Notarize no CI; **staple** o ticket no `.app`/`.dmg`. |
| **Windows** | **MSI** (ou setup) ou **MSIX**, com binário **assinado (Authenticode)** | Sem assinatura, o SmartScreen assusta o usuário ("editor desconhecido"). Assine com cert em cofre. MSIX pra loja/atualização gerenciada; MSI/instalador clássico pra distribuição direta. |

Pisos de empacotamento:

- **Assinatura/notarization não é opcional** onde a plataforma cobra (macOS e Windows). "Funciona
  aqui" ≠ "abre na máquina do usuário sem susto". A **chave de assinatura vive em cofre/CI**
  (`privacidade-seguranca.md`), **nunca no repo**.
- **Ícone, nome, `.desktop`/bundle/atalho corretos** — parte do pacote, não enfeite. No Linux o
  `.desktop` tem `Exec=` com **path ABSOLUTO** (`privilegio-fs.md`, piso da casa — o PATH do DE não
  acha o binário).
- **Metadados de versão coerentes** no pacote e no binário (`--version` bate com a tag). Ligado ao
  piso do updater (`auto-update.md`): git-dep pinado desalinha a versão embutida.

## 2. Binário pré-compilado por SO vs compilar-do-fonte — o modelo HÍBRIDO

A casa resolve o dilema com o **modelo híbrido do `schematize-updater`**: **binário pré-compilado
quando existe pro SO/arquitetura do usuário; fallback pra compilar-do-fonte quando não existe.**

- **Binário pré-compilado (default):** o **CI compila por SO/arquitetura** (Linux x86_64/aarch64,
  macOS arm64/x86_64, Windows x86_64) e publica os artefatos assinados na **Release** do GitHub. O
  usuário baixa e roda — **não precisa de toolchain**. É o caminho do `prever macacos`: não exija
  que o usuário saiba de `rustup`/compilador.
- **Compilar-do-fonte (fallback):** quando não há binário pro alvo do usuário (arquitetura exótica,
  SO fora da matriz), o bootstrapper **detecta a ausência**, **instala/configura o toolchain
  sozinho** (não manda o usuário fazer) e compila. É fallback, não o caminho normal.
- **Cross-OS no CI, não na mão:** publique a **matriz** de SOs num pipeline (o `schematize-updater`
  tem CI que publica binários dos 3 SOs). Compilar localmente "pro meu SO" e distribuir é o
  anti-padrão que deixa 2 dos 3 SOs quebrados.

## 3. Reprodutibilidade e supply chain

- **Build reprodutível/travado:** `Cargo.lock`/lockfile commitado (o app **é** reproduzível), deps
  com versão/licença/hash verificados (`cadeia-suprimentos.md`; typosquatting de crate nativo é
  real). **Atenção ao git-dep pinado** — ele reproduz *demais* (versão velha): ver `auto-update.md`.
- **Sem baixar-e-executar em runtime** sem verificação: se o app puxa um componente pós-install,
  **verifique assinatura/hash** antes de executar (senão é vetor de RCE).
- **SBOM/lista de deps nativas** no archive quando o projeto exige rastreabilidade.

## 4. Instalação que "prever macacos"

- **Instalador idempotente:** reexecutar não corrompe; detecta instalação anterior e **atualiza no
  lugar** (não duplica; não deixa duas versões que confundem o launcher — ver
  `gui-launcher-abs-path`).
- **Rodou como root/sudo?** Instale/grave **pra o usuário real**, nunca só em `/root`
  (`privilegio-fs.md`). Detecte o usuário sob `sudo` (`SUDO_USER`) e faça o certo.
- **Falta dependência de sistema?** (lib gráfica, webview) — **detecte e oriente com ação** ("instalo
  X pra você" onde possível), nunca um crash cru com stack trace.
- **Desinstalação limpa:** remova binário, atalho, e — se o usuário pedir — dados; **preserve dados
  por padrão** (nunca apague estado do usuário sem consentimento; casa com backup em
  `offline-estado.md`).

## 5. DoD de empacotamento

Antes de dizer "distribuível", prove por SO alvo:

- [ ] pacote **gerado no CI** (não na máquina do dev) pra cada SO/arquitetura suportado;
- [ ] **assinado/notarizado** onde a plataforma exige (macOS notarizado+stapled; Windows
      Authenticode); chave de assinatura em **cofre/CI**;
- [ ] **smoke test de empacotamento**: instala num ambiente limpo daquele SO e **abre** (`testes-locais.md`);
- [ ] `.desktop`/bundle/atalho com **path absoluto** e ícone/nome corretos;
- [ ] versão do pacote **bate** com `--version` do binário (coerência de versão, `auto-update.md`);
- [ ] instalação **idempotente** e desinstalação **preserva dados** do usuário.

Item sem prova **não** conta como feito (§35).
