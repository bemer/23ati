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

## 8. Criando um cookbook

Como já temos nosso ambiente funcional e com um node já registrado, vamos agora criar um novo cookbook para execução através do Chef Server.

Para isto, vamos utilizar o seguinte comando:

    # chef generate cookbook cookbooks/motd

Gaste algum tempo para analizar a saída deste comando.

Após entender o que foi feito, vamos editar a nossa receita. Para isto, o primeiro passo é acessar o diretório `cookbooks` em nosso chef-repo. Utilize o comando:

    # cd cookbooks/motd

Explore o conteúdo deste diretório, afim de entender como funciona a estrutura padrão dos cookbooks do chef. Você pode inclusive explorar o conteúdo dos arquivos utilizando o comando `cat`.

Mais informações sobre a estrutura dos cookbooks podem ser encontrados [neste link](https://docs.chef.io/cookbooks.html).

## 9. Editando a receita default

Vamos editar nosso cookbook para que possamos executar algo em nosso novo node. Neste caso, vamos criar uma receita simples, que cria um arquivo chamado `motd.txt` no diretório `/tmp` de nosso container.

Para isto, edite o conteúdo do arquivo `recipes/default.rb` e insira o seguinte:

    file '/tmp/motd.txt' do
      content 'Arquivo criado utilizando o Chef Server!'
    end

Após realizar a edição, salve o arquivo.

## 10. Enviando o cookbook para o Chef Server

Como vimos anteriormente, todas as receitas são disponibilizadas em nossos nodes através do Chef Server. Sendo assim, devemos enviar a receita recém criada para o servidor. Para isto, vamos executar os seguintes comandos:

    # knife upload cookbooks/motd

> NOTE: Este arquivo deverá ser executado a partir do diretório `chef-repo`.

Após realizar o upload de seu cookbook, você poderá verificar o conteúdo do mesmo através da interface web, clicando em `Policy` na parte superior da tela, clidando em seu cookbook, chamado `motd` e em seguida, na parte inferior da tela, selecionando `Recipes` e em seguida `default.rb`, conforme a imagem abaixo:

![listando_cookbook](https://github.com/bemer/23ati/blob/master/images/listando_cookbook.png)

Você também conseguirá listar os cookbooks presentes no servidor através do seguinte comando:

    # knife cookbook list

## 11. Alterando o Run List do Node

Como nosso cookbook já está presente em nosso servidor, o próximo passo é editar o `Run list` para que o Chef Client instalado em nosso container possa realizar o download e a execução de nossa receita.

Para isto, na interface web do servidor, vamos clicar em `Nodes` na parte superior da tela, selecionar o nosso node chamado `motd_server` e clicar no campo `Edit Run List` em actions:

![node_run_list_1](https://github.com/bemer/23ati/blob/master/images/node_run_list_1.png)

Em seguida, devemos arrastar a receita `motd` presente em `Available Recipes` para o campo `Current Run List`:

![node_run_list_2](https://github.com/bemer/23ati/blob/master/images/node_run_list_2.png)

Em seguida, vamos clicar em `Save Run List`.

Você pode validar a nova Run List de seu node através do comando:

    # knife node show motd_server

Verifique se a receita `motd` está sendo exibida.

## 12. Executando nossa receita

O próximo passo para isto, é executar a nossa Run List no node `motd_server` para que a receita `motd` possa criar o arquivo no diretório `/tmp`.

Vamos então acessar o nosso `Chef Client` e executar o seguinte comando:

    # docker exec -ti motd_server /bin/bash

> NOTE: Este comando irá nos mover para dentro do container `motd_server` que está simulando um servidor em nosso ambiente. A partir daí, todos os comandos são executados dentro do container, e não no Chef Client.

Vamos agora utilizar o chef-client que foi instalado em nosso container durante o processo de bootstrap. Para isto, utilizamos o comando:

    # chef-client

Note que o `chef-client` irá realizar o download de seu cookbook e executar os passos localmente.

## 13. Verificando os relatórios

Agora, na interface web do Chef Server, clique em `Reports` na parte superior da tela, e verifique o status de suas execuções:

![reporting](https://github.com/bemer/23ati/blob/master/images/reporting.png)

## 14. Criando um novo Node para instalação do Apache

Vamos criar um novo Node para que possamos simular a instalação do Apache. Para isto, vamos precisar gerar um novo container, expondo além da porta 22 para o ssh, também a porta 80 para o nosso web server.

Para isto, vamos voltar à nossa instância `Chef Client` e editar o nosso `Dockerfile`. Vamos editar a linha que contém a intrução `EXPOSE` adicionando também a porta `80`. Esta linha deverá ficar da seguinte forma:

    EXPOSE 22 80

Agora, vamos gerar uma nova imagem a partir de nosso Dockerfile. Esta imagem irá se chamar `ssh_apache_image`. Para isto, utilizaremos o seguinte comando:

    # docker build -t ssh_apache_image .

Após a geração desta imagem, vamos executar um novo container, expondo também a porta 80. Utilize o comando:

    # docker run -d -p 80:80 -P --name apache_server2 apache_image

Vamos agora listar as portas utilizadas em nosso novo container utilizando o comando:

    # docker port apache_server 22 | awk '{split($0,a,":"); print a[2]}'

Novamente, anote a porta apresentada para que possamos utilizar no processo de bootstrap.

Após a criação de uma nova imagem e a execução do container, realize o bootstrap deste novo node através do knife, em seu Chef Server. Execute os mesmos passos descritos no [passo 7](https://github.com/bemer/23ati#7-bootstrap-de-nosso-novo-node).

## 15. Criação de um novo Cookbook

Agora, ainda em seu Chef Server, realize a criação de um novo Cookbook, desta vez chamado `apache`. Utilize o comando:

    # chef generate cookbook cookbooks/apache

Edite o arquivo `cookbooks/apache/recipes/default.rb` e insira o seguinte conteúdo:

    package 'apache2' do
      action :install
    end

    service 'apache2' do
      action [:enable,:start]
      supports :reload => true
    end

Note que agora, além de instalar o apache, estamos utilizando a receita também para habilitar e iniciar o serviço `apache2`.

Envie seu novo cookbook para o servidor através do comando:

    # knife upload cookbooks/apache

## 16. Editando o Run List do Novo node

Siga o mesmo procedimento descrito no [passo 11](https://github.com/bemer/23ati#11-alterando-o-run-list-do-node), porém desta vez para o novo node utilizando a receita `apache`:

![apache_run_list](https://github.com/bemer/23ati/blob/master/images/apache_run_list.png)

## 17. Executando a o Run List no novo node

Em sua instância `Chef Client`, execute o seguinte comando para ter acesso ao container onde será instalado o Apache:

    # docker exec -ti apache_server /bin/bash

E em seguida, já dentro do container, execute o comando:

    # chef-client

Note que o Apache será instalado.

A primeira maneira para validar o funcionamento do Apache em nosso novo node, é verificando se o serviço foi devidamente instalado e está ativo. Faça isto através do comando:

    # service apache2 status

A saída deste comando deverá ser:

    \* apache2 is running

## 18. Acessando através de seu browser

Como estamos realizando a instalação de um servidor web, nada melhor do que utilizarmos o nosso browser para validação do funcionamento. Desta maneira, o primeiro passo é a liberação das regras do `Security Group` para que possamos ter acesso à porta utilizada pelo container.

Para isto, vamos abrir a console da AWS e lá, acessar o serviço `EC2`, clicar em `Security Groups` no menu lateral esquerdo e selecionar o Security Group `chef-client` criado anteriormente:

![security_group_1](https://github.com/bemer/23ati/blob/master/images/security_group_1.png)

Clique então na aba `Inbound` e então em `Edit`. Na próxima tela, clique em `Add Rule` e adicione uma regra com as seguintes informações:

* Type: Custom HTTP
* Source: Anywhere

E em seguida clique em `Save`:

![security_group_2](https://github.com/bemer/23ati/blob/master/images/security_group_2.png)

Agora, obtenha o hostname de sua instância chamada `Chef Client` e acesse o mesmo utilizando seu Browser. Você deverá se deparar com a interface do Apache Server.
