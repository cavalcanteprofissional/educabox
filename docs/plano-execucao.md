# Plano de Execucao - SUPER WHITE X (S905W / S905XQ4_V1.0)

Status atual de cada passo ao final deste documento.

## Passo 1 - Preparar o pendrive

- [x] Baixar imagem: `Armbian_26.08.0_amlogic_s905x_bookworm_6.18.38_server_2026.07.16.img.gz`
- [x] Conferir SHA256: `0a373487e510c3f81f01d6250a17fc474e9d2fc922d5be0950542c4ad88ec304`
- [x] Gravar com balenaEtcher no SanDisk Cruzer Blade 8GB
- [x] Verificar particoes (BOOT FAT32 + ext4) - valido
- [x] Criar `u-boot.ext` na BOOT (E:) = copia de `u-boot-s905x-s912.bin`

## Passo 2 - Obter o adb (platform-tools)

- [x] Baixar platform-tools do Google (Windows) - `tools/platform-tools`
- [x] Testar `adb version` (37.0.1)

## Passo 3 - Conectar a box na rede

- [x] Conectar a TV Box ao switch via cabo Ethernet (computador ja esta no switch)
- [x] Descobrir o IP da box: PC `192.168.100.29` / box `192.168.100.32`
      (MAC `ee-79-02-85-31-45`)

## Passo 4 - Boot via adb (caminho recomendado)

1. Inserir o pendrive na porta USB da box
2. `adb connect <IP>:5555`
3. Se conectar: `adb shell reboot update`
4. Observar: logo Armbian / texto de boot no HDMI

- [ ] Conectar via adb de rede
- [ ] Disparar `adb shell reboot update`
- [ ] Boot do Armbian no pendrive

## Passo 5 - Instalacao para eMMC

1. Login: `root` / `1234`
2. Backup do Android original: `armbian-ddbr` (opcao `b`)
3. `sudo armbian-install` -> instalar no eMMC (ext4)
4. Remover o pendrive e reiniciar

## Fallbacks (se o passo 4 falhar)

### F1 - App Update local da box
O fluxo `aml_autoscript` pode ser disparado pelo app de atualizacao local do
Android. Na SUPER WHITE X o "Local Update" nao reconheceu o pendrive (nao
funcionou nesta box). Requer `aml_autoscript.zip` com estrutura correta.

### F2 - USB Burning Tool (metodo OTG, usado pelo educabox)
Para boxes que nao inicializam SD/pendrive na etapa de boot, o educabox
documenta a injecao via cabo USB OTG (macho-macho) + botao reset / short
circuit nos pontos de contato da placa:

1. Instalar [USB Burning Tool](https://github.com/ophub/kernel/releases/tag/tools)
   (versao 2.2.4 - melhor compatibilidade; v2.0.8 ou superior serve)
2. Preparar cabo USB-A macho-macho
3. Importar imagem (arquivo `.img` descompactado) no USB Burning Tool
4. Desconectar a energia, segurar `reset`, conectar o cabo OTG ao computador
5. Iniciar a gravacao e aguardar 100%
6. Reiniciar a box - Armbian instalado direto no eMMC

> ATENCAO (placa clone): a imagem `.img` deve ser de ROM **S905W** com
> suporte ao wifi SSV6051P (NAO S905X/p212). Marcar "Erase Flash" e "Erase
> Bootloader"; nao usar "Overwrite key". Detalhes em `docs/specs-hardware.md`.

### F3 - Console serial (UART)
Cabo USB-TTL nos pinos UART da placa para interagir com o u-boot no boot.

## Referencias

- ophub README secao 12.4.1 (instalacao inicial via adb): `adb connect` + `adb shell reboot update`
- educabox `boxes/mytvbox.md` (S905X p212): u-boot-s905x-s912 -> u-boot.ext
- educabox `boxes/supertv.md` secao OTG: injecao via cabo USB OTG quando a box nao boota de SD/pendrive
