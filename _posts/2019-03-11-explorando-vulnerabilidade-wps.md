---
layout: post
title:  "Explorando a vulnerabilidade do WPS"
date:   2019-03-11 22:50:00 -0300
---



Neste post, vou mostrar como realizar um teste de segurança em redes sem fio, que possuam o WPS ativo. Alguns roteadores possuem uma vulnerabilidade que nos permite realizar um ataque de brute force para encontrar o PIN WPS, uma vez identificado o PIN, é feita a autenticação e o roteador fornece a senha da rede wireless. Para realizar o teste, utilizei meu roteador e criei uma rede apenas para este fim, sendo que, realizar este procedimento em redes alheias não é permitido por lei, muito cuidado com isso!

Para realizar esse teste, configurei o seguinte ambiente:

[Linux Mint 18](https://www.linuxmint.com/)

[PixieWPS](https://github.com/wiire/pixiewps)

[Reaver-wps-fork-t6x](https://github.com/t6x/reaver-wps-fork-t6x)

Bem, para começar, presumo que já esteja com o Mint instalado, então, pode começar a instalar as dependências para o Reaver e PixieWPS.

Instale os seguintes pacotes:

	# apt-get -y install build-essential libpcap-dev aircrack-ng

Depois de instalados os pacotes acima, pode instalar o Reaver e PixieWPS.

### PixieWPS  
Para fazer download do projeto, escolha utilizar um dos comandos abaixo:

	$ git clone https://github.com/wiire/pixiewps

ou

	$ wget https://github.com/wiire/pixiewps/archive/master.zip && unzip master.zip

Feito o download, será preciso executar estes comandos:

	$ cd pixiewps*/
	$ cd src/
	$ make
	$ sudo make install

### Reaver
Para fazer download do projeto, escolha utilizar um dos comandos abaixo:

	$ git clone https://github.com/t6x/reaver-wps-fork-t6x

ou

	$ wget https://github.com/t6x/reaver-wps-fork-t6x/archive/master.zip && unzip master.zip

Feito o download, será preciso executar estes comandos:

	$ cd reaver-wps-fork-t6x*/
	$ cd src/
	$ ./configure
	$ make
	$ sudo make install

Pronto!

---

### Configurações e execução
Agora Vamos alterar o modo de funcionamento da interface wireless.

É necessário identificar a interface wireless, para isso, basta usar o comando:

	$ iwconfig

<!-- ![wlp6s0](/assets/images/1.png "wlp6s0")
 -->
 <figure>
  <img src="{{site.url}}/assets/images/1.png" alt=" "/>
  <figcaption style="display: block; text-align: center;">Neste caso, temos a wlp6s0 como interface wireless.</figcaption>
</figure>

Após identificar a interface wireless, altere o modo de funcionamento para o modo “Monitor”, para isso utilize o comando:

	# airmon-ng start wlp6s0

<figure>
  <img src="{{site.url}}/assets/images/2.png" alt=" "/>
</figure>

Na saída do comando acima, esta descrito entre parenteses a informação de que foi habilitado o modo monitor na interface virtual “mon0”. Além disso, foram encontrados 5 processos que podem causar algum bug ao utilizar a interface em modo monitor, para garantir que não terá nenhum problema, esses processos devem ser encerrados, para isso basta usar o comando kill, seguido dos PID’s dos processos listados.

	# kill 897 898 958 1192 1273

Agora desabilite a interface wireless “wlp6s0”, para que não haja problemas ao identificar a rede alvo, com o comando:

	# ifconfig wlp6s0 down

<figure>
  <img src="{{site.url}}/assets/images/3.png" alt=" "/>
</figure>

Após ter encerrado os processos que podem causar problemas no funcionamento do airmon-ng e desabilitar a interface wireless que não será utilizada, vamos ao próximo passo, localizar a rede alvo. Para isso, utilize o comando “wash” que lista somente redes com WPS ativo.

	# wash -i mon0

<figure>
  <img src="{{site.url}}/assets/images/4.png" alt=" "/>
</figure>

Identificada a rede alvo, passe os parâmetros necessários para executar o Reaver e PixieWPS, com o seguinte comando:

	# reaver -i mon0 -b E8:CC:18:BE:01:AC -c 6 -vvv -K 1

<figure>
  <img src="{{site.url}}/assets/images/5.png" alt=" "/>
</figure>

Os parâmetros passados foram: -i {interface wireless} -b {MAC Adress alvo} -c {Canal de trasmissão}. Os dois últimos, são “-vvv” para ativar o modo verbose, para exibir todas as mensagens de cada etapa do processo e -K 1 que faz com que o Reaver passe os dados referentes ao PKE, PKR, e-hash1 e e-hash2 automaticamente, o pixieWPS fará um brute force “off-line”, evitando que o roteador alvo, bloqueie a conexão por meio do WPS após algumas tentativas, dessa forma serão testados todos os PIN’s. Por rodar o brute force com os dados coletados pelo Reaver, não há necessidade de a cada tentativa fazer a comunicação com o alvo, reduzindo drasticamente o tempo necessário para que o PIN seja encontrado, o tempo necessário em média, não supera os 30 minutos, porém a depender da capacidade de processamento do seu computador, esse tempo pode variar.

Vale lembrar que não são todos os roteadores que possuem essa vulnerabilidade, neste link, há uma lista de dispositivos vulneráveis. Outras distribuições Debian Based são compatíveis com esse tutorial.

P.S.: Realizei esse teste num roteador de minha propriedade, configurado para este fim. Não utilizem este conhecimento para testar em redes na quais não tenham autorização para tal.