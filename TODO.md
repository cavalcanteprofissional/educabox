# TODO - SUPER WHITE X (Amlogic S905W / S905XQ4_V1.0)

> Plano de execucao do projeto. Status atualizado em 2026-07-31.
> FASE ATUAL: Etapa 8 (USB Burning Tool/maskrom) - downloads prontos,
> flash pendente. Retomar por a partir da Etapa 8.
> ATENCAO: inspecao fisica corrigiu o hardware - o SoC e S905W (M16B1), nao
> S905X (o CPU-Z reportava o dt-id do firmware, `gxl_p212_2g`). A imagem
> Armbian `s905x` da ophub cobre a familia GXL (S905X/S905D/S905W/S905L) e
> permanece valida. Ver `docs/specs-hardware.md`.

## Etapa 1 - Preparar o pendrive

- [x] Baixar imagem: `Armbian_26.08.0_amlogic_s905x_bookworm_6.18.38_server_2026.07.16.img.gz`
- [x] Conferir SHA256: `0a373487e510c3f81f01d6250a17fc474e9d2fc922d5be0950542c4ad88ec304`
- [x] Gravar com balenaEtcher no SanDisk Cruzer Blade 8GB
- [x] Verificar particoes (BOOT FAT32 + ext4) - valido
- [x] Criar `u-boot.ext` na BOOT (E:) = copia de `u-boot-s905x-s912.bin` (hash `207cdbe1...`)

## Etapa 2 - Obter o adb (platform-tools)

- [x] Baixar platform-tools do Google (Windows) - `tools/platform-tools`
- [x] Testar `adb version` (37.0.1)

## Etapa 3 - Diagnostico da causa raiz do boot falho

- [x] Identificar hardware via CPU-Z (Android): reportou S905X, p212, 2GB, eMMC 4.64GB
- [x] Inspecao fisica (dissipador do SoC removido): **S905W (M16B1)**, placa
      clone S905XQ4_V1.0 (2018.03.20), wifi SSV6051P (2.4GHz, sem BT),
      ethernet RTL8201F, eMMC ~8GB est. - registro em `docs/specs-hardware.md`
- [x] Diagnosticar que a imagem ophub 26.08.0 NAO grava bootloader no setor 0
- [x] Confirmar chainload `s905_autoscript` -> `u-boot.ext` (offsets 439/463/480)
- [x] Documentar em `docs/diagnostico-tecnico.md`

## Etapa 4 - Publicar no fork do educabox

- [x] Autenticar GitHub (token `GITHUB_TOKEN`)
- [x] Fork: `cavalcanteprofissional/educabox`
- [x] Branch `feat/superwhitex-s905x` com commits `9f4ab8a` + `3da0eae`
- [x] Sem PR - apenas commits no fork

## Etapa 5 - Conectar a box na rede

- [x] Conectar a TV Box ao switch via cabo Ethernet (PC ja esta no switch)
- [x] Desligar o wifi da box (somente ethernet ativa)
- [x] Descobrir o IP: PC `192.168.100.29` / box `192.168.100.32` (ping OK, TTL 64)
- [x] MAC da box em `arp -a`: `ee-79-02-85-31-45` (localmente administrado)

## Etapa 6 - Boot via adb (DESCARTADO)

- [x] Inserir o pendrive na porta USB da box (USB1 ou USB2 - tanto faz)
- [x] `adb connect 192.168.100.32:5555` - FALHOU: `adbd` nao escuta
      (scan de portas negativo)
- [x] Android original nao expoe Developer Options / USB Debugging -> adb inviavel
- [x] CONCLUSÃO: caminho via adb descartado; seguir para Etapa 8 (USB Burning Tool)

## Etapa 7 - Instalacao para eMMC

- [ ] Login: `root` / `1234`
- [ ] Backup do Android original: `armbian-ddbr` (opcao `b`)
- [ ] `sudo armbian-install` -> instalar no eMMC (ext4)
- [ ] Remover o pendrive e reiniciar

## Etapa 8 - USB Burning Tool (maskrom) - CAMINHO EM ANDAMENTO

> Contexto: toothpick testado nas 2 portas USB (pendrive com `u-boot.ext` +
> reset pinhole) -> NAO boota a midia. Causa provavel: u-boot Android do
> clone sem suporte `recovery_from_udisk` (reset so entra em modo burning).
> Flash de ROM **S905W** com u-boot conhecido resolve o chainload.
> ATENCAO: ROM "p212/S905X" NAO funciona nesta box (bootloader diferente).
>
> FASE ATUAL (2026-07-31, sessao interrompida): instalacao de tool/driver
> travada em 2 pontos:
>  1. USB Burning Tool abre com erro "mfc100.dll nao encontrado" -> falta o
>     Microsoft Visual C++ 2010 Redistributable (x86) - instalar primeiro.
>  2. dpinst64 falha (libwdi/WinUSB) E a box NAO enumera no USB (nunca houve
>     VID_1B8E no historico do Windows) -> o problema real e a conexao/
>     entrada em maskrom, NAO o driver. Diagnosticar cabo/porta antes.

- [x] Baixar USB Burning Tool v2.2.4 + driver (release `tools` do ophub):
      `https://github.com/ophub/kernel/releases/download/tools/amlogic_usb_burning_tool_v2.2.4_and_driver.tar.xz`
      SHA256: `c08b68952fd08305582514848427c1a61ac179133bbdc04c3e7f617b0ccdb0b3` (14.5MB)
- [x] Baixar imagem S905W MXQ Pro 4K (mesma familia, SSV6051P) - MEGA (link FileFactory morreu):
      `https://mega.nz/file/YrBTCIaR#fhxJPn4f_-t4Gu2WelbX4sAflQ8gwsVfIzjdN3R0pAQ` (691MB, PC/USB Burning Tool)
      - extrair -> usar `aml_upgrade_package.img` (NAO o zip)
      - fallback se wifi nao pegar: build `905w_ssv6051_8g1g2g_171101` (ethernet cobre)

### Etapa 8b - Variante B: atvXperience v4.3 S905W (FALLBACK se a stock falhar)

> DECISAO: comecar pela ROM MXQ Pro stock (Etapa 8). Usar a atvX v4.3 SOMENTE
> se a stock nao bootar apos o flash.
> Por que considerar: ROM Android 9 feita para S905W, suporta o chip wifi
> **SSV6051P** e vem com **ADB over Ethernet** ja ativo - resolve o bloqueio
> do Android original (sem Developer Options) e evita depender de toothpick.
> Testada oficialmente em X96 Mini (placa p282, mesma familia do nosso clone)
> e MXQ Pro. Uso via USB Burning Tool igual ao MXQ Pro stock.
> RISCO: atvX v4 e Android 9 (kernel/DTB diferente do stock 7.1).

- [ ] Baixar atvXperience v4.3 S905W:
      `https://mega.nz/file/Jvw3CAQA#rPhpGfPI-4sNfSayxlgRP-wcFlIfKZwBqU4HnKPxdiw` (751MB)
      - e arquivo de remotes (controle) se o IR da box nao responder:
        `https://mega.nz/folder/JnxTDKTD#f3SSkmX1by3LJO7dhWx7dw`
- [ ] Mesmo flash da Etapa 8 (maskrom + USB Burning Tool, sem Erase Bootloader na 1a vez)
- [ ] No 1o boot: `adb connect 192.168.100.32:5555` (ADB over Ethernet ja ativo)
      OU Terminal Emulator na box: `reboot update`
- [ ] Se bootar OK: seguir Etapa 7 (ddbr backup + armbian-install)
- [ ] Se NAO bootar: decidir entre tentar flash novamente com "Erase Bootloader"
      marcado OU pular para a atvX v4.3
- [ ] Validar hash/checksum dos downloads
- [ ] Instalar Microsoft Visual C++ 2010 Redistributable (x86) -> corrige o
      erro "mfc100.dll nao encontrado" do USB Burning Tool
- [ ] DIAGNOSTICO maskrom (a box nunca enumerou - resolver ANTES do driver):
      - ver se o conector USB-A do cabo tem 4 pinos (cabo "so energia" nao tem dados)
      - testar se a box acende LED alimentada SO pelo cabo USB (sem DC)
      - conectar em porta DIRETA da placa-mae (PC tem 2 hubs: VID_214B, VID_05E3)
      - testar TODAS as portas USB-A da box (a OTG pode nao ser a do pinhole)
      - segurar reset ~10s com o DC ligado na tomada e a box desligada
- [ ] Instalar driver SOMENTE apos a box aparecer: Gerenciador de Dispositivos ->
      dispositivo desconhecido -> Atualizar driver -> procurar no computador ->
      `Amlogic_Driver\Amlogic_Driver\amd64\android_winusb.inf` (VID_1B8E&PID_C004)
      - obs: pacote ophub NAO tem pasta `license/`; a chave SECURE_BOOT_SET ja
        esta ao lado do .exe (nao mover o .exe sozinho)
- [ ] Entrar em maskrom: box DESLIGADA + cabo USB-A macho-macho na porta OTG
      (acima do pinhole) + segurar reset + conectar ao PC (USB alimenta) +
      soltar o reset ao detectar "WorldCup Device" (VID_1B8E) - confirmar via PowerShell
- [ ] USB Burning Tool: File > Import image -> `aml_upgrade_package.img`
- [ ] Flash: marcar "Erase Flash"; NAO marcar "Erase Bootloader" na 1a tentativa
      (clone perde reflash se bootloader sumir); NAO usar "Overwrite key";
      aguardar 100% e parar
- [ ] Se nao bootar: repetir flash com "Erase Bootloader" marcado
- [ ] 1o boot demora minutos; testar `adb connect 192.168.100.32:5555`
- [ ] Boot Armbian: `adb shell reboot update` OU toothpick (u-boot MXQ Pro suporta udisk)
- [ ] Backup do Android: `armbian-ddbr` (opcao `b`)
- [ ] Instalar eMMC: `sudo armbian-install`
- [ ] Corrigir a pagina do fork com os dados S905W e commitar

## Fallbacks (se o boot via adb falhar)

- [ ] F1 - App "Local Update" da box (`aml_autoscript.zip`) - nao funcionou nesta box
- [x] F2 - USB Burning Tool - EM ANDAMENTO (Etapa 8 acima); toothpick com pendrive
      falhou (u-boot sem `recovery_from_udisk`)
- [ ] F3 - Console serial (USB-TTL nos pinos UART)

## Pendencias antes de finalizar os docs

- [x] Identificar chip wifi da box: SSV6051P (2.4GHz, sem BT) - `docs/specs-hardware.md`
- [ ] Confirmar driver SSV6051P no Armbian (a testar no primeiro boot)
- [ ] Confirmar audio (ok/falho) apos primeiro boot
- [ ] Capturar screenshots para a pasta `imagens` do repositorio
