# Configurando um certificado SSL em um servidor apache2 - Ubuntu 18.04

Este tutorial foi feito para pessoas que já possuem o certificado e precisam inserir no servidor e configurar para funcionar em um server Apache2. 

Caso você não tenha o certificado e queria gerar um certificado autoassinado gratuito, aconselho a seguir o tutorial da Digital Ocean([https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04-pt](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04-pt)).

Esse tutorial também foi escrito através da minha experiência configurando o servidor e também com a ajuda do material da Digital Ocean([https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04-pt](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04-pt))

### Passo 1:

Alterando as permissões das pastar **certs** e **private** que localizado em **`cd /etc/ssl`**. Ao acessar esse diretório, executar o seguinte comando abaixo:

```bash
sudo chmod 777 certs
```

```bash
sudo chmod 777 private
```

### Passo2:

Com o certificado que você gerou, você deve transferi-los para as as pastas **certs** e **private** através de algum programa que acesse o seu servidor via FTP. Eu utilizei o **WinSCP** para fazer isso, mas fique a vontade para usar o programa que quiser.

Você deve transferir o arquivo ***.pem** para a pasta **certs** e o arquivo ***.key** para a pasta **private**.

### Passo 3:

Iremos configurar o Apache para usar o SSL.

Colocamos nossos arquivos do certificado no diretório **`cd /etc/ssl`**. Agora, precisamos modificar nossa configuração do Apache para aproveitarmos melhor o mesmo.

Vamos fazer alguns ajustes na nossa configuração:

1. Criaremos um snippet de configuração para especificar configurações padrão robustas do SSL.
2. Vamos modificar o arquivo SSL Apache Host Virtual incluso para apontar para nossos certificados SSL gerados.
3. (Recomendado) Modificaremos o arquivo de Host Virtual não criptografado para redirecionar automaticamente as solicitações para o Host Virtual criptografado.

Quando terminarmos, devemos ter uma configuração SSL segura.

### Passo 4:

### Criando um snippet de configuração do Apache com configurações de criptografia robustas

Primeiro, vamos criar um snippet de configuração do Apache que defina algumas configurações do SSL. Isso irá configurar o Apache com uma série de criptografia SSL forte e habilitar algumas funcionalidades avançadas que irão ajudar a manter nosso servidor seguro. Os parâmetros que vamos definir podem ser usados por qualquer Host Virtual habilitando o SSL.

Crie um novo snippet no diretório `/etc/apache2/conf-available`. Vamos nomear o arquivo `ssl-params.conf` para deixar seu objetivo claro:

```bash
sudo nano /etc/apache2/conf-available/ssl-params.conf
```

Cole a configuração no arquivo `ssl-params.conf` que abrimos:

                                    

                                   `/etc/apache2/conf-available/ssl-params.conf`

```bash
SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLHonorCipherOrder On
# Disable preloading HSTS for now.  You can use the commented out header line that includes# the "preload" directive if you understand the implications.# Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff
# Requires Apache >= 2.4
SSLCompression off
SSLUseStapling on
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
# Requires Apache >= 2.4.11
SSLSessionTickets Off
```

Salve e feche o arquivo quando você terminar.

### Passo 5:

### Modificando o arquivo padrão de Host Virtual SSL do Apache

### OBS: Estou usando o `default-ssl.conf` como exemplo, mas você deve aplicar isso para o arquivo de configuração do seu site.

Em seguida, vamos modificar o `/etc/apache2/sites-available/default-ssl.conf`, o arquivo padrão de Host Virtual SSL do Apache Se estiver usando um arquivo de bloco de servidor diferente, substitua seu nome nos comandos abaixo.

Antes de continuar, vamos voltar para o arquivo de Host Virtual SSL original:

```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Dentro, com a maioria dos comentários removidos, o arquivo de Host Virtual deve se parecer com isso por padrão:

```
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin webmaster@localhost

                DocumentRoot /var/www/html

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                SSLEngine on

                SSLCertificateFile      /etc/ssl/certs/ssl-cert-snakeoil.pem
                SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key

                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>

        </VirtualHost>
</IfModule>
```

Vamos fazer alguns pequenos ajustes no arquivo. Vamos definir as coisas normais que gostaríamos de ajustar em um arquivo de Host Virtual (endereço de e-mail do ServerAdmin, ServerName, etc., e ajustar as diretivas do SSL para que apontem para nossos arquivos de certificado e chave.

Após fazer essas alterações, seu bloco de servidor deve se parecer com este:

OBS: O que está sublinhado em Vermelho é o que você deve alterar. 

```
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin your_email@example.com
                ServerName server_domain_or_IP
								ServerAlias server_domain_or_IP						

                DocumentRoot /var/www/pasta_do_site

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                SSLEngine on

                SSLCertificateFile      /etc/ssl/certs/seu_certificado.pem
                SSLCertificateKeyFile /etc/ssl/private/seu_certificado.key

                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>

        </VirtualHost>
</IfModule>
```

Salve e feche o arquivo quando você terminar.

### (Recomendado) Modificando o arquivo de host HTTP para redirecionar para HTTPS

Da forma em que se encontra, o servidor fornecerá tanto tráfego HTTP não criptografado quanto HTTPS criptografado. Para maior segurança, é recomendável redirecionar na maioria dos casos o HTTP para HTTPS automaticamente. Se não quiser ou precisar dessa funcionalidade, você pode pular essa seção com segurança.

Para ajustar o arquivo de Host Virtual não criptografado para redirecionar todo o tráfego a ser criptografado pelo SSL, podemos abrir o arquivo `/etc/apache2/sites-available/redirect_http_https.conf`:

```bash
sudo nano /etc/apache2/sites-available/redirect_http_https.conf
```

Lá, dentro dos blocos de configuração VirtualHost, precisamos adicionar uma diretiva Redirect, que direciona todo o tráfego para a versão SSL do site:

```
<VirtualHost *:80>
        . . .

        Redirect "/" "https://your_domain_or_IP/"

        . . .
</VirtualHost>
```

Salve e feche o arquivo quando você terminar.

### Passo 6:

### Como ajustar o Firewall

Se você tem o firewall `ufw`ativado, conforme recomendado pelos guias de pré-requisitos, pode ser necessário ajustar as configurações para permitir o tráfego SSL. Felizmente, o Apache registra alguns perfis com o `ufw` na instalação.

Podemos ver os perfis disponíveis digitando:

```bash
sudo ufw app list
```

Você deve ver uma lista como essa:

```
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```

Em seguida execute o comando:

```bash
sudo ufw enable
```

Você pode verificar a configuração atual digitando:

```bash
sudo ufw status
```

Se você permitiu apenas o tráfego HTTP regular mais cedo, seu resultado pode se parecer com este:

```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Apache                     ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Apache (v6)                ALLOW       Anywhere (v6)
```

Para também admitir o tráfego HTTPS, podemos permitir o perfil “Apache Full” e então excluir a permissão de perfil redundante “Apache”:

```bash
sudo ufw allow 'Apache Full'
sudo ufw delete allow 'Apache'
```

Agora, seu status deve se parecer com este:

```bash
sudo ufw status
```

```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Apache Full                ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Apache Full (v6)           ALLOW       Anywhere (v6)
```

### Passo 7:

### Habilitando as alterações no Apache

Agora que fizemos nossas alterações e ajustamos nosso firewall, é possível habilitar os módulos SSL e de cabeçalhos no Apache, habilitar nosso Host Virtual pronto para o SSL e reiniciar o Apache.

Podemos habilitar o `mod_ssl`, o módulo SSL do Apache e o `mod_headers`, necessário para algumas das configurações no nosso snippet SSL, com o comando `a2enmod`:

```bash
sudo a2enmod ssl
sudo a2enmod headers
```

Em seguida, podemos habilitar nosso Host Virtual SSL com o comando `a2ensite`:

### OBS: Lembrando que o default-ssl.conf está sendo como exemplo, você deve aplicar o arquivo de configuração do seu site que se encontra nesse diretório.

```bash
sudo a2ensite default-ssl.conf
sudo a2ensite redirect_htpp_https.conf
```

Também precisaremos habilitar nosso arquivo `ssl-params.conf`, para que leia nos valores que definirmos:

```bash
sudo a2enconf ssl-params
```

Neste ponto, nosso site e os módulos necessários estão habilitados. Devemos verificar e garantir que não haja erros de sintaxe em nossos arquivos. Podemos fazer isso digitando:

```bash
sudo apache2ctl configtest
```

Se tudo for bem-sucedido, você receberá um resultado que se parecerá com este:

```
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
Syntax OK
```

A primeira linha é apenas uma mensagem informando que a diretiva `ServerName` não está definida globalmente. Se você quiser se livrar dessa mensagem, é possível definir o `ServerName` como o nome do domínio do seu servidor ou o endereço IP em `/etc/apache2/apache2.conf`. Isso é opcional, uma vez que a mensagem não causará problemas.

Se seu resultado tiver `Syntax OK`, seu arquivo de configuração não possui erros de sintaxe. Podemos reiniciar com segurança o Apache para implementar nossas alterações:

```bash
sudo systemctl reload apache2.service
sudo systemctl restart apache2.service
```

### Finalizando...

Agora que você fez todas as configurações, você deve ir até o diretório `cd /var/www/seu_site` e abrir o arquivo config.php. Nesse arquivo você vai mudar apenas uma coisa:

### De: `http://`

```bash
$CFG->wwwroot = 'http://seu_site';
```

### Para: `https://`

```bash
$CFG->wwwroot = 'https://seu_site';
```

Pronto! Agora é só acessar o seu site utilizando a requisição `https://seu_site`.
