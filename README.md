# install-redmine-postgres-docker-ubuntu-18.04 (06/2019)
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

  
<h1>Creating postgres docker container</h1>
<h3>(the database is stored on the server and not on the docker with  /var/data/postgres/datadir)</h3>


docker run -d -v /var/data/postgres/datadir:/var/lib/postgresql/data --name some-postgres --network some-network -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=redmine postgres


<h1>Configuração para envio de e-mail com gmail</h1>
<h3>create  configuration.yml file on the server in the  /usr/src/redmine/config/ folder with the following content:<h3>


default:<br/>
	email_delivery:<br/>
	delivery_method: :smtp<br/>
	smtp_settings:<br/>
      	enable_starttls_auto: true<br/>
      	address: "smtp.gmail.com"<br/> 
     	port: 587<br/>
    	domain: "smtp.gmail.com"<br/>
     	authentication: :login<br/>
      	user_name: "email@gmail.com" <br/>
      	password: "senha"<br/>


<h1>Creating redmine container docker expands it to by 80</h1>   


docker run -d -p 80:3000 -v /usr/src/redmine/config/configuration.yml:/usr/src/redmine/config/configuration.yml --name some-redmine --link some-redmine --network some-network -e REDMINE_DB_POSTGRES=some-postgres -e REDMINE_DB_USERNAME=redmine -e REDMINE_DB_PASSWORD=secret redmine

sudo systemctl enable docker

sudo systemctl start docker


<h1>Command to check ip from docker redmine<h1>


docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' some-redmine

<h1>Hosts configuration</h1>
<h3>include in the file  hosts in the /etc the ip of the docker container with your domain:</h3>


172.18.0.3	www.meusite.com.br


<h1>apache2 configuration</h1>
<h3>Replace localhost with the application url in the 000-default.conf file on the server in the /etc/apache2/sites-avaliable.<h3>


redirect permanent:

<VirtualHost *:80>
      Redirect permanent / http://localhost:3000/
</VirtualHost>


docker container ip configuration:

<VirtualHost *:80>
	
  ServerName 34.66.109.176
  <Location "/manager">
      ProxyPass "http://172.18.0.3:3000/"
      ProxyPassReverse "http://172.18.0.3:3000/"
      Order allow,deny
      Allow from all
  </Location>
</VirtualHost>


<h1>restart on your apache server after the configuration changes</h1>


sudo systemctl restart apache2.service

or

service apache2 reload

or

restart on the ubunto machine, but if it does it will be necessary to start in the containers manually as below.


<h1>If the server falls and does not raise the containers run following script for start and stop on putty </h1>


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
