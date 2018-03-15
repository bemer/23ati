# Laboratório - 23ATI

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

## 3. 
