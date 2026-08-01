# Diagnostico Tecnico - SUPER WHITE X (Amlogic S905W / S905XQ4_V1.0)

## 1. Contexto

TV Box SuperTV "WHITE X" com Amlogic S905W (placa clone S905XQ4_V1.0, 2GB).
Objetivo: instalar Armbian inicializando a partir de midia removivel
(pendrive/SD) e depois gravar no eMMC.

Sintoma inicial: qualquer tentativa de boot via botao `reset` abria o menu
CLI do u-boot Android (SuperTV) e o sistema voltava ao Android. Nenhuma
midia removivel era inicializada.

## 2. Hardware identificado (CPU-Z + inspecao fisica)

- CPU: Amlogic S905W (4x Cortex-A53, 100-1512 MHz) - o CPU-Z do Android
  reportava "S905X/p212" (usa o dt-id do firmware `gxl_p212_2g`); a leitura
  direta do chip apos remover o dissipador confirmou S905W (marcacao M16B1)
- Placa: CLONE S905XQ4_V1.0 (2018.03.20) - NAO e a p212 de referencia
- GPU: Mali-450 MP
- RAM: 2GB DDR3 (2x1GB, NANYA provavel)
- eMMC: ~8GB fisicos estimados (4.64GB visivel; chip coberto por etiqueta)
- Wifi: SSV6051P (2.4GHz, sem BT)
- Ethernet: RTL8201F (10/100)
- DTB: `meson-gxl-s905x-p212.dtb` (dt-id do Android: `gxl_p212_2g`);
  alternativa S905W: `meson-gxl-s905w-p281.dtb`

## 3. Imagem utilizada (validada)

- Arquivo: `Armbian_26.08.0_amlogic_s905x_bookworm_6.18.38_server_2026.07.16.img.gz`
- Fonte: ophub/amlogic-s9xxx-armbian, release `Armbian_bookworm_arm64_server_2026.07`
- SHA256: `0a373487e510c3f81f01d6250a17fc474e9d2fc922d5be0950542c4ad88ec304`
- Tamanho: 826.470.689 bytes
- Gravacao: balenaEtcher em SanDisk Cruzer Blade 8GB (Flash Complete)

## 4. Estrutura da imagem (verificada a partir do arquivo .img.gz)

### 4.1 Tabela de particoes (MBR no setor 0)

| Particao | Tipo | Inicio (LBA) | Inicio (byte) | Tamanho |
|----------|------|:---:|:---:|:---:|
| 0 | 0x0C FAT32 LBA | 8192 (4MB) | 0x400000 | 1.044.480 setores (~510MB) |
| 1 | 0x83 ext4 | 0x102000 (~518MB) | - | resto do disco |

Assinatura MBR `0x55AA` valida. A particao 0 e a `BOOT` (montada como unidade
E: no Windows).

### 4.2 Sem bootloader no setor 0 (causa raiz)

O espaco entre o MBR (setor 0) e o inicio da particao 0 (4MB) esta
inteiramente preenchido com zeros. A varredura dos primeiros 4MB pela
assinatura dos binarios de u-boot (`0a 00 00 14 1f 20 03 d5`) nao encontrou
nada.

Consequencia: o bootrom da S905X, que procura um bootloader nos primeiros
setores da midia removivel, nao encontra nada e cai no u-boot do eMMC
(Android) -> menu SuperTV. Por isso o metodo do botao `reset` nunca funcionou
nem com a SD antiga (que era apenas arquivos copiados, sem imagem bruta).

### 4.3 Arquivos de boot na particao BOOT (E:)

- `uEnv.txt` (ativo, configuracao padrao ophub para p212):
  - `LINUX=/zImage`
  - `INITRD=/uInitrd`
  - `FDT=/dtb/amlogic/meson-gxl-s905x-p212.dtb`
  - `APPEND=root=UUID=... rw ...`
- `extlinux/extlinux.conf.bak` (INATIVO - nao renomear; extlinux e necessario
  apenas para T95/T95Z-Plus/R3300L)
- `u-boot-s905x-s912.bin` (646.455 B, hash `207cdbe1...`) - u-boot mainline
  para S905X/S905W/S912, usado como `u-boot.ext` (receita educabox)
- `u-boot.usb` / `u-boot.sd` (709.768 B cada; diferem em apenas 11 bytes -
  ordem de busca usb/mmc). Variante USB para boot por pendrive
- `s905_autoscript` (binario): executa `fatload usb 0 0x1000000 u-boot.ext; go
  0x1000000` (encadeia o u-boot mainline) e, se ausente, tenta boot direto
  via `uEnv.txt` -> `booti`

### 4.4 `u-boot.ext` ausente na imagem 26.08.0

A imagem ophub 26.08.0 NAO inclui `u-boot.ext` na particao BOOT, embora o
`s905_autoscript` o procure. O Educabox documenta exatamente esse passo:
renomear `u-boot-s905x-s912` para `u-boot.ext` (S905X/S905W/S912). A criacao
desse arquivo e necessaria para o encadeamento robusto via modo update.

## 5. Fluxo de boot correto (caminho via Android)

1. Dispara-se o modo update: `adb shell reboot update` (ou app Update local)
2. O u-boot do Android (eMMC) le `aml_autoscript` da particao FAT do pendrive
3. `aml_autoscript` define o `bootcmd` e reinicia
4. `s905_autoscript` carrega `u-boot.ext` do pendrive e salta para ele (`go`)
5. O u-boot mainline carrega `uEnv.txt`, `zImage`, `uInitrd` e o DTB p212
6. `booti` inicia o kernel Armbian (rootfs na particao ext4 do pendrive)

## 6. Notas sobre a tentativa com o botao reset

- O bootrom da S905W/S905X varre SD e porta USB OTG (modo burning), nao a porta
  USB-A do pendrive
- A imagem 26.08.0 nao grava bootloader no inicio da midia
- Logo, mesmo com pendrive gravado via balenaEtcher, o botao `reset` abre o
  menu SuperTV (u-boot do eMMC)
- A rota SD so seria viavel com `u-boot.sd` gravado no setor 0 do cartao e se
  o bootrom da placa varresse o slot de SD (nao confirmado nesta placa)

## 7. Conclusao

O caminho recomendado e o metodo oficial do ophub (secao 12.4.1): adb de rede
(`adb connect <IP>:5555` + `adb shell reboot update`) com o `u-boot.ext`
presente na particao BOOT. Fallbacks documentados no plano de execucao.
