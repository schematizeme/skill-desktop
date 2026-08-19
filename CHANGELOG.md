# Changelog — schematize-desktop

Formato: [Keep a Changelog](https://keepachangelog.com/pt-BR/). Versionamento semântico.

## [0.1.0] — 2026-08-18

Primeira versão da skill de **engenharia de software local/nativo/desktop** da casa — o mesmo piso
da `schematize-engineering` (segurança, IAM, testes, ops, DoD §35, archive §28, índice §39) aplicado
ao software que roda **na máquina do usuário**, sob três verdades novas: **não há servidor pra
segurar a barra**, **cada SO é diferente** e **o usuário é leigo e sem suporte**. Irmã de
`schematize-mobile` e `schematize-web`; muda o "como", não o "o quê".

### Adicionado
- **SKILL.md** com 8 pisos inegociáveis (o piso da casa é o mesmo; **UX de massa — prever macacos**;
  **um só toolkit de GUI por produto**; **updater desacoplado + versão embutida coerente**;
  **launcher com path absoluto**; **nunca segredo no cliente**; privilégio mínimo + privacidade por
  padrão; cross-OS é requisito) + mapa de references + relação com engineering/rust/mobile/web/
  pentest/audit.
- **references/**:
  - `toolkits-gui.md` — rol de toolkits (Slint default, Tauri por fit/justificativa, **egui não como
    principal** — fantasma de layout, GTK/Qt), **um produto = um toolkit** (o pecado capital de
    misturar dois), web-in-native (Tauri>Electron, ponte validada), arquitetura em camadas com
    domínio sem toolkit.
  - `empacotamento.md` — formato por SO (AppImage/Flatpak/.deb/.rpm, .dmg+notarization,
    MSI/MSIX+Authenticode), **binário pré-compilado por SO vs compilar-do-fonte (híbrido**, modelo
    `schematize-updater`**)**, build assinado no CI, instalação idempotente que prever macacos, smoke
    de empacotamento.
  - `auto-update.md` — updater **DESACOPLADO** do app, a armadilha real do **git-dep pinado no
    `Cargo.lock`** (embute versão VELHA → `cargo update -p <dep>` antes de compilar; CI prova
    coerência), **trocar binário em uso** por SO, **matar processo antigo (não só fechar janela) +
    re-exec**, pin/rollback/self-check, canal verificado (assinatura/hash).
  - `offline-estado.md` — dirs por SO (XDG/`~/Library`/`%APPDATA%`), **estado ≠ cache**, **migração
    de schema local reversível/versionada**, backup automático + escrita atômica, export/import, sync
    eventual (herda `offline-sync.md`).
  - `privilegio-fs.md` — **root/sudo → usuário real** (nunca `/root`, `chown` correto), **elevar só
    onde precisa**, **launcher com `Exec` absoluto** + spawn por fallback dirs (o PATH do DE),
    **single-instance** por lock/socket robusto a crash e a órfão pós-update, **IPC local** validado
    e entrada externa canonicalizada.
  - `cross-os.md` — path/permissão/processo/launcher/sinais por SO (sem `fork` no Windows),
    **Wayland E X11**, HiDPI, **fontes com cobertura CJK/árabe/emoji + shaping/bidi** (senão tofu),
    UTF-8/locale/EOL.
  - `privacidade-seguranca.md` — **sem telemetria por padrão** (opt-in, sem PII, local-first),
    **nunca segredo no cliente** (public client, BFF, keychain do SO), autorização no servidor,
    **assinatura de binário** + verificação no update, **supply chain de deps nativas** (`cargo
    audit`, unsafe/FFI), superfície de ataque local.
  - `testes-locais.md` — domínio **headless**, **GUI headless no CI** por SO (snapshot com strings
    CJK/árabe), **smoke de empacotamento** em ambiente limpo, **update ponta-a-ponta**
    (vN→vN+1→rollback, versão coerente, processo antigo morto, artefato inválido recusado), gates por
    SO.
- **assets/commands/**: `/desktop-help`, `/desktop-load`, `/desktop-toolkit`, `/desktop-package`,
  `/desktop-update`, `/desktop-privilege`, `/desktop-claude`, `/desktop-cc`, `/desktop-handoff`.
- **assets/CLAUDE.md** — regra sempre-on: prever macacos; um só toolkit; updater desacoplado + versão
  coerente (git-dep); launcher com path absoluto; sem segredo no cliente; privilégio mínimo +
  privacidade por padrão; cross-OS testado; estado do usuário sagrado.
