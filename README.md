# install-redmine-postgres-docker-ubuntu-18.04
 installation script for redmine and postgres with docker on ubuntu 18.04


<h1>create host Ubuntu 18.04</h1>

<h3>Execute commands in the following order:</h3>


sudo apt update

sudo apt upgrade

sudo apt-get install  curl apt-transport-https ca-certificates software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update

sudo apt install docker-ce

sudo systemctl status docker

docker pull postgres

docker pull redmine

docker network create some-network

  
<h3>Creating postgres docker container</h3>
<h3>(the database is stored on the server and not on the docker with  **/var/data/postgres/datadir **)</h3>

docker run -d -v /var/data/postgres/datadir:/var/lib/postgresql/data --name some-postgres --network some-network -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=redmine postgres


<h3>Configuração para envio de e-mail com gmail</h3>

<h4>create  <i>configuration.yml</i> file on the server in the  <i>/usr/src/redmine/config/</i> folder with the following content:<h4>

<code>
<p>default:</p>
<p>	email_delivery:</p>
<p>		delivery_method: :smtp</p>
<p>		smtp_settings:</p>
<p>      	enable_starttls_auto: true</p>
<p>      	address: "smtp.gmail.com"</p> 
<p>     	port: 587</p>
<p>     	domain: "smtp.gmail.com"</p> 
<p>      	authentication: :login</p>
<p>      	user_name: "email@gmail.com"</p> 
<p>      	password: "senha"</p>  
<code>


<h3>Creating redmine container docker expands it to by 80</h3>   

docker run -d -p 80:3000 -v /usr/src/redmine/config/configuration.yml:/usr/src/redmine/config/configuration.yml --name some-redmine --link some-redmine --network some-network -e REDMINE_DB_POSTGRES=some-postgres -e REDMINE_DB_USERNAME=redmine -e REDMINE_DB_PASSWORD=secret redmine

sudo systemctl enable docker

sudo systemctl start docker

<h3>Command to check ip from docker redmine<h3>

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' some-redmine

<h3>Hosts configuration</h3>
include in the file  <i>hosts</i> in the <i>/etc</i> the ip of the docker container with your domain:

172.18.0.3	www.meusite.com.br

<h3>apache2 configuration</h3>

<h3>Replace localhost with the application url in the 000-default.conf file
on the server in the <i>/etc/apache2/sites-avaliable<i>.</h3>

<i>redirect permanent:</i>

<VirtualHost *:80>
      Redirect permanent / http://localhost:3000/
</VirtualHost>

<i>docker container ip configuration:</i>

<VirtualHost *:80>
	
  ServerName 34.66.109.176
  <Location "/manager">
      ProxyPass "http://172.18.0.3:3000/"
      ProxyPassReverse "http://172.18.0.3:3000/"
      Order allow,deny
      Allow from all
  </Location>
</VirtualHost>


<h3>restart on your apache server after the configuration changes</h3>

sudo systemctl restart apache2.service

or

service apache2 reload

or

restart on the ubunto machine, but if it does it will be necessary to start in the containers manually as below.

<h3>If the server falls and does not raise the containers run following script for start and stop on putty </h3>

docker container start some-postgres
docker container start some-redmine

docker container stop some-postgres
docker container stop some-redmine


<h1>References</h1>

https://www.hostinger.com.br/tutoriais/install-docker-ubuntu
https://hub.docker.com/_/redmine
https://hub.docker.com/_/postgres
https://www.redmine.org/projects/redmine/wiki/EmailConfiguration
https://stackoverflow.com/questions/17157721/how-to-get-a-docker-containers-ip-address-from-the-host
https://stackoverflow.com/questions/46099348/how-to-use-apache-to-redirect-to-docker-container
