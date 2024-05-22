# **Install-Dspace**
Install Dspace on Ubuntu 22.04.4 LST (wsl)

Prerequisite software:

i) Java JDK

ii) Apache Maven

iii) Apache Ant

iv) Apache Tomcat

v) PostgreSQL

vi) Solr

vii) Dspace 7.6 – backend

viii) Dspace-angular 7.6 – frontend

ix) Node.js

x) Node Version Manager (NVM)

xi) Yarn

xii) gedit (text editor software)

## First update and upgrade your package system
sudo apt update && sudo apt upgrade -y

## Create a Dspace user with password
sudo useradd -m dspace
sudo passwd dspace

## Add dspace user to sudoers group
sudo usermod -aG sudo dspace

## Create the directory for the DSpace installation
cd
sudo mkdir dspace

## Change the dspace folder permission to the dspace user
sudo chown dspace /dspace

## Build the Installation Package
## Install packages to support the Dspace installation
sudo apt install wget curl git build-essential gedit zip unzip -y

## Install Open JDK
sudo apt install openjdk-11-jdk -y

## Set the JAVA_HOME & JAVA_OPTS Environment Variable
sudo mousepad /etc/environment

## Add the below lines at the bottom of the file
JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"
JAVA_OPTS="-Xmx512M -Xms64M -Dfile.encoding=UTF-8"
PATH=${JAVA_HOME}/bin:${PATH}

## Save and close the file.
## Run the following commands to check the status of Java Home & Java OPTS
source /etc/environment
echo $JAVA_HOME
echo $JAVA_OPTS

## Install Maven and Ant
sudo apt install maven ant -y

## Install PostgreSQL
sudo apt-get install postgresql postgresql-client postgresql-contrib libpostgresql-jdbc-java -y

## Check the PostgreSQL version number by running the below command
psql -V psql

sudo pg_ctlcluster 14 main start
sudo systemctl status postgresql

## Create a password for PostgreSQL
sudo passwd postgres [Give a password. for ex. “dspace”]
su postgres

## Exit from the current path
exit

## Open the following file
sudo mousepad /etc/postgresql/14/main/postgresql.conf

## Comment out the line (remove #) listen_addresses = ‘localhost’ under connection settings option.
## Save and exit

## We have to encrypt the security of PostgreSQL. Change the PostgreSQL version no. if required. Open the following file.
sudo mousepad /etc/postgresql/14/main/pg_hba.conf

## Add the following above the line, # Database administrative login by Unix domain socket.
#DSpace configuration
host dspace dspace 127.0.0.1 255.255.255.255 md5

## Restart Postresql
sudo systemctl restart postgresql

## Solr Installation 
Throught docker (https://docs.docker.com/engine/install/ubuntu/)
sudo docker pull solr
sudo docker run -p 8983:8983 -t solr

## Download the DSpace package
sudo mkdir /build

cd /build
sudo wget https://github.com/DSpace/DSpace/archive/refs/tags/dspace-7.6.1.zip
sudo unzip dspace-7.6.1.zip
sudo chmod 777 -R /build

## Install Tomcat
sudo docker pull tomcat
sudo docker run -it --rm tomcat

## Database setup
cd /etc/postgresql/14/main
createuser --username=postgres --no-superuser --pwprompt dspace
createdb --username=postgres --owner=dspace --encoding=UNICODE dspace
psql --username=postgres dspace -c "CREATE EXTENSION pgcrypto;"

cd /build/DSpace-dspace-7.6.1/dspace/config
sudo cp local.cfg.EXAMPLE local.cfg
### See config file
sudo mousepad local.cfg

## Installation of DSpace backend
cd /build/DSpace-dspace-7.6.1/
mvn package

cd dspace/target/dspace-installer
ant fresh_install
