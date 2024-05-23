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


### Start when reboot
sudo su

crontab -e

1

Add the following lines, save and exit

@reboot /root/startdspaceservice.sh

@reboot sleep 10 && bash -ci 'pm2 start /opt/dspace-7-angular/dspace-ui.json'

sudo su dspace

crontab -e

Replace:

#GLOBAL VARIABLES
#Full path of your local DSpace Installation (e.g. /home/dspace or /dspace or similar)
#MAKE SURE TO CHANGE THIS VALUE!!!
DSPACE = /dspace
#Shell to use
SHELL=/bin/sh
#Add all major 'bin' directories to path
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
#Set JAVA OPTS with defaults for DSpace Cron Jobs.
#Only provides 512MB of memory by default (which should be enough for most sites).
JAVA_OPTS="-Xmx512M -Xms512M -Dfile.encoding-UTF-8"

#------
#HOURLY TASKS (Recommended to be run multiple times per day, if possible)
#At a minimum these tasks should be run daily.
#Send information about new and changed DOIs to the DOI registration agency NOTE: ONLY NECESSARY IF YOU REGISTER DOIS USING DATACITE AS REGISTRATION AGENCY (Disabled by def
#0 4,12,20 $DSPACE/bin/dspace doi-organiseruq; [dspace]/bin/dspace doi-organiser -s

#DAILY TASKS
#(Recommended to be run once per day. Feel free to tweak the scheduled times below.)
#Update the DAI-PMH index with the newest content at midnight every day
#REQUIRED to update content available in CAI-PH (However, it can be removed if you do not enable 
0 0 * * * $DSPACE/bin/dspace index-discovery > /dev/null
#Cleats and Update the Discovery Indexes at eidnight every day
#(This ensures that any deleted documents are cleaned from the Discovery search/browse Index)
#RECOMMENDED to ensure your search/browse index stays fresh 100 SOSPACE/bin/dspace index-discovery>/dev/null
#run the index-authority script once a day at 12:45. to ensure the Soin Authority coche is up to 
45 0 * * * $DSPACE/bin/dspace index-authority > /dev/null
#Cleanup Wet Spiders from DSpace Statistics Solr Index at 01:00 every day (This resaves any known web spiders from your usage statistics)
#RECOMMENDED If you are running Soir Statistics.
0 1 * * * $DSPACE/bin/dspace stats-uti -i
#Send out "daily update subscription e-mails at 02:00 every day
#(This sends an email to any users who have subscribed to a Community/Collection, notifying the
0 2 * * * $DSPACE/bin/dspace subscription-send -f D

#Run the media filter at 03:00 every day
0 3 * * * $DSPACE/bin/dspace filter-media

#(Recommended to be run coce per week, but can be run more or less frequently, based on your loca
#Send out "weekly update subscription e-mails at 02:00 every Sunday
#(This sends an email to any users who have "subscribed" to a Community/Collection, notifying the REQUIRED for weekly "Email Subscriptions to work properly.
0 2 * * 0 $DSPACE/bin/dspace subscription-send -f W
#Run the checksum checker at 04:00 every Sunday
#By default it runs through every file (-1) and also prunes old results (p)
#(This re-verifies the checksums of all files stored in Space. If any files have been changed/co OPTIONAL, but useful if you want to enable regular regular checksum validation of files stored 04 SOSPACE/bin/dspace checker
#NOTE: LARGER SITES MAY WISH TO USE DIFFERENT OPTIONS. The above "-1" option tells DSpace to chec
#If your site is very large, you may need to only check a portion of your content per week. The t would instead check all the content it can within one hour. The next week it would start again
#04 SDSPACE/bin/dspace checkerd thp the results of the checksum checker (see above) to the configured "sall.admin" at 05:00 ev
#Mail (This ensures the system administrator is notified whether any checksues were found to be differ
#050 SDSPACE/bin/dspace checker-emailer
#MONTHLY TASKS (Recommended to be run once per month, but can be run more or less frequently, based on your 100
#Send out "monthly update subscription e-mails at 02:00, on the first of every month
#(This sends an email to any users who have "subscribed" to a Community/Collection, notifying the 
#REQUIRED for monthly "Email Subscriptions to work properly. 
0 2 1 * * $DSPACE/bin/dspace subscription-send -f M
#Permanently delete any bitstreams flagged as "deleted in DSpace, on the first of every month at 
#(This ensures that any files which were deleted from DSpace are actually removed from your local By default they are just marked as deleted, but are not removed from the filesystem.) 
#REQUIRED to fully remove deleted content files from the "assetstore" folder 
0 1 1 * * $DSPACE/bin/dspace cleanup > /dev/null


exit

nano /root/startdspaceservice.sh

Add the following lines to above file

#!/bin/sh

/opt/solr-8.11/bin/solr stop -force

/opt/solr-8.11/bin/solr start -force

systemctl restart postgresql

systemctl restart tomcat9

chmod +x /root/startdspaceservice.sh

# Tài liệu tham khảo: https://www.youtube.com/watch?v=VKeTo4xuKq8
