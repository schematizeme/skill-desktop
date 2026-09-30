# schematize-desktop

> **Engenharia de software local/nativo/desktop** da casa — o **mesmo piso** (segurança, IAM,
> testes, ops, DoD, archive) da `schematize-engineering`, aplicado ao software que roda **na máquina
> do usuário**, onde **não há servidor pra segurar a barra**, **cada SO é diferente** e **o usuário
> é leigo e sem suporte**. Muda o "como", nunca o "o quê": nada de afrouxar segurança porque "é só
> um app local".

Pacote de **skill normativa para [Claude Code](https://claude.com/claude-code)**.
Parte do catálogo **schematize skills**. Irmã de `schematize-mobile` e `schematize-web`; pareia com
a `schematize-engineering` (a base: UX de massa/segurança/DoD/archive/índice), com a
`schematize-rust` (Slint/Tauri, o `schematize-updater`) e com a `schematize-pentest` (a máquina do
usuário é território hostil).

## Instalar

### Pelo app schematize (recomendado)

```bash
schematize install desktop      # requer o CLI schematize instalado
```

### Última versão (a partir de um clone)

```bash
git clone https://github.com/schematizeme/skill-desktop.git
cd skill-desktop && ./install.sh            # instala no projeto atual
# ./install.sh /caminho/do/projeto          # ou aponte para outro projeto
```

Ou baixe o `.zip` da última release e descompacte em `.claude/skills/`:

```bash
curl -L -o skill-desktop.zip \
  https://github.com/schematizeme/skill-desktop/releases/latest/download/skill-desktop.zip
unzip skill-desktop.zip -d .claude/skills/
```

## O que tem dentro

- **SKILL.md** — o contrato: 9 pisos inegociáveis (o piso da casa é o mesmo; **UX de massa — prever
  macacos**; **um só toolkit de GUI por produto**; **updater desacoplado + versão embutida
  coerente**; **launcher com path absoluto**; **sem segredo no cliente**; privilégio mínimo +
  privacidade por padrão; cross-OS é requisito) + mapa de references.
- **references/** — `toolkits-gui` (Slint/Tauri/egui/GTK/Qt, um só por produto), `empacotamento`
  (.deb/.rpm/AppImage/Flatpak, .dmg/notarization, MSI/MSIX/assinatura, binário vs fonte híbrido),
  `auto-update` (desacoplado, git-dep no Cargo.lock, matar processo antigo + re-exec, pin/rollback),
  `offline-estado` (dirs XDG/por SO, migração de schema, backup, sync), `privilegio-fs`
  (root→usuário real, launcher absoluto, single-instance, IPC), `cross-os` (path/processo/fontes
  CJK-árabe/Wayland-X11), `privacidade-seguranca` (sem telemetria por padrão, assinatura de binário,
  sem segredo no cliente), `testes-locais` (GUI headless, smoke de pacote, update ponta-a-ponta).
- **assets/commands/** — `/desktop-help`, `/desktop-load`, `/desktop-toolkit`, `/desktop-package`,
  `/desktop-update`, `/desktop-privilege`, `/desktop-claude`, `/desktop-cc`, `/desktop-handoff`.
- **assets/CLAUDE.md** — regra sempre-on do piso desktop.

## Regra de ouro

**O piso da casa é o mesmo — o desktop só muda o "como".** Na máquina do usuário não há servidor pra
segurar a barra e o usuário é leigo: por isso o software **prever macacos** (se adapta e nunca culpa
o usuário — root vira usuário real, PATH mínimo se resolve, toolchain ausente se instala), **um só
toolkit por produto** (misturar = fantasma de layout), **updater desacoplado com versão embutida
coerente** (a armadilha do git-dep pinado no `Cargo.lock`), **launcher com path absoluto** (o PATH do
DE não acha o binário), e **segredo nunca no cliente**. Segurança que depende do binário local é
enfeite; a que vale mora no servidor.

## Relação com as outras skills

- **schematize-engineering** — a base: UX de massa (prever macacos), segurança, IAM (`iam.md`), DoD
  (§35), archive (§28), índice (§39).
- **schematize-rust** — Slint/Tauri em Rust, o `schematize-updater`, os pisos de código.
- **schematize-mobile / schematize-web** — as irmãs: mesmo piso, cliente diferente; "segredo nunca
  no cliente" nos três.
- **schematize-pentest** — o oráculo do cliente hostil (segredo no binário, update sequestrável,
  escalada por privilégio, path traversal em IPC/deep-link).

Co-autoria / patrocínio: Lucassa — https://lucassa.me

MIT.
