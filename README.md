# **Install-Dspace**
Install Dspace on Ubuntu 22.04.4 LST (wsl)
Lời khuyên: để hạn chế config bạn nên đặt cấu hình như mặc định.
username: dspace
password: dspace

## Backend

### Change user to root
sudo su

sudo apt update

sudo apt-get upgrade

### Install git, jdk
sudo add-apt-repository ppa:git-core/ppa

sudo apt update; apt install git

sudo apt install openjdk-11-jdk ant maven

### Install postgresqql
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

echo "deb http://apt.postgresql.org/pub/repos/apt/ jammy"-pgdg main | sudo tee /etc/apt/source.list.d/pgdg.list

// tee: /etc/apt/source.list.d/pgdg.list: No such file or directory

sudo apt-get update

sudo apt-get install postgresql postgresql-client postgresql-contrib libpostgresql-jdbc-java -y

psql -V psql

sudo su postgres

cd /etc/postgresql/14/main

createuser --username=postgres --no-superuser --pwprompt dspace

createdb --username=postgres --owner=dspace --encoding=UNICODE -T template0 dspace

psql --username=postgres dspace -c "CREATE EXTENSION pgcrypto;"

exit

sudo nano /etc/postgresql/14/main/pg_hba.conf

// add following line above # "local" is for Unix ...

local all dspace md5

local all all 127.0.0.1/32 md5

sudo /etc/init.d/postgresql restart


### Add user
sudo useradd -m dspace

sudo passwd dspace

### Install solr

mkdir /dspace

chown dspace:dspace /dspace

mkdir /build

chmod -R 777 /build

sudo mkdir /opt/solr-8.11

cd /opt/solr-8.11

sudo chown -R dspace:dspace /opt/solr-8.11

wget -c https://dlcdn.apache.org/lucene/solr/8.11.3/solr-8.11.3.tgz

tar xvf solr-8.11.3.tgz

cp -rf /opt/solr-8.11/solr-8.11.3/* /opt/solr-8.11/

rm -rf solr-8.11.3 solr-8.11.3.tgz

/opt/solr-8.11/bin/solr start -force

/opt/solr-8.11/bin/solr status

http://localhost:8983/solr

exit

### Download dspace
sudo su

cd /build

wget https://github.com/DSpace/DSpace/archive/refs/tags/dspace-7.6.1.tar.gz

if error: wget: unable to resolve host address ‘github.com’

 nano /etc/resolv.conf
 
add line

nameserver 8.8.8.8

tar zxvf dspace-7.6.1.tar.gz

cd /build/DSpace-dspace-7.6.1/

sudo mvn -U package 

cd dspace/target/dspace-installer

sudo ant fresh_install

cd 

sudo apt-get install tomcat9

sudo nano /lib/systemd/system/tomcat9.service

Add under security

ReadWritePaths=/dspace

sudo cp -R /dspace/webapps/* /var/lib/tomcat9/webapps

sudo cp -R /dspace/solr/* /opt/solr-8.11/server/solr/configsets/

sudo chown -R dspace:dspace /opt/solr-8.11/server/solr/configsets/

sudo nano /etc/profile

add the end

export JAVA_HOME=/user/lib/jvm/java-11-openjdk-amd64

export CATALINA_HOME=/etc/tomcat9

### Start
sudo systemctl restart tomcat9.service

systemctl daemon-reload

sudo systemctl restart tomcat9.service

sudo systemctl enable tomcat9.service

sudo /dspace/bin/dspace create-administrator

dspace@localhost

dspace

dspace

y

sudo /etc/init.d/postgresql restart

/opt/solr-8.11/bin/solr stop -force

/opt/solr-8.11/bin/solr start -force

cp /dspace/config/local.cfg.EXAMPLE /dspace/config/local.cfg

/dspace/bin/dspace database migrate

# config if you want
nano /dspace/config/local.cfg

rest.cors.allowed-origins = ${dspace.ui.url}

cd /root

chown -R tomcat /dspace/upload 

sudo systemctl restart tomcat9.service

http://localhost:8080/server

### Fontend
sudo su

cd

apt-get update && apt-get upgrade -y

apt-get install curl -y

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

source ~/.bashrc

nvm install v18.20.3

npm install --global yarn

cd /root

wget -c https://github.com/DSpace/dspace-angular/archive/refs/tags/dspace-7.6.1.tar.gz

tar zxvf dspace-7.6.1.tar.gz

mv dspace-angular-dspace-7.6.1/ dspace-7-angular

cd dspace-7-angular

yarn install

cp -r /root/dspace-7-angular /opt/

chown dspace:dspace -R /opt/dspace-7-angular

cp /opt/dspace-7-angular/config/config.example.yml /opt/dspace-7-angular/config/config.prod.yml

nano /opt/dspace-7-angular/config/config.prod.yml

ui:

	ssl: false
 
	host: localhost
 
	port: 4000

rest:

	ssl: false
 
	host: localhost
 
	port: 8080
 
save

nano /opt/dspace-7-angular/config/config.yml

rest:

	ssl: false
 
	host: localhost
 
	port: 80
 
	nameSpace: /server

### Install PM2

npm install --global pm2

nano /opt/dspace-7-angular/dspace-ui.json

Add the following lines and save
{

  "apps": [{
  
	"name": "dspace-ui",
 
	"cwd": "/opt/dspace-7-angular",
 
	"script": "dist/server/main.js",
 
	"env": {
 
		"NODE_ENV": "production",
  
		"DSPACE_REST_SSL": "false",
  
		"DSPACE_REST_HOST": "localhost",
  
		"DSPACE_REST_PORT": "8080",
  
		"DSPACE_REST_NAMEPSACE": "/server"
  
		}
  
	}
 
	]
 
}


cd /opt/dspace-7-angular/

yarn build:prod

cd /opt/dspace-7-angular/

systemctl stop tomcat9

/opt/solr-8.11/bin/solr stop -force

pm2 delete all

pm2 start dspace-ui.json

/opt/solr-8.11/bin/solr start -force

systemctl start tomcat9


### To start backend
/opt/solr-8.11/bin/solr start -force

systemctl restart postgresql

systemctl start tomcat9

url: http://localhost:8080/server

### To start frontend
pm2 start /opt/dspace-7-angular/dspace-ui.json

url: http://localhost:4000
# Tài liệu tham khảo: https://www.youtube.com/watch?v=VKeTo4xuKq8
