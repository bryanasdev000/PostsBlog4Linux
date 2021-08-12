# Jitsi: Uma solução aberta completa para videoconferências

![Logo](./banner.png)

## Introdução

Durante o momento que vivemos, basicamente todas nossas interações se tornaram virtuais, com o home office sendo a alternativa primaria para muitas empresas, tivemos um súbito crescimento (também conhecido como BOOM) em soluções que visam aproximar as pessoas, sejam ferramentas de chat, colaborativas ou de videoconferência.

O Jitsi é uma solução para o último caso, com mais de 15 anos de atividade, ele é uma solução completa para videoconferências, de código aberto, fácil implementação, segura, robusta, extensível, escalável e multiplataforma, baseado em tecnologia de ponta e capaz de ser integrado com diversas ferramentas, oferecendo uma solução de videoconferência superior, podendo iniciar rapidamente uma reunião, sem cadastro e sem burocracias no [meet.jit.si](https://meet.jit.si/) ou tendo seu próprio ambiente, não devendo em nada em relação a funcionalidades para ferramentas como o Google Meet ou o Microsoft Teams, possuindo como principais apoiadores a 8x8 e a Atlassian além de uma grande comunidade.

O Jitsi possui um uso eficiente de recursos, sendo que uma única instância consegue servir dezenas de usuários simultaneamente, sua arquitetura possibilita também o fácil escalonamento para atender a demanda da sua organização de forma precisa e uma vez implantado demanda pouca manutenção, possuindo assim uma excelente relação de custo x benefício.

## Arquitetura

O Jitsi é, na verdade, uma série de componentes, cada um com uma responsabilidade específica e que juntos formam a solução que conhecemos como Jitsi.

De forma resumida, esses são os componentes do Jitsi:

- Apache ou NGINX
  - Responsável por servir o Jitsi Meet.
  
- Jitsi Meet
  - Frontend do Jitsi
  - Aplicativo mobile (Android/iOS)
  
- Jitsi Videobridge
  - Responsável por cuidar das transmissões de áudio e vídeo
  
- Jicofo
  - Coordenação das transmissões
  
- Jigasi (SIP)
  - Ponte para o protocolo SIP
  
- Jibri
  - Responsável por efetuar gravações e streaming das reuniões
  
- Prosody
  - Servidor XMPP que controla as salas

### Alta disponibilidade e elasticidade

O Jitsi tem uma excelente capacidade de escalar, saindo de uma máquina que atende dezenas de usuários para uma solução robusta e que funciona em escala global, o que o torna indicado para diferentes variações de ambientes, desde startups até empresas multinacionais.

Como o Jitsi é basicamente stateless, isso é, não guarda estado (com uma possível exceção para o método de autenticação escolhido), ele possui uma elasticidade muito grande, um ponto que ajuda bastante na parte de custos em especial para ambientes em cloud.

Atualmente o Jitsi possui 3 principais métodos para escalar:

- JVB bridges

Esse método é o mais simples e consiste em desacoplar a JVB da máquina do Jitsi, dedicando uma ou mais máquinas somente a JVB. Como esse é o componente principal para manipulação do aúdio e vídeo escalar ele garante que sua implementação atenda mais usuários por sala sem ter problemas de desempenho.

Como a JVB é puramente stateless, ela permite facilmente o autoscaling para atender o negócio e prover um bom custo x benefício.

- Sharding

Esse tipo de implementação tem foco em "fragmentar" (sharding) a sua instalação, em vez de possuir uma única máquina ou aumentar as bridges, se escala a instalação por completo, tendo vários "silos" de Jitsi e colocando um balanceador de carga entre eles.

Além disso, é possível mesclar essa configuração com a anterior, tendo várias instalações e cada uma com múltiplas bridges.

Essa implementação consegue atender grandes números de usuários sem problemas, no entanto, é preciso tomar cuidado no como as requisições vão chegar, pois, não existe um "sinalizador" conhecido por todos, ou seja, cada instalação coordena suas salas, se o balanceador de carga envia a requisição para uma instalação que não tem determinada sala, seus usuários vão se desencontrar.

- OCTO bridges

Essa é uma implementação um pouco diferente, o objetivo principal dela é ligar usuários entre diversas regiões do mundo mantendo uma boa performance, ela consiste basicamente de ter instalações regionais que se interligam, ou seja, em vez de um usuário na Ásia ter que conectar na América do Sul para entrar em uma reunião ele se conecta em uma instalação regional e essa instalação se conecta na América do Sul, garantido que o controle de latências e performance fique a cargo do servidor e não do cliente.

Essa implementação é especialmente útil em regiões do mundo que o acesso à Internet é restrito de alguma maneira, evitando ter que adicionar mais componentes de rede, como uma VPN, que podem prejudicar a experiência do usuário durante as reuniões.

A implementação do OCTO é a que trabalha primariamente com o Jicofo e pode possuir variações que incluem os métodos anteriores.

## Módulos e integrações

Pela sua arquitetura e por ser uma solução de código aberto, o Jitsi tem uma flexibilidade muito grande para se adaptar, é possível escrever módulos em Lua que desempenham papeis importantes em integrações com outras ferramentas, além de permitir customizações que melhorem a experiência do usuário.

Na 4Linux, por exemplo, temos módulos que coordenam a frequência do aluno, assim como gravações e funções de administração básicas, tendo sua base primaria em módulos do Jitsi (Prosody) e microsserviços escritos em Golang e Python.

Um dos pontos mais interessantes do Jitsi é sua integração com outras ferramentas, podemos destacar o uso dele com plataformas de chats de código aberto como o Rocket.Chat e Mattermost, integrando videoconferências diretamente na sua ferramenta de chat preferida.

Além disso, no nosso contexto atual, ele é uma excelente ferramenta para o uso em aulas remotas ao vivo, já que possui integração com vários LMS, como o Moodle, possibilitando uma experiência agradável durante as aulas e permitindo aluno e professor focar no que é importante, o aprendizado.

## Segurança

O Jitsi é uma das poucas soluções que tem em mente como prioridade a segurança do usuário, possuindo um alto nível de segurança por padrão e que pode ser melhorado tanto através de configurações no servidor, como pelos clientes através do uso de criptografia de ponta a ponta, ou seja, como empresa, possui-se controle completo dos dados que são transmitidos durante suas reuniões, além do uso de funcionalidades como a gravação das reuniões que permitem efetuar auditorias se necessário.

Por padrão a instalação do Jitsi é aberta, no entanto, é possível escolher entre diferentes tipos de autenticação, tendo o token JWT e o LDAP como mais utilizados.

Possuir a infraestrutura de comunicação da empresa em poder direto dela é especialmente útil com assuntos como a GDPR/LGPD batendo na porta.

## Deploy

O Jitsi tem como meios de instalação suportados o .deb onde suporta oficialmente o Debian 9 e Ubuntu 18.04 em diante e containers através de um docker compose oficial, possui também suporte da comunidade para instalações no NixOS e no Kubernetes (e Openshift).

A parte de requisitos de hardware varia bastante, pois existem N fatores que afetam o consumo do hardware, como quantidade de usuários com câmera aberta, qualidade padrão, clientes por sala, etc. Hoje a instalação padrão do Debian consome até 6GB de RAM, portanto uma máquina com 4vCPUs e 8GB pode atender muito bem nos padrões da instalação, isso é, *setup & forget*. Se você deseja subir numa máquina com menos memória é interessante editar os serviços para configurar a JVM dos mesmos para evitar crashs por falta de recursos, é possível encontrar essas configurações nos scripts de post install do Debian, disponível em https://github.com/jitsi/jitsi-meet/tree/master/debian . O ideal é que seja feito um capacity planning básico antes da implementação do Jitsi, assim é possível validar quanto uma instalação consegue atender sua empresa com base nos requisitos de qualidade, por exemplo.

Existem numerosos relatos do Jitsi rodando em máquinas mais simples, com 1 vCPU e 1GB de RAM, por exemplo, é importante destacar que o Jitsi utiliza primariamente CPU e banda e novamente existem uma variação para determinadas qualidades de aúdio e vídeo, e são pontos que afetam diretamente a quantidade de salas e usuários que o Jitsi pode atender simultaneamente.

Pela natureza stateless do Jitsi, a infraestrutura imutável desempenha um papel importante, nesse caso é relativamente fácil ter uma configuração baseada em imagens e que provisiona os recursos em cloud necessários utilizando o Packer e o Terraform, por exemplo, aumentando a elasticidade em que o Jitsi pode atender seu negócio.

### Instalação rápida

#### Requisitos

Os requisitos são bem simples e voltados para comunicação do Jitsi com os clientes, basicamente você precisa de:

- Um registro do tipo A apontando para a máquina do Jitsi

- Um certificado HTTPS
  - Para testes pode se usar um certificado auto assinado, no entanto, dependendo da política de segurança do seu navegador não é possível transmitir áudio ou vídeo, além disso, os aplicativos mobile **NÃO** conectam em instâncias onde o certificado não foi assinado por uma CA confiável.

- As portas utilizadas pelo Jitsi abertas, no caso das portas pode existir pequenas variações com base no tipo de implementação e versão do Jitsi. Atualmente essas são as mais usadas: 
  - 80/tcp
  - 443/tcp
  - 10000/udp
  - 22/tcp
  - 3478/udp
  - 5349/tcp
  
- Acesso de administrador na máquina a ser configurada

#### Debian

A implantação para o Debian é voltada primariamente para o uso de meta pacotes que trazem como dependências as versões dos componentes que atendem pela versão X do Jitsi junto de scripts para instalação e manutenção do mesmo.

A primeira coisa a ser feita é garantir que o pacote `gnupg2` esteja instalado, feito isso podemos seguir.

1. Atualize a metadata sobre os pacotes e garanta o uso de HTTPS para o APT:

```shell
apt update;
apt install -y apt-transport-https
```

2. Adicione o repositório do Jitsi:

```shell
curl https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list > /dev/null
sudo apt update
```

3. Caso use um firewall na máquina, garanta que o mesmo permite as portas descritas anteriormente, exemplo com o UFW:

```shell
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 10000/udp
ufw allow 22/tcp
ufw allow 3478/udp
ufw allow 5349/tcp
ufw enable
```

Se estiver atrás de uma NAT não se esqueça de fazer o devido redirecionamento para as portas descritas anteriormente.

4. Instale o meta pacote do Jitsi:

```shell
sudo apt install jitsi-meet
```

> OBS.: O Jitsi é primariamente Java, por isso, pode ser necessário instalar o OpenJDK caso o meta pacote não o traga como dependência.

O instalador faz duas perguntas, sobre o certificado HTTPS e o domínio que irá responder, por padrão ele também checa se já existe um servidor web instalado e o configura, se não tiver ele instala e provisiona um NGINX.

Se você já tiver o domínio, pode optar pelo certificado da Let's Encrypt como método rápido e fácil para obter um certificado 100% funcional.

A partir daqui, você já tem uma instalação do Jitsi pronta, se desejar seguir pelo caminho do Let's Encrypt basta chamar o script de configuração:

```shell
/usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

> OBS.: Esse script efetua uma challenge HTTP-01 então certifique-se que seu servidor está acessível na internet a partir das portas 80 e 443.

Se você possui muitas customizações para o Jitsi pode ser interessante efetuar o `hold` (bloquear atualizações automáticas) de alguns dos pacotes do Jitsi, em especial o Prosody, isso evita que uma atualização do mesmo possa quebrar algum plugin customizado, por exemplo, efetuando as atualizações somente com versões pré validadas para seu ambiente.

5. (Opcional) Solucionar possíveis problemas de NAT:

Caso tenha problemas com uma implantação atrás de NAT, certifique-se que as portas necessárias estão abertas, você também pode adicionar um harvester ao Jitsi com os IPs da máquina em que ele está instalado:

```shell
vim /etc/jitsi/videobridge/sip-communicator.properties
# Adicione as linhas abaixo
org.ice4j.ice.harvest.NAT_HARVESTER_LOCAL_ADDRESS=<Local.IP.Address>
org.ice4j.ice.harvest.NAT_HARVESTER_PUBLIC_ADDRESS=<Public.IP.Address>
# Comente a linha abaixo
org.ice4j.ice.harvest.STUN_MAPPING_HARVESTER_ADDRESSES
```

6. (Opcional) Aumentando os limites do Systemd para servir mais usuários:

Se tem a intenção de servir mais de 100 usuários é interessante aumentar o limite de processos e arquivos abertos do Systemd:

```shell
vim /etc/systemd/system.conf
# Adicione as linhas abaixo
DefaultLimitNOFILE=65000
DefaultLimitNPROC=65000
DefaultTasksMax=65000
```

#### NixOS

Por ter um módulo suportado pela comunidade do NixOS, a instalação é extremamente simples e consiste em adicionar algumas entradas a sua configuração:

```nix
...
   services.jitsi-meet = {
    enable = true;
    hostName = "jitsi.example.com";
  };
  services.jitsi-videobridge.openFirewall = true;
  networking.firewall.allowedTCPPorts = [ 80 443 ];
  security.acme.email = "me@example.com";
  security.acme.acceptTerms = true;
...
```

As configurações acima declaram, o certificado HTTPS utilizando o Let's Encrypt, abrem as portas necessárias para o Jitsi junto as de HTTP e HTTPS e configuram o domínio em que o Jitsi operará, feito essa adição basta efetuar o build da sua configuração:

```shell
nixos-rebuild switch
```

Com isso o Jitsi já estará disponível no seu servidor NixOS, vale lembrar que o módulo [possui muitas outras configurações](https://search.nixos.org/options?channel=20.09&from=0&size=50&sort=relevance&query=jitsi) que permitem de maneira fácil e declarativa habilitar ou desabilitar funções do Jitsi, inclusive da interface.

