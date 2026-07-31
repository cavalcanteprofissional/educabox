# TV BOX SUPER WHITE X

## Hardware

| Sumario | Detalhes |
|---------|:--:|
| Codename | SUPER WHITE X |
| Fabricante | SuperTV / streambus |
| Modelo | SUPER WHITE X |
| Placa Mae | p212 |
| Placa DTB | glx_p212_2g |
| CPU | Amlogic S905X |
| Familia | Cortex-A53 |
| Velocidade | 100 - 1512 MHz |
| GPU | Mali-450 MP |
| Wifi | a confirmar |
| Memoria | 2GB |
| Armazenamento | eMMC (4.64GB visivel) |
| Resolucao | 1920x1080 |

> Dados obtidos via app CPU-Z no Android original. Placa p212 (mesma do
> MyTVBox BRAVE 4K). Diferente da SUPER TV (Rockchip RK3229) ja catalogada
> neste repositorio: esta e a variante Amlogic S905X.

## Sistema Operacional

| S.O | Kernel | Versao | Interface | Download |
|---------|:------:|:------:|:---------:|:--------:|
| Armbian (ophub) | 6.18.38 | 26.08.0 | Server | [.img.gz](https://github.com/ophub/amlogic-s9xxx-armbian/releases/download/Armbian_bookworm_arm64_server_2026.07/Armbian_26.08.0_amlogic_s905x_bookworm_6.18.38_server_2026.07.16.img.gz) |

**DTB** = `meson-gxl-s905x-p212.dtb` (ja definido em `/boot/uEnv.txt` na imagem)

## Servicos Ativos/Inativos

- [OK] CPU
- [OK] GPU/HDMI
- [ ] USB 2.0 (a confirmar no boot)
- [ ] WIFI (a confirmar)
- [OK] ETHERNET
- [ ] AUDIO (a confirmar)

## Guia de Instalacao Imagem Oficial Armbian

O guia abaixo descreve a instalacao de uma imagem limpa da
[Imagem Oficial Armbian](https://github.com/ophub/amlogic-s9xxx-armbian) com
os parametros necessarios para que o Armbian seja instalado corretamente na
TV Box SUPER WHITE X (Amlogic S905X / p212 / 2GB).

### 1. Pre-requisitos

1. Dispositivo USB (Pendrive) - minimo 8GB
2. Software [balenaEtcher](https://etcher.balena.io/), [Rufus](https://rufus.ie/pt_BR/)
   ou [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/)
3. Imagem Oficial S.O Armbian para Amlogic S905X (release `amlogic_s905x`)
4. [Platform Tools (adb)](https://developer.android.com/tools/releases/platform-tools)
   no computador (necessario para o `adb shell reboot update`)

### 2. Gravacao da Imagem

1. Execute o balenaEtcher
2. Grave a imagem Armbian no Pendrive USB
3. Remova com seguranca o Pendrive e insira-o novamente
4. Ignore/Feche as caixas de dialogo do Windows/MacOS para formatar o Pendrive
   inserido. A particao legivel (BOOT/FAT32) sera montada.

### 3. Configurar o Pendrive para Armbian

1. Abra a particao legivel do Pendrive no Windows Explorer (Ex: `BOOT`)
2. Copie/renomeie o arquivo apropriado para `u-boot.ext` na raiz do Pendrive:

   - `u-boot-s905x-s912` (para s905x, s905w e s912) -> renomeie/copie para `u-boot.ext`

   > A imagem 26.08.0 do ophub nao inclui o `u-boot.ext` na particao BOOT.
   > O script `s905_autoscript` (carregado via `aml_autoscript` pelo u-boot do
   > Android no modo update) executa `fatload usb 0 0x1000000 u-boot.ext; go`,
   > encadeando o u-boot mainline que carrega o kernel via `/boot/uEnv.txt`.

3. Confira o arquivo `/boot/uEnv.txt`:

   - `LINUX=/zImage`
   - `INITRD=/uInitrd`
   - `FDT=/dtb/amlogic/meson-gxl-s905x-p212.dtb`

   > Para esta placa (p212) o uEnv.txt ja vem correto na imagem ophub.
   > Nao renomeie `/boot/extlinux/extlinux.conf.bak` para `extlinux.conf`
   > (necessario apenas para T95/T95Z-Plus/R3300L).

4. Remova o Pendrive com seguranca

### 4. Inicializando o Armbian pela primeira vez via Pendrive

Metodo recomendado (via adb de rede):

1. Conecte o Pendrive na porta USB da TV Box
2. Conecte a TV Box na rede (Ethernet no switch ou WiFi) e descubra o IP dela
   no roteador ou via varredura ARP
3. No computador: `adb connect <IP_DO_DEVICE>:5555`
4. No computador: `adb shell reboot update`
5. O sistema reinicia e inicializa o Armbian a partir do Pendrive
6. Ao ser solicitado, faça login com `usuario: root` `senha: 1234`

> Nota: o `adb` de rede costuma funcionar nessas boxes mesmo sem habilitar
> "Depuracao USB" nas opcoes do desenvolvedor (o adbd escuta na porta 5555
> por padrao em muitas firmwares de fabrica).

Metodo alternativo (botao reset/update na TV Box):

1. Conecte o Pendrive na TV Box
2. Conecte o adaptador de energia na TV Box
3. Com um clips acione o botao `reset` (ou o oculto na entrada `AV`, quando
   existir) por alguns segundos ate a tela apagar e reiniciar o equipamento
4. O sistema reinicia e inicializa o Armbian a partir do Pendrive

> Nesta placa o botao `reset` abriu o menu do u-boot Android (SuperTV CLI) e
> nao disparou o boot pela midia: a imagem ophub 26.08 nao grava bootloader
> no setor 0 da midia (apenas MBR + particoes), entao o bootrom nao tem o que
> carregar. Detalhes no [diagnostico tecnico](docs/diagnostico-tecnico.md).

### 5. Instalacao Armbian Pendrive para eMMC

1. Depois que o sistema inicializar, abra o terminal
2. No terminal digite: `sudo armbian-install`
3. Selecione a opcao para instalar no armazenamento interno eMMC
4. Confirme a formatacao do eMMC (recomendado `ext4`)
5. Remova o Pendrive com seguranca
6. Reinicie a TV Box

> Faca antes um backup do Android original com `armbian-ddbr` (opcao `b`),
> salvo em `/ddbr/BACKUP-arm-64-emmc.img.gz`, para possibilitar a restauracao.
