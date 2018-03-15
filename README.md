# Laboratório - 23ATI

## Material de apoio

* [Conexão da sua instância do Linux no Windows usando PuTTY](https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/putty.html)


## 1. Reconfigurar o Chef-Server

O primeiro passo é reconfigurar o Chef Server. Precisaremos fazer isto pois o endereço IP público do servidor é alterado quando reiniciamos o mesmo. É importante notar que isto ocorre pois quando realizamos a criação do servidor não especificamos o uso de um endereço IP público fixo.

Após iniciar o servidor através da console da AWS, vamos executar os seguintes comandos:

    $ sudo bash
    # echo api_fqdn "\""$(curl --silent --show-error --url http://169.254.169.254/latest/meta-data/public-ipv4)"\"" > /etc/opscode/chef-server.rb
    # chef-server-ctl reconfigure
    # chef-manage-ctl reconfigure


## 2. Acessar o Chef Server e excluir o node existente

No último lab, utilizamos um container para simular o registro de um `Node` no Chef Server. Como o container em questão não existe mais (pois desligamos o servidor onde o mesmo está em execução), devemos excluir o Node através da interface web do Chef Server.

Para isto, em `Nodes`, vamos selecionar o node `first_container` e no campo actions, no canto direito da página, clicar em `Delete`, conforme a imagem a seguir:

![delete_node](https://github.com/bemer/23ati/blob/master/images/delete_node.png)

## 3. Download do Starter Kit

Agora que já temos nosso node deletado, precisamos baixar o Starter Kit novamente para reconfigurar nosso workspace. Neste caso, em nosso Chef Server, vamos clicar em `Administration` na parte superior da tela, em seguida devemos clicar em `Starter Kit` e então em `Download Starter Kit`:

![download_starter_kit](https://github.com/bemer/23ati/blob/master/images/download_starter_kit.png)

Feito isto, devemos enviar nosso novo Starter Kit para o Chef Server.

Podemos fazer isto utilizando o `Amazon S3` ou algum client SCP, como o Filezilla, por exemplo.

## 4. Reconfiguração do Chef Workspace

Vamos agora excluir o diretório `chef-repo` que existia em nosso servidor. Este diretório será substituído pelo novo arquivo gerado.

Para isto, vamos utilizar o comando:

    # rm -rf chef-repo/

Em seguida, devemos extrair o conteúdo do novo Starter Kit. Vamos utilizar o comando:

    # unzip unzip chef-starter.zip

Agora, devemos acessar o diretório `chef-repo` e baixar os certificados ssl. Vamos utilizar os comandos:

    # cd chef-repo
    # knife ssl fetch

Para validar o funcionamento de nosso workspace, vamos utilizar o comando:

    # knife client list

## 5. Iniciar um novo Node

Agora vamos iniciar novamente nossa instância chef-server.
Após iniciar nossa instância e acessar a mesma via SSH vamos executar um novo container, afim de simular um servidor.

Antes de termos um novo container em execução, no entanto, vamos remover o container anterior, que está com o status `Exited`, utilizando os seguintes comandos:

    $ sudo bash
    # docker rm -f $(docker ps -a -q)

Vamos validar se o nosso ambiente está sem nenhum container em execução através do seguinte comando:

    # docker ps

Note que não existe nenhum container em execução.

Vamos agora, executar um novo container, agora chamado `motd_server` executando o comando:

    # docker run -d -P --name motd_server ssh_container

Verifique o status de seu container através do comando:

    # docker ps

A saída deverá ser semelhante a:

    CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                   NAMES
    580fe1c98903        ssh_container       "/usr/sbin/sshd -D"   2 seconds ago       Up 1 second         0.0.0.0:32768->22/tcp   motd_server

## 6. Obtendo os dados para Bootstrap do novo Node

Ainda no `Chef Client`, obtenha os dados para bootstrap de seu novo node. O primeiro passo é obter o endereço IP de sua instância. Faça isso através do comando:

    # curl --silent --show-error --url http://169.254.169.254/latest/meta-data/local-ipv4; echo

Tome nota deste endereço IP, pois o mesmo será utilizado para o bootstrap de nosso novo Node.
Agora, devemos obter a porta sendo mapeada em nossa instância para o acesso via ssh ao container em execução. Para isto, execute o comando:

    # docker port motd_server 22 | awk '{split($0,a,":"); print a[2]}'

Anote também a porta utilizada por este container.


## 7. Bootstrap de nosso novo Node

Vamos agora acessar o `Chef Server` e utilizando o knife realizar o bootstrap de nosso novo node.
Ao acessar o Chef Server, acesse o diretório `chef-repo`. Você pode verificar se já se encontra no diretório correto através do comando:

    # pwd

Caso não esteja no diretório, acesse o mesmo utilizando o comando:

    # cd chef-repo

Vamos agora utilizar o knife para realizar o bootstrap de nosso novo container. Para isto, devemos utilizar o comando:

    # knife bootstrap `<endereço_IP_do_chef_client>`:`<porta_do_container>` -x root -P 123456 -N motd_server

Feito isto, vamos validar o bootstrap de nosso novo container primeiro em nossa interface web. Acesse novamente o Chef Server através da Web, e note que deverá existir um novo Node com o nome `motd_server`:

![motd_server](https://github.com/bemer/23ati/blob/master/images/motd_server.png)

Podemos também listar o nosso node através do comando:

    # knife client list

O node `motd_server` deverá ser listado em sua linha de comando.
