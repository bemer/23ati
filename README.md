# Laboratório - 23ATI

## 1. Reconfigurar o Chef-Server

O primeiro passo é reconfigurar o Chef Server. Precisaremos fazer isto pois o endereço IP público do servidor é alterado quando reiniciamos o mesmo. É importante notar que isto ocorre pois quando realizamos a criação do servidor não especificamos o uso de um endereço IP público fixo.

Após iniciar o servidor através da console da AWS, vamos executar os seguintes comandos:

    $ sudo bash
    # echo api_fqdn "\""$(curl --silent --show-error --url http://169.254.169.254/latest/meta-data/public-ipv4)"\"" > /etc/opscode/chef-server.rb
    # chef-server-ctl reconfigure
    # chef-manage-ctl reconfigure

Após isto, devemos acessar novamente o Chef-Server através da interface web, baixar o Starter Kit e enviar para o nosso Workspace.
