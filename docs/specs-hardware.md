# REGISTRO DE HARDWARE - TV BOXES

> Registro consolidado das especificacoes de hardware das TV boxes do projeto.
> Atualizado em 2026-08-01 (revisao 3: leitura macro da PCB por IA identifica
> S905X / HAM905X1A0-B / RTL8723BS / eMMC Toshiba BGA).

## SUPER WHITE X (SuperTV / streambus)

### Identificacao do sistema (firmware atual)

| Campo | Valor |
|-------|:------|
| Nome do dispositivo | SUPER WHITE X (SuperTV) |
| Versao Android | **6.0.1** (revisado; antes registrado 7.1.2) |
| Build | MHC19J / 20210318 |
| Board id / target | SuperTV / p212 / p212 |
| Software version | 65.09.18 places |
| Serial Number | 7902853145 |
| IP (rede local) | 192.168.100.32 |
| MAC | ee-79-02-85-31-45 |

### Hardware (leitura macro da PCB por IA + inspecao fisica previa)

| Componente | Detalhe |
|------------|:--:|
| Placa Mae (silkscreen) | **HAM905X1A0-B V4 2017-04-1** (nome interno da PCB; "S905XQ4_V1.0" era referencia generica de clone) |
| SoC | **Amlogic S905X** (familia GXL; lote `J-T3YH14.OG` / `D0LXCN09808`). Obs.: o mesmo die (M16B1) aparece em S905X e S905W - leitura anterior "S905W" foi por inferencia |
| GPU | Mali-450 MP |
| RAM | **4x Nanya NT5CC256M16EP-EK** (DDR3 256Mx16, 4Gbit cada) = **2GB** |
| eMMC | **Toshiba/Kioxia THGBMFG6C1LBAIL** (lote 1917KAE) - **encapsulamento BGA**, sem pinos laterais acessiveis. **Short fisico NAO viavel** |
| WiFi/Bluetooth | **Realtek RTL8723BS** (combo SDIO, 2.4GHz b/g/n + **Bluetooth 4.0**). Correcao do SSV6051P |
| Ethernet | PHY integrado ao SoC S905X + transformador **AE-SB1600+** (2009H) no RJ45 (sem PHY discreto) |
| USB | 2x USB 2.0 |
| Conector OTG | Nao confirmado visualmente qual das 2 USB-A e OTG (silkscreen ilegivel). Pista: o outro porta USB-A (nao o acima do pinhole) entra no recovery CLI com reset |
| Botao reset | Pinhole na borda direita da placa, entre a porta USB-A e a HDMI |
| Header J3 | 4 pinos (perto do SoC, entre WiFi/BT e indutor `4R7`). **Suspeita forte: UART TX/RX/GND/VCC** - pinagem NAO confirmada, precisa multimetro |
| Header IR | 3 pinos `IR / GND / 3.3V` (sensor IR ja soldado). NAO e UART, nao usar para debug |

### Observacoes criticas (firmware)

- **SoC = S905X** (familia GXL). O die M16B1 e compartilhado entre S905X e S905W
  (o mesmo GXL, binning diferente) - por isso o CPU-Z reportava p212 e a leitura
  fisica anterior apontou S905W. A leitura macro identifica S905X.
- **ROM de reposicao deve ser S905X** (NAO S905W): a MXQ Pro S905W baixada fica
  arquivada como referencia. Usar ROM p212/S905X com suporte ao wifi RTL8723BS,
  ou atvXperience v4.x S905X.
- **Wifi RTL8723BS no Armbian**: driver Realtek a confirmar no boot (Ethernet e o
  caminho garantido).
- **Armbian**: imagem ophub `amlogic_s905x` cobre a familia GXL inteira
  (S905X/S905D/S905W/S905L) - **a imagem ja baixada permanece valida**.
- **eMMC BGA = sem short**: recovery via USB OTG (maskrom) ou debug UART (J3).
- Backup do Android original via `armbian-ddbr` apos o primeiro boot.

### Cronologia da identificacao

1. CPU-Z (Android): reportou S905X / placa p212 / 2GB / eMMC 4.64GB
2. Inspecao fisica com dissipador removido: inferencia "S905W (M16B1)"
   - M16B1 e o die GXL compartilhado - nao distingue S905X de S905W
3. **Leitura macro da PCB por IA (2026-08-01)**: S905X (lote J-T3YH14.OG),
   placa HAM905X1A0-B V4, wifi RTL8723BS, eMMC Toshiba BGA - revisao 3
4. Confirmacao definitiva pendente: u-boot/UART imprime o modelo no boot

## Outras boxes (referencia)

| Box | SoC | Placa | RAM | eMMC | WiFi | Ethernet | Obs |
|-----|-----|-------|:---:|------|------|:--:|-----|
| SUPER WHITE X | Amlogic **S905X** | HAM905X1A0-B V4 (clone) | 2 GB (4x Nanya) | Toshiba THGBMFG6C1LBAIL ~8GB (BGA) | RTL8723BS (2.4GHz + BT4.0) | PHY integrado + AE-SB1600+ | 2026-08-01 (macro IA + fisico) |
| SUPER TV | Rockchip RK3229 | a complementar | a complementar | a complementar | a complementar | a complementar | ja catalogada no educabox |
| MyTVBox BRAVE 4K | Amlogic S905X (p212) | p212 | a complementar | a complementar | a complementar | a complementar | mesmo DTB p212 da SUPER WHITE X |
