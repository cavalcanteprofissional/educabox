# REGISTRO DE HARDWARE - TV BOXES

> Registro consolidado das especificacoes de hardware das TV boxes do projeto
> (identificacao visual por fotografia da PCB). Atualizado em 2026-07-31
> (revisao 2: dissipador do SoC removido -> chip identificado como S905W).

## SUPER WHITE X (SuperTV / streambus)

### Identificacao do sistema (Android original)

| Campo | Valor |
|-------|:------|
| Nome do dispositivo | SUPER WHITE X (SuperTV) |
| Versao Android | 7.1.2 |
| Software version | 65.09.18 places |
| Serial Number | 7902853145 |
| IP (rede local) | 192.168.100.32 |
| MAC | ee-79-02-85-31-45 |

### Hardware (inspecao visual da PCB - fotos, com dissipador removido)

| Componente | Detalhe |
|------------|:--:|
| Placa Mae (silkscreen) | **S905XQ4_V1.0** (placa CLONE de terceiros, NAO p212 de referencia) |
| Data de fabricacao | 2018.03.20 |
| SoC | **Amlogic S905W** (leitura direta do chip apos remover dissipador; marcacao "M16B1 ou similar") - Quad-Core ARM Cortex-A53 |
| GPU | Mali-450 MP |
| RAM | 2 GB DDR3 (2x 1 GB cada - marca provavel NANYA) |
| eMMC | 8 GB fisicos (ESTIMADO) - chip coberto pela etiqueta branca de fabrica, marca/modelo indeterminados. 4,64 GB visiveis ao SO |
| WiFi | **SSV6051P** (Sigmastar) - 802.11 b/g/n 2,4 GHz **SEM Bluetooth** |
| Ethernet | RTL8201F (Realtek 10/100 Mbps) |
| USB | 2x USB 2.0 |
| Conector OTG | Porta USB-A imediatamente acima do pinhole de reset (USB macho-macho -> PC) |
| Botao reset | Pinhole na borda direita da placa, entre a porta USB-A e a HDMI |

### Observacoes criticas (firmware)

- **Esta NAO e uma placa p212 original nem um S905X**: e um clone
  S905XQ4_V1.0 com SoC **S905W** (familia GXL). ROMs Android para "p212" ou
  "S905X" NAO funcionam (bootloader diferente -> tela preta ou sem boot).
- **Android de reposicao**: usar ROM **S905W** com suporte ao wifi SSV6051P
  (ex.: MXQ Pro 4K / X96 Mini / T95 S1 com SSV6051P, ou custom
  atvXperience/slimBOX). O wifi e secundario para este projeto (usaremos
  Ethernet), mas a ROM precisa bootar na placa.
- **Armbian**: a imagem ophub `amlogic_s905x` cobre a familia GXL inteira
  (S905X/S905D/S905W/S905L) - **a imagem ja baixada permanece valida**. O
  `u-boot-s905x-s912.bin` (-> `u-boot.ext`) tambem serve para S905W.
- **DTB**: manter `meson-gxl-s905x-p212.dtb` como primario (o Android usa o
  dt-id `gxl_p212_2g`, mesmo para S905W; p212 e p281 sao quase identicos).
  Se houver problema de USB/rede no Armbian, trocar por
  `meson-gxl-s905w-p281.dtb` (canonico para S905W).
- **Wifi SSV6051P no Armbian**: depende de driver no kernel ophub
  (a confirmar no boot). Ethernet RTL8201F e o caminho garantido.
- Backup do Android original via `armbian-ddbr` apos o primeiro boot.

### Cronologia da identificacao

1. CPU-Z (Android): reportou S905X / placa p212 / 2GB / eMMC 4.64GB
   (CPU-Z usa o dt-id do firmware: `gxl_p212_2g` - nao reflete o chip fisico)
2. Visao computacional com dissipador: S905X v1 (M14B2) - **incorreta**
   (chip coberto, leitura por inferencia)
3. Remocao do dissipador + visao computacional: **S905W** (M16B1) - **correta**

## Outras boxes (referencia)

| Box | SoC | Placa | RAM | eMMC | WiFi | Ethernet | Obs |
|-----|-----|-------|:---:|------|------|:--:|-----|
| SUPER WHITE X | Amlogic **S905W** | S905XQ4_V1.0 (clone) | 2 GB | ~8 GB est. (etiqueta) | SSV6051P (sem BT) | RTL8201F | 2026-07-31 (fotos, dissipador removido) |
| SUPER TV | Rockchip RK3229 | a complementar | a complementar | a complementar | a complementar | a complementar | ja catalogada no educabox |
| MyTVBox BRAVE 4K | Amlogic S905X (p212) | p212 | a complementar | a complementar | a complementar | a complementar | mesmo DTB p212 da SUPER WHITE X |
