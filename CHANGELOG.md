# CHANGELOG - SUPER WHITE X (Amlogic S905X / HAM905X1A0-B V4)

> Registro do que ja foi executado no projeto. Formato: `AAAA-MM-DD`.

## 2026-08-01 (2a sessao) - Diagnostico maskrom completo + ROM S905X escolhida

- Recovery CLI mapeado via pinhole: **Android Recovery padrao** (SuperTV/p212,
  6.0.1/MHC19J/20210318), menu numerado com 10 opcoes. **NAO e maskrom.**
  - Opcao 2 "Reboot to bootloader" -> volta ao Android normal (u-boot do
    clone NAO aciona modo burning por essa opcao)
  - Opcao 5 "Apply update from ADB" -> entra em `adb sideload` mas falha com
    `E:Cannot load volume /misc!` (particao /misc ilegivel/corrompida)
- Diagnostico USB completo (PowerShell): a box aparece SEMPRE como
  `USB\VID_0000&PID_0002` **error 43** ("Falha na Solicitação de Descritor")
  no hub interno `VID_05E3` (porta 4) - Windows ve D+/D- puxados mas a box
  NAO entrega o descritor. `adb devices` vazio. **Nunca** VID_1B8E.
- CONCLUSAO: box NAO esta em maskrom. Causas provaveis:
  1. **Energia instavel**: sem fonte DC, box puxa tudo do USB 2.0 (500mA);
     eMMC/DDR afundam a tensao na enumeração -> error 43 (mais provavel)
  2. **Porta conectada e host, nao OTG**: ligar host↔host no PC nunca enumera
- ATENCAO registrada: as 2 USB-A da box sao hosts; conectar as DUAS ao PC ao
  mesmo tempo cria host↔host + retroalimentacao 5V (risco de dano) - NAO fazer.
- **ROM de reposicao ESCOLHIDA**: atvXperience v4 (AndroidTV 9) **S905X**,
  versao **Realtek/Broadcom Multi Wi-Fi** (RTL8723BS), post #1 do XDA thread
  4175723. Links MEGA completos com chave confirmados no HTML do post:
  - Realtek: `mega.nz/file/TEw3UASD#1gtjWEWeKM_X24-48wLZyuiPl6_FMt2rksCxt1357dk`
  - Ampak fallback: `mega.nz/file/yYxBVYZR#B2J-KShlyUFrrc-fUdcZ-vkUS9-KFSis_so_9YAn40g`
  - Download NO NAVEGADOR (escolha do usuario); link aberto, hash a validar
- Link direto do site oficial (atvX v5 BETA1, WiFi universal)
  `download.umedialink.com/...zip` esta **fora do ar** (HTTP 522 Cloudflare,
  3 tentativas). GitHub `atvXperience/downloads` e so mirror (README).
- Imagem Armbian confirmada (2026-08-01): mesma da Etapa 1
  (`Armbian_26.08.0_amlogic_s905x_bookworm_6.18.38_server_2026.07.16.img.gz`)
  localizada em `releases/download/Armbian_bookworm_arm64_server_2026.07/`
- Decisao: boot Armbian via pendrive NAO passa pelo recovery; usar opcao 1
  ("Reboot system now") SEM segurar reset (chainload roda no u-boot normal).
- BLOQUEIO ATUAL: energia DC/hub alimentado para maskrom (usuario arranjara
  depois). Multimetro descarregado (UART J3 fica para depois).
- Docs atualizados: `TODO.md` (Etapa 8 reestruturada + bloqueio + Etapa 7),
  `CHANGELOG.md`. `docs/specs-hardware.md` e `README.md`/`boxes/` ja estavam
  atualizados na 1a sessao (ainda sem commit).
- Sem commit/PR (aguardando autorizacao do usuario).

## 2026-08-01 - Leitura macro da PCB corrige specs + USB Burning Tool funcional

- Leitura macro da PCB por IA (imagens de `imagens/`) corrige o hardware:
  - **SoC = Amlogic S905X** (lote J-T3YH14.OG) - NAO S905W; die M16B1 e
    compartilhado entre S905X e S905W (mesmo GXL, binning diferente), por isso
    a inspecao fisica anterior leu "S905W" por inferencia
  - **Placa = HAM905X1A0-B V4 2017-04-1** (silkscreen real)
  - **Wifi = Realtek RTL8723BS** (2.4GHz + Bluetooth 4.0) - NAO SSV6051P
  - **RAM = 4x Nanya NT5CC256M16EP-EK** (2GB DDR3)
  - **eMMC = Toshiba/Kioxia THGBMFG6C1LBAIL** em **BGA** -> **short fisico
    CANCELADO** (sem pinos laterais acessiveis)
  - Ethernet = PHY integrado ao SoC + trafo AE-SB1600+ (NAO RTL8201F discreto)
  - Firmware atual: Android 6.0.1, build MHC19J/20210318 (SuperTV/p212)
  - Novo: header J3 (4 pinos, suspeito UART) + header IR (3 pinos)
- Decisao: **ROM de reposicao deve ser S905X**; MXQ Pro S905W (725MB) arquivada
  como referencia, NAO usar. Buscar ROM p212/S905X com RTL8723BS ou
  atvXperience v4.x S905X.
- USB Burning Tool funcional: Microsoft Visual C++ 2010 Redistributable (x86)
  instalado -> `mfc100.dll`/`msvcp100.dll` agora em SysWOW64 (erro resolvido)
- Diagnostico maskrom da sessao:
  - Box em porta USB 2.0 direta da placa-mae (VID_05E3 = chip hub interno da
    placa, NAO hub externo)
  - "Reboot to bootloader" / "Update via adb" do recovery CLI faz a box
    enumerar como `VID_0000&PID_0002` error 43 (falha de descritor) - ainda
    NAO como `VID_1B8E` (WorldCup Device); `adb devices` vazio
  - Descoberta: o **outro** porta USB-A (NAO o acima do pinhole) entra no
    recovery CLI com reset -> provavelmente e a OTG/device real
  - Suspeita: energia instavel (box so com USB, sem DC) e/ou porta OTG errada
  - Pendente: entrar em maskrom na porta OTG correta + instalar driver
    `android_winusb.inf` (VID_1B8E&PID_C004) quando a box aparecer
- Docs atualizados: `docs/specs-hardware.md` (revisao 3), `boxes/`,
  `README.md`, `TODO.md` (Etapa 8 reestruturada, curto cancelado)
- Sem commit/PR (aguardando autorizacao do usuario)

## 2026-07-31 - Sessao interrompida: instalacao de tool/driver travada

- Pacote ophub extraido OK:
  `Amlogic_USB_Burning_Tool_v2.2.4.exe` (8,9MB) + `SECURE_BOOT_SET`
  (licenca embutida, 1024B - NAO precisa copiar pasta `license/`) +
  `Amlogic_Driver.zip`; INF registra VID_1B8E&PID_C004 (UTF-16LE)
- ERRO 1: USB Burning Tool abre com "mfc100.dll nao encontrado" -> falta
  Microsoft Visual C++ 2010 Redistributable (x86)
- ERRO 2: dpinst64 falha na instalacao (libwdi/WinUSB) - NAO determinante,
  porque a box nunca enumerou no USB (zero VID_1B8E no historico do Windows)
- Diagnostico de maskrom pendente: testar 4 pinos no conector do cabo, LED
  via USB sem DC, porta direta da placa-mae (PC tem 2 hubs), todas as portas
  USB-A da box, reset ~10s com DC na tomada
- ROM baixada: `MXQPRO_S905W_20171218_PC_AndroidPC.rar` (Downloads)
- Proxima sessao: VC++ 2010 x86 -> diagnosticar maskrom -> driver via
  Device Manager (`amd64\android_winusb.inf`) -> flash sem Erase Bootloader

## 2026-07-31 - Decisao: USB Burning Tool (maskrom) apos toothpick falhar

- Toothpick testado nas 2 portas USB (pendrive com `u-boot.ext` + reset
  pinhole): NAO boota a midia; causa provavel = u-boot Android do clone sem
  suporte `recovery_from_udisk` (reset so entra em modo burning)
- Android original nao expoe Developer Options / USB Debugging e `adbd` nao
  escuta (scan de portas negativo) -> caminho adb descartado
- Decidido: maskrom + USB Burning Tool + ROM **S905W** (MXQ Pro 4K, SSV6051P)
- Downloads marcados como baixados (pendente validacao de hash):
  - USB Burning Tool v2.2.4 + driver (ophub release `tools`):
    `https://github.com/ophub/kernel/releases/download/tools/amlogic_usb_burning_tool_v2.2.4_and_driver.tar.xz`
    (SHA256 `c08b68952fd08305582514848427c1a61ac179133bbdc04c3e7f617b0ccdb0b3`)
  - ROM `MXQPRO_S905W_20171218` -> link FileFactory expirou (30 dias sem
    download); substituto no MEGA (verificado, 691MB):
    `https://mega.nz/file/YrBTCIaR#fhxJPn4f_-t4Gu2WelbX4sAflQ8gwsVfIzjdN3R0pAQ`
    - usar `aml_upgrade_package.img` extraido
  - Alternativa com ADB over Ethernet (resolve acesso adb):
    atvXperience v4.3 S905W (Android 9, ssv6051):
    `https://mega.nz/file/Jvw3CAQA#rPhpGfPI-4sNfSayxlgRP-wcFlIfKZwBqU4HnKPxdiw`
- TODO.md atualizado com a Etapa 8 (maskrom/flash) para retomada em nova sessao

## 2026-07-31 - Identificacao fisica do hardware (CORRECAO para S905W)

- Dissipador do SoC removido e PCB fotografada -> chip lido diretamente
- **CORRECAO**: o SoC e **Amlogic S905W** (marcacao M16B1), NAO S905X. O
  CPU-Z do Android reportava S905X/p212 porque usa o dt-id do firmware
  (`gxl_p212_2g`) - comum em boxes S905W
- Placa: CLONE **S905XQ4_V1.0** (2018.03.20), NAO p212 de referencia
- RAM: 2GB DDR3 (2x1GB, NANYA provavel); eMMC ~8GB est. (chip sob etiqueta)
- Wifi: **SSV6051P** (2.4GHz, SEM Bluetooth); Ethernet: **RTL8201F** (10/100)
- OTG: porta USB-A acima do pinhole de reset; reset na borda direita
  (entre USB-A e HDMI)
- Impacto: imagem Armbian `s905x` ophub permanece valida (familia GXL cobre
  S905W); `u-boot-s905x-s912.bin` continua correto; DTB p212 mantido
  (dt-id do Android), alternativa `meson-gxl-s905w-p281.dtb`
- ROM Android para flash via maskrom (F2): deve ser **S905W** com suporte a
  SSV6051P (NAO S905X/p212)
- Docs atualizados: `docs/specs-hardware.md` (novo registro), `boxes/`,
  `README.md`, `TODO.md`, `docs/diagnostico-tecnico.md`, `docs/plano-execucao.md`
- OBS: pagina `boxes/superwhitex.md` ja publicada no fork com dados S905X;
  requer commit de correcao no fork quando finalizado

## 2026-07-31 - Box conectada na rede (preparacao do boot)

- Wifi da box desligado (somente ethernet ativa)
- Cabo Ethernet conectado ao switch - link OK
- IP descoberto: PC `192.168.100.29`, box `192.168.100.32` (ping OK, TTL 64)
- MAC da box em `arp -a`: `ee-79-02-85-31-45`
- Pendrive (com `u-boot.ext`) inserido na USB da box
- Proximos passos: `adb connect` + `adb shell reboot update`

## 2026-07-31 - Publicacao no fork + docs de planejamento

- Fork de `educabox/educabox` criado em `cavalcanteprofissional/educabox`
- Branch `feat/superwhitex-s905x` com 2 commits:
  - `9f4ab8a` - docs(boxes): add SUPER WHITE X (Amlogic S905X/p212) box page
  - `3da0eae` - docs(readme): add SUPER WHITE X row to box table
- Autenticacao GitHub via `GITHUB_TOKEN` (o MCP nao pegou a variavel por ser
  processo antigo; fork/push feitos via `git`/API no shell)
- Adicionados `TODO.md` e `CHANGELOG.md` na raiz do projeto
- Commit local `d59b641` - docs: mark fork publish done

## 2026-07-31 - Preparacao completa do pendrive

- Imagem gravada: `Armbian_26.08.0_amlogic_s905x_bookworm_6.18.38_server_2026.07.16.img.gz`
- SHA256 conferido: `0a373487e510c3f81f01d6250a17fc474e9d2fc922d5be0950542c4ad88ec304`
- Particioes validadas (BOOT FAT32 + ext4)
- `u-boot.ext` criado na BOOT (E:) = copia de `u-boot-s905x-s912.bin`

## 2026-07-31 - Causa raiz do boot falho identificada

- Imagem ophub 26.08.0 NAO grava bootloader nos setores 0-4MB da midia
- Por isso o botao reset nunca iniciou a midia (bootrom sem o que carregar)
- Fluxo correto: `aml_autoscript` -> `s905_autoscript` -> `u-boot.ext` -> kernel
- Metodo oficial ophub: `adb connect <IP>:5555` + `adb shell reboot update`
- Documentado em `docs/diagnostico-tecnico.md`

## 2026-07-31 - Identificacao do hardware

- CPU-Z no Android original: Amlogic S905X, placa p212, 2GB RAM, eMMC 4.64GB
- DTB: `meson-gxl-s905x-p212.dtb` (p212 = variante do MyTVBox BRAVE 4K)
- Distinta da SUPER TV (Rockchip RK3229) ja catalogada no educabox

## 2026-07-30 - Estrutura inicial do projeto

- Diretorios: `boxes/`, `docs/`, `tools/`
- Arquivos: `README.md`, `boxes/superwhitex.md`, `docs/diagnostico-tecnico.md`,
  `docs/plano-execucao.md`, `docs/PR-educabox.md`, `.gitignore`
- Git local iniciado (branch `feat/superwhitex-s905x`)
- Commits: `599c452` (docs iniciais), `11cfad1` (remove zip), `76d733e` (status/rumo)
