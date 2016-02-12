## Criar novo usuário do sistema

Somos tentados a usar sempre o usuário root porque não pede senha para rodar comandos. Apesar disso ser uma facilidade, esconde uma vulnerabilidade que pode ser encontrada quando for tarde demais. Usar o sistema como usuário root o tempo todo abre margens para ataques contra o seu servidor. Segurança nunca é demais e para se previnir disso, uma das coisas a se fazer é criar um novo usuário com ou sem permissões de root.

Para criar um novo usuário, digite no terminal:

```
adduser nomedousuario
```

Serão feitas algumas perguntas começando pela senha. O ideal é que se utilize uma senha forte. De nada adianta fazer este passo e colocar uma senha que qualquer pessoa tem acesso. Fora a senha, as outras informações são opcionais. Este comando cria o novo usuário com uma pasta própria dentro de /home.

Então, digite:

```
gpasswd -a nomedousuario sudo
```

Este comando faz com que o usuário que acabou de criar entre no grupo `sudo`. Todos os usuários que estiverem no grupo `sudo` terão permissões de usuário administrador quando executarem um comando precedido de 'sudo'.

Faça uma nova conexão SSH usando o novo usuário e senha para testar se tudo correu bem.

### Dicas para uso de novos usuários do sistema

Crie um novo usuário para uso geral do sistema.

Pode-se adotar qualquer política para adição de novos usuários no sistema. Uma das mais eficazes é adicionar um usuário relacionado ao projeto. Por exemplo:

Se você tem um projeto chamado **site1**, logo, o nome do novo usuário deve se chamar **site1**. Os arquivos deste projeto devem ficar dentro pasta `/home/site1`. Assim o projeto só é visto e executado pelos usuários site1 e outros com privilégios de administrador.

**Evite ao máximo o uso do usuário root!**

## Criar chave SSH

Precisamos de chave SSH para ter acesso a alguns serviços como Github e Bitbucket, acessar outros servidores, etc. Para criar esta chave, digite no terminal:

```
ssh-keygen
```

Serão feitas algumas perguntas onde armazenar a chave SSH e uma frase. Para os dois campos, pode deixar em branco para ser feito o modo padrão, mas é recomendado na frase você escrever alguma coisa. Não precisa ser nada elaborado. Isso servirá apenas para criar a chave de segurança.

Feito isso, a chave pode ser encontrada na pasta /home/nomedeusuario/.ssh. Dentro dessa pasta, você terá a chave pública `id_rsa.pub` e chave privada `id_rsa`.

## Instalação de pacotes necessários para desenvolvimento

Primeiramente, precisamos instalar pacotes que são necessários para desenvolvimento web. Para isso, digite o seguinte comando:

```
sudo apt-get install curl build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion git git-svn gitk ssh libssh-dev
```

Não se preocupe muito com o que cada um desses pacotes faz. São pacotes básicos que estão presentes em todo servidor web.

Se for trabalhar com banco de dados Redis (ex.: Laravel, Ruby on Rails), instale os pacotes:

```
sudo apt-get install redis-server libhiredis-dev
```

Para trabalhar com cache de aplicações, use o Memcache:

```
sudo apt-get install memcached libmemcached-dev
```

Muitas aplicações precisam mainipular imagens. Para isso, instalamos o Imagemagick:

```
sudo apt-get install imagemagick libmagickwand-dev
```

## Instalar Postgres

É preciso incluir o repositório oficial do Postgres se quiser usar a versão mais recente do banco de dados:

```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main" >> /etc/apt/sources.list.d/postgresql.list'
```

Feito isso, rode os comandos:

```
sudo apt-get update
sudo apt-get install postgresql-9.4 postgresql-server-dev-9.4 postgresql-contrib-9.4
```

### Criação de usuário no Postgres

O Postgres vem com o usuário postgres por padrão. Para adicionar um novo:

```
sudo su postgres
createuser -P -s -e nomedousuario
```

Feito isso, edite o arquivo /etc/postgresql/9.4/main/pg_hba.conf e adicione ao final:

```
host all all all md5
```

Isso indica que **todos os bancos de dados estão disponíveis para acesso a todos os usuários de qualquer IP através de autenticação por senha**. Verifique a documentação do Postgres para saber mais informações de como personalizar este arquivo.

### Habilitando conexões remotas no Postgres

Por padrão, o Postgres aceita apenas conexões locais. Para mudar isso, abra o arquivo /etc/postgresql/9.4/main/postgres.conf. Procure por:

```
#listen_addresses = 'localhost'
```

Altere para:

```
listen_addresses = '*'
```

Isso habilita conexões de qualquer IP. Caso queira restringir a um ou alguns IPs, basta substituir o * pelo número desejado.

Neste arquivo também é possível alterar a porta padrão do Postgres que é 5432. Procure por:

```
port = 5432
```

E altere para o número desejado.

## Instalar MySQL

```
sudo apt-get install mysql-server
```

### Gerar a estrutura de diretórios do MySQL

```
sudo mysql_install_db
```

Para finalizar, rode este comando configuração ativação/desativação de acesso remoto entre outros:

```
sudo mysql_secure_installation
```

Você vai precisar da senha de root do MySQL.

### Alterar a porta de acesso ao MySQL

O MySQL vem configurado para ser acessado através da porta 3306. Para alterar essa porta, digite o comando:

```
sudo vim /etc/mysql/my.cnf
```

Há dois lugares onde se deve alterar a porta. São eles:

```
[client] port = numero_da_porta
```

e

```
[mysqld] port = numero_da_porta
```

### Autorizar conexões remotas ao MySQL

É preciso informar ao MySQL qual usuário e ip que estarão autorizados a acessar o banco de dados remotamente. Para isso precisamos comentar uma linha no mesmo arquivo /etc/mysql/my.cnf.

Procure por:

```
bind-address 127.0.0.1
```

Deixe assim:

```
#bind-address 127.0.0.1
```

Após isso, dentro do servidor, entre no MySQL com o comando:

```
mysql -u NOMEDOUSUARIO -P PORTA -p
```

Digite a senha.

Dentro do MySQL, digite os comandos:

```
GRANT ALL PRIVILEGES ON *.* TO 'NOMEDOUSUARIO'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

Este comando dá todos os privilégios ao usuário vindas de qualquer IP a todos os banco de dados e tabelas.

O ideal é criar um usuário especifíco para acesso remoto ao MySQL. Para criar um novo usuário já com essas permissões, digite:

```
GRANT ALL PRIVILEGES ON *.* TO 'NOMEDEUSUARIO'@'%' IDENTIFIED BY 'SENHA' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

Se preferir, você pode especificar um único IP com acesso remoto.

```
GRANT ALL PRIVILEGES ON *.* TO 'NOMEDOUSUARIO'@'NUMERODOIP' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

Feito isso, reinicie o MySQL.

```
sudo service mysql restart
```

## Instalar PHP

```
sudo apt-get install php5-fpm php5-mysql
sudo apt-get install php5 php5-mcrypt
```

### Configuração do PHP processor

```
sudo vim /etc/php5/fpm/php.ini
```

Procure por cgi.fix_pathinfo. Sua linha deve estar comentada. Descomente-a e deixe-a da seguinte forma:

```
cgi.fix_pathinfo=0
```

Isso é feito, por questões de segurança, porque o PHP, por padrão, executa o arquivo mais próximo se um arquivo PHP não for encontrado. Essa alteração previne a execução de scripts maliciosos por um invasor.

Salve o arquivo e feche.

Rode o seguinte comando para efetivar as alterações:

```
sudo service php5-fpm restart
```

### Atualizar a versão do PHP

Os passos anteriores instalam o PHP 5.5. Para projetos atuais que utilizam o framework Laravel 5.1, precisamos da versão 5.6 do PHP. Para termos a versão atual do PHP rode o comando:

```
sudo add-apt-repository ppa:ondrej/php5-5.6
```

Este comando adiciona o repositório com a versão do 5.6 do PHP. Aperte Enter para confirmar e espere a finalização do processo. Para atualizar:

```
sudo apt-get update
sudo apt-get install php5
```

Terminado o processo de instalação, verifique a versão do PHP com o comando.

```
php -v
```

## Instalar Ruby

```
curl -L https://get.rvm.io | bash -s stable
```

Fique atento na saída deste comando. Ele mesmo fica responsável por instalar a chave GPG e o RVM que gerencia as versões de Ruby, mas isso pode falhar. Siga as instruções que aparecerem no terminal caso isso ocorra e rode novamente o comando.

Depois de baixado, rode o seguinte comando para adicionar o path:

```
source ~/.rvm/scripts/rvm
```

O próximo passo é instalar os pacotes necessários para o funcionamento do Ruby. O próprio RVM faz isso. Basta rodar o comando:

```
rvm requirements
```

Para listar as versões de Ruby disponíveis:

```
rvm list known
```

Para instalar uma versão:

```
rvm install 2.2.3
```

Com isso, já podemos instalar o Ruby on Rails:

```
gem install rails
```

## Instalar Nginx através do Phusion Passenger

Primeiramente, instale os seguintes pacotes:

```
sudo apt-get install libcurl4-openssl-dev
```

Feito isso, é preciso instalar o Passenger:

```
gem install passenger
```

Depois rode os comandos:

```
sudo chown -R `whoami` /opt
passenger-install-nginx-module --auto-download --auto
```

O primeiro comando coloca a permissão da pasta /opt para o seu usuário. Isso é apenas um truque para não ter que ficar usando sudo a todo momento.

O segundo é o comando que instala o Nginx com Passenger de forma automática. Após finalizado, os arquivos de configuração do Nginx estarão na pasta opt/nginx.

Podemos então executar:

```
sudo chown -R root /opt
```

para a pasta `/opt` voltar a pertencer ao usuário root.

***Obs: Caso este comando apresente algum erro, muito provável que seja pela quantidade de memória. A instalação deste módulo requer 2GB de memória ram no mínimo para ser executada.***

Após isso, precisamos de um script que fará o serviço nginx rodar no seu sistema. Excute os comandos abaixo:

```
wget -O init-deb.sh https://gist.github.com/mdamaceno/de2eed3e23f76c853c0b/raw/430f60a6223068c2fcbcaf74f097da461a45c3f7/660-init-deb.sh
sudo mv init-deb.sh /etc/init.d/nginx
sudo chmod +x /etc/init.d/nginx
sudo /usr/sbin/update-rc.d -f nginx defaults
```

Agora, você pode usar um dos seguintes comandos:

```
sudo service nginx start
sudo service nginx stop
sudo service nginx restart
```

Acesse http://ENDERECO_IP_DO_SEU_SERVIDOR

## Configurar o Nginx para usar o PHP Processor

Entre no arquivo:

```
/opt/nginx/conf/nginx.conf
```

Para uma configuração básica, seu nginx.conf deve se parecer com isso (os comentários foram ignorados):

```
user  www-data;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    passenger_root /home/vagrant/.rvm/gems/ruby-2.2.3/gems/passenger-5.0.23;
    passenger_ruby /home/vagrant/.rvm/gems/ruby-2.2.3/wrappers/ruby;

    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    gzip  on;

    server {
        listen       80;
        server_name  localhost;
        root   /home;

        location / {
            index  index.php index.html index.htm;
            try_files $uri $uri/ =404;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php5-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }
}
```

Lembrando que os caminhos em `passenger_root` e `passenger_ruby` apontam para o caminho onde estão instalados o **Passenger** e **Ruby** respectivamente.

Feito isso, reinicie o Nginx.

```
sudo service nginx restart
```

Agora o seu Nginx está funcionando!

## Instalar NPM e Node.js

Frameworks como Laravel e Ruby on Rails utilizam NPM / Node.js. Para instala-los, vamos usar o gerenciador NVM. Digite o comando:

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.29.0/install.sh | bash
```

Terminado o processo, digite:

```
source ~/.bashrc
nvm --version
```

Para instalar o Node.js:

```
nvm install 5.5.0
node -v
npm -v
```

Verifique a versão atual no site do [Node.js](https://nodejs.org/en/).

## Instalar Composer

O framework Laravel usa o Composer para gerenciamento de dependências. Para instalar, digite o comando:

```
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

Aguarde todo o processo de instalação e digite `composer`.

Tudo pronto para iniciar os trabalhos no servidor!

Referências:

[DigitalOcean](https://www.digitalocean.com/community/tutorials/como-instalar-a-pilha-linux-apache-mysql-php-lamp-no-ubuntu-14-04-pt)

[AkitaOnRails](http://www.akitaonrails.com/2015/01/28/ruby-e-rails-no-ubuntu-14-04-lts-trusty-tahr)
