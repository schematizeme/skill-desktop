# Anexo volátil — toolkits, empacotamento e requisitos de SO

> Parte da skill **schematize-desktop**. **Fonte volátil:** versão de toolkit, requisito de
> assinatura e regra de notarization mudam. O corpo normativo diz **princípio**; o que tem prazo
> mora aqui. O lint do catálogo (regra `anexo-volatil`) reprova versão cravada fora daqui.
>
> **Verificado em: 2026-08-21.** Cadência: **revisão trimestral**, e sempre antes de um release.

## Toolkits de GUI

| Toolkit | Posição da casa | Nota |
|---|---|---|
| **Slint** (Rust) | **viés preferencial** para produto novo | declarativo, sem runtime web; casa com o backend do rol |
| **Tauri** | aceito **com justificativa em ADR** | traz webview do sistema: herda o piso de `schematize-web` §43.9 — **XSS vira RCE local** se a ponte for ampla; allowlist mínima de comandos |
| **Electron** | aceito **com justificativa em ADR** | `contextIsolation: true` + `nodeIntegration: false` **sem exceção**; ponte por `contextBridge` com allowlist |
| **egui** | **não** como principal | modo imediato: o "fantasma de layout" reaparece a cada release |
| **GTK / Qt** | aceitos por fit (integração de desktop Linux, licença do Qt em ADR) | |

> **UM toolkit por produto.** Dois toolkits = dois bugs de layout, dois pipelines de empacotamento
> e duas matrizes de teste. Isso **não é volátil** e mora no corpo normativo.

## Empacotamento e assinatura — o que tem prazo

| Plataforma | Requisito | Nota (2026-08-21) |
|---|---|---|
| **macOS** | **notarization obrigatória** para distribuição fora da App Store; `.dmg`/`.pkg` assinados com Developer ID e **stapled** | sem notarizar, o Gatekeeper bloqueia com mensagem que o usuário lê como "vírus" |
| **Windows** | assinatura de código; **certificado EV/attestation em HSM** — a era do `.pfx` no CI acabou | MSIX ou MSI assinado; SmartScreen usa reputação, que **começa do zero** a cada troca de certificado |
| **Linux** | `.deb`/`.rpm` assinados, AppImage com assinatura destacada, Flatpak pelo runtime da distro | repositório próprio com chave publicada |

## A regra que NÃO é volátil

Updater **desacoplado** (app quebrado não pode travar o update), versão embutida **coerente** com
a publicada, launcher com **`Exec` absoluto**, **nunca segredo no cliente** (binário distribuído é
segredo publicado), e **UX de massa**: o software se adapta e **nunca culpa o usuário**.

> **A armadilha que já custou release aqui:** dependência de **git** no lockfile (`Cargo.lock`
> pinando um commit velho) faz o build embutir uma versão **antiga** sem avisar. Rode o
> `cargo update -p <crate>` antes de compilar o release, e faça o `--version` do binário ser
> gerado do que foi **realmente** compilado.
