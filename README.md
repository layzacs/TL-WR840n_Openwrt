# TL-WR840n_Openwrt
This repository contains a firmware made for TL-WR840n router using Openwrt/Lede project. It's kind of a walkthrough.


## Intro

![Logo OpenWrt](https://imgs.unisec.com.br/docs/openwrt/openwrtlogo.png)

OpenWrt is an open-source project that basically gives a device from [this list](https://openwrt.org/toh/start) (normally routers) the possibility to run a very small linux distribution. Why? Because it's easier to set up configuration when you know what you're dealing with, and people usually knows how to do stuff with linux.
For this specifi case, we needed to configure dozens of routers and the easiet way was with Ansible. For that, we needed to access the router through SSH on WAN.

## História

O projeto OpenWrt iniciou em janeiro de 2004. Originalmente o primeiro hardware compatível foi o Linksys WRT54G (aquele azul de guerra), mas o projeto expandiu rápido e passou a suportar outros fabricantes, incluíndo o Netgear, TP-LInk, Mikrotik, D-Link, ASUS e alguns outros.

O desenvolvimento foi impulsionado inicialmente graças à licença GPL, que obrigava a todos aqueles fabricantes que modificavam e melhoravam o código, a liberar essas mudanças aos demais e contribuir a cada vez mais ao projeto em geral

Pouco a pouco o software foi crescendo e atualmente possui características implementadas que não existem normalmente em outros fabricantes de dispositivos comerciais, que tornam o dispositivo realmente potente e versátil.

## Recursos

Por padrão o OpenWrt permite acesso aos dispositivos via SSH ou interface gráfica (LuCi).

Possibilita também transformar roteadores em servidores de arquivo, nós P2P, servidores de webcams, firewall e sevidores VPN.

[Aqui](https://openwrt.org/about) você pode conhecer a respeito da fusão que ocorreu em Jan/2018 entre OpenWRT e LEDE Projetct. Além dos principais desenvolvedores do projeto.

## Criando o Firmware

As imagens podem ser criadas com o [Buildsystem para distribuição do Linux OpenWRT](https://github.com/lede-project/source).

Quickstart para criação de imagens: https://openwrt.org/docs/guide-developer/quickstart-build-images

Procendimento:

1. Download do código do projeto no github ([Lede Project](https://github.com/lede-project/source));

2. Ter instalado todas as dependências para criação do firmware. Para isso, vá na pasta ``source`` e siga as instruções:
  - Dê o comando "./scripts/feeds update -a" para fazer download de todos os pacotes e definições encontradas no arquivo feeds.conf do repositório do projeto.
  - Dê o comando "./scripts/feeds install -a" para instalar symlinks para instalar todos os pacotes instalados anteriormente que se encontram na pasta ``package/feeds/``.

3. Agora que todas as dependências foram instaladas, na pasta source do projeto dê o comando ``make menuconfig``, que abrirá um menu para escolher dispositivo, drivers, pacotes e funcionalidades que farão parte do firmware. Um arquivo será gerado a partir das opções escolhidas. Você deve nomeá-lo ``.config``. Você pode duplicar, mudar o nome deste arquivo e salvá-lo em algum lugar seguro para usar futuramente, se for necessário.

4. Dê o comando ``make V=s`` e espere (muito tempo). Leva por volta de 30 minutos e **é necessário ter pelo menos 15GB de espaço no disco**. É muito importante que o comando **não seja executado como ROOT**. Certifique-se que todo conteúdo da pasta source esteja com as devidas permissões ao seu usuário local (não root) com o comando ``ls -l``

Até agora você aprender a criar uma imagem com todas as possíveis modificações que podem ser feitas com o ``make menuconfig``. Mas, talvez seja necessário uma configuração padrão de rede ou wifi dos seus roteadores.

## Adicionando Arquivo de Configuração no Firmware

Utilizando o OpenWrt é possível modificar as configuraões iniciais direto na imagem, inserindo no firmware os arquivos de configuração já editados. Isto possibilita um menor uso do armazenamento interno do roteador, e caso o dispositivo seja retornado ao padrão de fábrica, configurações contidas direto no firmware não são perdidas.

Para adicionar arquivos ao firmware:

1. Copie os arquivos para a pasta ``source/files``. Por exemplo: Queremos uma configuração padrão de wireless específica. Criamos um arquivo wireless baseado no que é encontrado em uma imagem anterior, do mesmo dispositivo. O arquivo, no sistema do roteador, pode ficar em ``/etc/config/``, então basta colocar o arquivo na pasta ``source/files/etc/config/wireless``. É importante que você tenha certeza de que está colocando a imagem no lugar correto.

## Outras configurações Interessantes

### Possibilitando SSH via WAN

OBS: Após a queima do firmware é preciso reiniciar o roteador para acessar via WAN.

1. Crie uma nova regra no firewall para aceitar SSH na WAN:

```
# Allow SSH via WAN
config rule
	option name             Allow-SSH-WAN
	option src              wan
	option proto            tcp
	option dest_port        22
	option target           ACCEPT
 ```
2. Adicione as chaves SSH públicas para acesso no arquivo ``/etc/dropbear/authorized_keys``. É importante que o arquivo eteja com as permissões corretas. Para isso, a pasta dropbear precisa estar com permissões 700 e o arquivo authorized_keys com a permissão 600.

3. Configure o Dropbear para não aceitar autenticação por senha quando é feito SSH:
```
# No password auth for ssh on wan
config dropbear
        option Port '22'
        option RootPasswordAuth 'off'
        option PasswordAuth 'off'

config dropbear
        option RootPasswordAuth 'off'
        option PasswordAuth 'on'
        option Port '22'
        option Interface 'lan'
```
### Mudando a senha padrão de Root

First set the password in a live router, then copy /etc/passwd and /etc/shadow from the router to your build environment and include them in the firmware build as custom files.

https://lede-project.org/docs/guide-developer/use-buildsystem#custom_files 86

https://forum.openwrt.org/viewtopic.php?pid=339453#p339453 57

### Para desativar o botão de reset:

Remover a linha do script /etc/rc.button/reset:
```
jffs2reset -y && reboot &
```

Dessa forma o botão de reset para de funcionar totalmente.

### Possíveis mensagens de erro durante o ``make V=s``

Relacionado a tar: Mudar as permissõs da pasta source do projeto LEDE. TUDO dentro do diretório source deve ter o usuário não-root como proprietário.

Relacionado a cmake: ????  - provavelmente faltava espaço em disco.
Mensagem de erro: ``make[2]: *** [tools/Makefile:155: tools/cmake/compile] Error 2``

## Referências

- [https://pt.wikipedia.org/wiki/OpenWrt](https://pt.wikipedia.org/wiki/OpenWrt)

- Quickstart para criação de imagens: https://openwrt.org/docs/guide-developer/quickstart-build-images

