# **Install-Dspace**
Install Dspace on Ubuntu 22.04.4 LST (wsl)

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

sudo /etc/init.d/postgresql restart


### Add user
sudo useradd -m dspace

sudo passwd dspace

### Install solr
sudo mkdir /build

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

export CATALINA_HOME=/etc/tomcat

### Start
sudo systemctl restart tomcat9.service

systemctl daemon-reload

sudo systemctl restart tomcat9.service

sudo /dspace/bin/dspace create-administrator

dspace@localhost

dspace

dspace

y

sudo rm -rf /build

sudo /etc/init.d/postgresql restart

/opt/solr-8.11/bin/solr start -force

/opt/solr-8.11/bin/solr stop -force

/opt/solr-8.11/bin/solr start -force

sudo systemctl restart tomcat9.service

http://localhost:8080/server
