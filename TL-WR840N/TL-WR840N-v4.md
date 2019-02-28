# TL-WR840N v4

Firmware versão 4 para roteadores tp-Link TL-WR840N v4.

O firmware foi criado usando o source do projeto LEDE (https://github.com/lede-project/source).

- Target: MediaTek Ralink MIPS
- Subtarget: MT7628 based boards
- Device: TP-Link WR841N v13 or TP-Link WR840N v4

## Essa imagem posssui:

- Compatibilidade IPv6
- LuCi
- Wireguard
- nano
- Arquivos de configuração editados
- SSH via WAN liberado
- Chaves SSH públicas para acesso

## Configurações Inseridas

### Firewall
No arquivo ``/etc/config/firewall``:

```
# Allow SSH via WAN
config rule
	option name             Allow-SSH-WAN
	option src              wan
	option proto            tcp
	option dest_port        22
	option target           ACCEPT
```

As chaves SSH inseridas no dispositivo para possibilitar acesso estão no arquivo ``/etc/dropbear/authorized_keys``

### Rede e Wireless
No arquivo ``/etc/config/wireless``:

```
config wifi-iface 'default_radio0'
	option device 'radio0'
	option network 'lan'
	option mode 'ap'
	option ssid 'tp_link'
	option encryption 'psk2+aes'
	option key '********'            #informação sensivel!
```

No arquivo ``/etc/config/network``:

```
config interface 'lan'
	option type 'bridge'
	option ifname 'eth0.1'
	option proto 'static'
	option ipaddr '10.0.0.1'
	option netmask '255.255.255.0'
	option ip6assign '60'
```

Você pode ver os arquivos de configuração inteiros na pasta ``files/etc/`` desde repositório.

## Espaço Alocado em Disco

```
root@OpenWrt:~# df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/root                 3.5M      3.5M         0 100% /rom
tmpfs                    29.3M    448.0K     28.9M   1% /tmp
/dev/mtdblock4            2.8M    224.0K      2.5M   8% /overlay
overlayfs:/overlay        2.8M    224.0K      2.5M   8% /
tmpfs                   512.0K         0    512.0K   0% /dev
```

### Procedimento

Faça o download do código do projeto no github ([Lede Project](https://github.com/lede-project/source)).

O procedimento para a criaçao do firmware que se encontra neste repositório começa com o arquivo .config, que é responsável por definir o dispositivo, drivers que farão parte do firmware, pacotes e funcionalidades. Para replicar este firmware, faça download do arquivo ``.config`` deste repositório e coloque-o na pasta ``source`` do projeto Lede. Você pode gerar um novo arquivo .config com modificações dando o comando ``make menuconfig``.

Antes de queimar a imagem, é necessário colocar os arquivos referentes as chaves SSH públicas, configuração padrão de firewall, WiFi e VPN. Cada uma delas se encontra como abaixo:

**Considere ``source`` como a raiz da pasta do projeto LEDE baixado do github.**

``source/files/etc/config/firewall`` - Arquivo de configuração do firewall

``source/files/etc/config/wireless`` - Arquivo de configuração wifi

``source/files/etc/config/network`` - Arquivo de configuração da rede

``source/files/etc/dropbear/authorized_keys`` - Chaves SSH públicas para acesso. A pasta e o arquivo devem ter as permissões 0700 e 0600, respectivamente, antes de queimar a imagem.

Na pasta do projeto dar o comando ``make menuconfig`` e configurar de acordo com o dispositivo e pacotes desejados OU usar o arquivo .conf encontrado nesse repositório, para replicar corretamente os firmwares que se encontram neste repositório. Para isso, basta colocá-lo na pasta raiz do repositório do projeto LEDE.

Dar o comando ``make V=s`` e esperar (muito tempo). Leva por volta de 30 minutos e **é necessário ter pelo menos 15GB de espaço livre no disco**.

Após terminado, os arquivos recovery e sysupgrade estarão na pasta source/bin/targets/ramips/mt76x8.

Para queimar o firmware no dispositivo TP-Link TL-WR840n v4, você precisará de um servidor TFTP rodando em sua máquina, configurado no IP 192.168.0.66, na porta 69. Insira o arquivo do firmware desejado dentro da pasta do servidor TFTP e reproduza as etapas abaixo:
- Suba o servidor TFTP com o arquivo do firmware nomeado de ``tp_recovery.bin``, se o nome do arquivo não for este ele não será baixado pelo dispositivo;
- Conecte o roteador à maquina onde o servidor TFTP está rodando;
- Desligue o roteador e ligue-o segurando o botão de reset por 5-6 segundos (até os leds piscarem). Ele irá automaticamente buscar pelo servidor e fazer download do arquivo de firmware.
Após o download do arquivo, o dispositivo irá automaticamente iniciar o upgrade.

OBS: tenha em mente que caso o dispositivo esteja com o firmware de fábrica, você usará o arquivo de firmware terminado em recovery.bin.
Para um dispositivo que já possui uma imagem OpenWRT, você deve usar o arquivo terminado em sysupgrade.bin.
