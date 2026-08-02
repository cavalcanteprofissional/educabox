# TODO - SUPER WHITE X (Amlogic S905X / HAM905X1A0-B V4)

> Plano de execucao do projeto. Status atualizado em 2026-08-01 (2a sessao).
> FASE ATUAL: Etapa 8 (maskrom via USB OTG) - **BLOQUEADA por energia**.
> USB Burning Tool funcional (VC++ 2010 x86). ROM S905X escolhida e link
> confirmado (aguardando download no navegador). Maskrom ainda NAO enumera
> VID_1B8E; sintoma = VID_0000 error 43 (descritor falho) em todos os metodos.
> **PAROU AQUI**: box em porta USB da placa-mae; falta fonte DC/hub alimentado
> p/ dar corrente estavel; depois testar porta OTG correta 1 cabo por vez.
> ATENCAO (2026-08-01): leitura macro da PCB por IA corrigiu o hardware:
> **SoC = S905X** (die M16B1 compartilhado com S905W), placa
> **HAM905X1A0-B V4**, wifi **RTL8723BS** (com BT) e eMMC **Toshiba BGA**
> (sem short fisico viavel). ROM de reposicao deve ser **S905X**.
> Ver `docs/specs-hardware.md`.
> Recovery CLI (via pinhole) = recovery Android padrao, NAO maskrom; opcao
> "Reboot to bootloader" volta ao Android normal; "Apply update from ADB"
> entra em adb sideload mas falha com `E:Cannot load volume /misc!`
> (particao /misc ilegivel - reforca que o caminho certo e USB Burning Tool).

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

> Boot do Armbian (pendrive): NAO ha opcao "boot from USB" no recovery CLI.
> O chainload roda no u-boot NORMAL: inserir o pendrive e escolher a opcao 1
> ("Reboot system now") SEM segurar o pinhole. Se a box entrar em bootloop
> antes do Android, o chainload ainda roda (u-boot e anterior ao Android).
> Imagem Armbian confirmada (2026-08-01): `Armbian_26.08.0_amlogic_s905x_...`
> em `releases/download/Armbian_bookworm_arm64_server_2026.07/` (mesma ja
> gravada; link e SHA256 na Etapa 1).

- [ ] (PENDENTE TESTE) Pendrive na USB + opcao 1 do recovery -> ver se o
      chainload sobe o Armbian (testar sem segurar reset)
- [ ] Login: `root` / `1234`
- [ ] Backup do Android original: `armbian-ddbr` (opcao `b`)
- [ ] `sudo armbian-install` -> instalar no eMMC (ext4)
- [ ] Remover o pendrive e reiniciar

## Etapa 8 - USB Burning Tool / maskrom - EM ANDAMENTO

> Contexto: o eMMC e **BGA** (Toshiba THGBMFG6C1LBAIL) -> **short fisico
> CANCELADO** (sem pinos laterais). Recovery possivel via **maskrom USB OTG**
> (placa desligada + cabo USB na OTG + energia) ou **debug UART (J3)**.
> RESULTADO DO DIAGNOSTICO (2026-08-01, 2a sessao):
> - "Reboot to bootloader" (recovery CLI, opcao 2) NAO entra em modo burning:
>   volta ao Android normal (u-boot do clone sem esse suporte)
> - "Apply update from ADB" (opcao 5): entra em adb sideload mas falha com
>   `E:Cannot load volume /misc!` -> particao /misc ilegivel/corrompida
> - Em TODOS os metodos a box aparece como `USB\VID_0000&PID_0002` error 43
>   ("Falha na Solicitação de Descritor") no hub interno 05E3 (porta 4) -
>   Windows ve os pinos D+/D- puxados mas a box NAO entrega o descritor.
>   `adb devices` vazio. **Nunca** apareceu VID_1B8E (WorldCup).
> - CONCLUSAO: box NAO esta em maskrom. Causas provaveis: (a) **energia
>   instavel** (sem fonte DC, box puxa tudo do USB 2.0 = 500mA; eMMC/DDR
>   afundam a tensao na enumeração) e/ou (b) **porta conectada e host**, nao
>   OTG (ligar host↔host no PC nunca enumera -> exatamente VID_0000 error 43).
> - ATENCAO: as 2 USB-A da box sao hosts; ligar as duas ao PC ao mesmo tempo
>   cria host↔host + retroalimentacao de 5V - NAO fazer. Um cabo por vez.
> - PROXIMO PASSO (bloqueio): obter **fonte DC 5V** (ou hub USB alimentado)
>   e testar a porta OTG correta (o outro USB-A, o que entra no recovery CLI),
>   com USB Burning Tool aberto em Start (aguardando) + reset na energizacao.

- [x] Baixar USB Burning Tool v2.2.4 + driver (release `tools` do ophub):
      `https://github.com/ophub/kernel/releases/download/tools/amlogic_usb_burning_tool_v2.2.4_and_driver.tar.xz`
      SHA256: `c08b68952fd08305582514848427c1a61ac179133bbdc04c3e7f617b0ccdb0b3` (14.5MB)
- [x] Instalar Microsoft Visual C++ 2010 Redistributable (x86) -> corrige o
      erro "mfc100.dll nao encontrado" do USB Burning Tool
- [x] Baixar ROM MXQ Pro 4K (S905W, arquivada como REFERENCIA - NAO usar):
      `https://mega.nz/file/YrBTCIaR#fhxJPn4f_-t4Gu2WelbX4sAflQ8gwsVfIzjdN3R0pAQ` (691MB)
- [ ] Baixar ROM **S905X** correta - atvXperience v4 (AndroidTV 9) S905X,
      versao **Realtek/Broadcom Multi Wi-Fi** (compativel com RTL8723BS),
      fonte XDA `[S905X] [9.0] atvXperience v4 - AndroidTV Pie`
      (thread 4175723, post #1). Links MEGA completos (com chave):
      - Realtek/Broadcom (escolhida p/ RTL8723BS):
        `https://mega.nz/file/TEw3UASD#1gtjWEWeKM_X24-48wLZyuiPl6_FMt2rksCxt1357dk`
      - Ampak (fallback):
        `https://mega.nz/file/yYxBVYZR#B2J-KShlyUFrrc-fUdcZ-vkUS9-KFSis_so_9YAn40g`
      ADB over Ethernet ja ativo nesta ROM.
      STATUS (2026-08-01): link MEGA confirmado com chave no post do XDA;
      download NO NAVEGADOR escolhido pelo usuario (link aberto em 2026-08-01,
      baixar para `imagens/`, validar hash apos).
      Obs: link direto do site oficial (v5 BETA1, WiFi universal)
      `https://download.umedialink.com/Beta%201-20211124T213627Z-001.zip`
      esta fora do ar (HTTP 522 Cloudflare, 3 tentativas) - verificado em
      2026-08-01; GitHub `atvXperience/downloads` e so um mirror (README).
      Alternativa stock p212: `MXQPRO_S905X` (arquivos de forum costumam
      estar corrompidos - atvX e mais confiavel)
- [x] DIAGNOSTICO maskrom (2026-08-01, 2a sessao):
      - recovery CLI mapeado: menu Android Recovery padrao (10 opcoes),
        SuperTV/p212, 6.0.1/MHC19J/20210318
      - "Reboot to bootloader" -> volta ao Android normal (NAO vira WorldCup)
      - "Apply update from ADB" -> adb sideload, mas `E:Cannot load volume /misc!`
      - box sempre em `VID_0000` error 43 (hub interno 05E3, porta 4), nunca
        VID_1B8E; `adb devices` vazio; testado com reset segurado + conectar
        USB e com os 2 cabos nas 2 USB-A (host↔host - NAO fazer)
      - CONCLUSAO: nao esta em maskrom; suspeita #1 energia (sem DC),
        #2 porta OTG (as 2 USB-A sao host)
- [ ] OBTER ENERGIA ESTAVEL (BLOQUEIO): fonte DC 5V pra box OU hub USB
      alimentado. Sem isso o error 43 tende a persistir. [Usuario: nao tem
      agora, tera depois; multimetro descarregado, usara depois]
- [ ] Testar maskrom com energia: USB Burning Tool -> Import image (S905X)
      -> Start (aguardando) -> box com DC -> segurar reset -> energizar ->
      soltar reset ao detectar WorldCup (VID_1B8E) - confirmar via PowerShell
- [ ] Instalar driver SOMENTE apos a box aparecer: Gerenciador de Dispositivos ->
      dispositivo desconhecido -> Atualizar driver -> procurar no computador ->
      `Amlogic_Driver\Amlogic_Driver\amd64\android_winusb.inf` (VID_1B8E&PID_C004)
- [ ] Entrar em maskrom: box DESLIGADA + cabo USB-A macho-macho na porta OTG
      correta + segurar reset + conectar ao PC (USB alimenta) + soltar o reset
      ao detectar "WorldCup Device" (VID_1B8E) - confirmar via PowerShell
- [ ] USB Burning Tool: File > Import image -> `aml_upgrade_package.img` (S905X)
- [ ] Flash: marcar "Erase Flash"; NAO marcar "Erase Bootloader" na 1a tentativa
      (clone perde reflash se bootloader sumir); NAO usar "Overwrite key";
      aguardar 100% e parar
- [ ] Se nao bootar: repetir flash com "Erase Bootloader" marcado
- [ ] 1o boot demora minutos; testar `adb connect 192.168.100.32:5555`
- [ ] Boot Armbian: `adb shell reboot update` OU toothpick (u-boot da ROM
      S905X suporta udisk)
- [ ] Backup do Android: `armbian-ddbr` (opcao `b`)
- [ ] Instalar eMMC: `sudo armbian-install`
- [ ] Corrigir a pagina do fork com os dados S905X e commitar

### Etapa 8b - Variante: atvXperience v4.x S905X (ROM principal escolhida)

> DECISAO (2026-08-01): a ROM de reposicao deve ser **S905X**, NAO a S905W
> ja baixada. **atvXperience v4 S905X (Android 9, ADB over Ethernet ativo)
> e a ROM ESCOLHIDA** (versao Realtek/Broadcom para o RTL8723BS) - ver links
> na Etapa 8. Stock p212 e alternativa (arquivos de forum costumam estar
> corrompidos).

- [ ] Baixar atvXperience v4 **S905X** Realtek/Broadcom (link confirmado, Etapa 8)
- [ ] Mesmo flash da Etapa 8 (maskrom + USB Burning Tool, sem Erase Bootloader na 1a vez)
- [ ] No 1o boot: `adb connect 192.168.100.32:5555` (ADB over Ethernet ja ativo)
      OU Terminal Emulator na box: `reboot update`
- [ ] Se bootar OK: seguir Etapa 7 (ddbr backup + armbian-install)
- [ ] Se NAO bootar: decidir entre tentar flash novamente com "Erase Bootloader"
      marcado OU pular para outra ROM S905X

### Etapa 8c - Debug via UART (J3) - alternativo ao maskrom

> Header J3 (4 pinos, perto do SoC) com suspeita forte de ser UART
> (TX/RX/GND/VCC). Pinagem NAO confirmada - precisa multimetro.
> STATUS (2026-08-01, 2a sessao): usuario tem multimetro mas esta
> DESCARREGADO; testara depois. Conversor USB-TTL nao confirmado.

- [ ] Confirmar pinagem do J3 com multimetro: continuidade para GND; tensao DC
      para TX (varia no boot) vs RX/VCC (~3.3V estavel)
- [ ] Conectar conversor USB-TTL cruzado (TX placa -> RX conversor, RX placa ->
      TX conversor; NAO conectar VCC do conversor com a placa alimentada)
- [ ] Interagir com u-boot: `update` / boot USB forçado / identificar SoC

## Fallbacks (se o boot via adb falhar)

- [ ] F1 - App "Local Update" da box (`aml_autoscript.zip`) - nao funcionou nesta box
- [ ] F2 - USB Burning Tool / maskrom - EM ANDAMENTO (Etapa 8 acima)
- [ ] F3 - Console serial (USB-TTL nos pinos UART do J3)

## Pendencias antes de finalizar os docs

- [x] Identificar chip wifi da box: RTL8723BS (2.4GHz + BT) - `docs/specs-hardware.md`
- [ ] Confirmar driver RTL8723BS no Armbian (a testar no primeiro boot)
- [ ] Confirmar audio (ok/falho) apos primeiro boot
- [ ] Capturar screenshots para a pasta `imagens` do repositorio
