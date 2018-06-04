---
layout: post
title: "Install MediaWiki 1.30 + VisualEditor 1.30 on CentOS 7.5"
categories: [recipe]
comments: true
date: "2018-05-28 17:03:20 -0300"
---

# Install MediaWiki 1.30 Centos 7.5 LAMPS VisualEditor 1.30

## Installing LAMPS (MariaDB, PHP 7.2)
```
yum -y update
yum -y install mariadb-server mariadb

systemctl start mariadb.service
systemctl enable mariadb.service

mysql_secure_installation

yum -y install httpd

systemctl start httpd.service

systemctl enable httpd.service

firewall-cmd --permanent --zone=public --add-service=http 
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload

rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm

yum -y install yum-utils

yum-config-manager --enable remi-php72

yum -y install php php-opcache

yum -y install php-mysqlnd php-pdo

yum -y install php-gd php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-soap curl curl-devel

systemctl restart httpd
```

ref: [How to install Apache, PHP 7.1 and MySQL on CentOS 7.4 (LAMP)](https://www.howtoforge.com/tutorial/centos-lamp-server-apache-mysql-php/)

## Installing MediaWiki 1.30 + (VisualEditor 1.30 + Parsoid 0.8.0)
`yum -y install git` 

Before installing `composer` beware you are using the correct php release repo as set in LAMPS above.  
`yum -y install composer`   

Clone MediaWiki version 1.30 to `/var/www/html` directory  
`git clone https://gerrit.wikimedia.org/r/p/mediawiki/core.git --branch REL1_30 /var/www/html/`  

then update submodules:  
```
cd /var/www/html
git submodule update --init
composer update --no-dev
```

Installing VisualEditor:  
```
cd extensions
git clone https://gerrit.wikimedia.org/r/p/mediawiki/extensions/VisualEditor.git --branch REL1_30
cd VisualEditor
git submodule update --init
```

In order to VisualEditor to work one needs to also install Parsoid:  
`yum -y install nodejs npm vim`  

Installing parsoid to `/opt/parsoid`  
```
mkdir -p /opt/parsoid
git clone https://gerrit.wikimedia.org/r/p/mediawiki/services/parsoid /opt/parsoid
cd /opt/parsoid
git checkout tags/v0.8.0
npm install
```

`cp localsettings.example.js localsettings.js`  
`vim localsettings.js`  
Uncomment ‘parsoidConfid.setMwApi’ and change the ‘uri’ value to your MediaWiki API:  
```
exports.setup = function(parsoidConfig) {
         // Do something dynamic with `parsoidConfig` like,
         parsoidConfig.setMwApi({
                 uri: 'http://wiki.hakase-labs.me/api.php',
          });
 };
```

`cp config.example.yaml config.yaml`  
`vim config.yaml`  
Set ‘url’ and ‘domain’:  
```
mwApis:
         - # This is the only required parameter,
           # the URL of you MediaWiki API endpoint.
           uri: 'http://wiki.hakase-labs.me/api.php'
           # The "domain" is used for communication with Visual Editor
           # and RESTBase.  It defaults to the hostname portion of
           # the `uri` property above, but you can manually set it
           # to an arbitrary string. It must match the "domain" set
           # in $wgVirtualRestConfig.
           domain: 'wiki.hakase-labs.me' 
           #optional
```

Add parsoid’s service to systemd `/etc/systemd/system/parsoid.service`:  
```
[Unit]
 Description=Mediawiki Parsoid web service on node.js
 Documentation=http://www.mediawiki.org/wiki/Parsoid
 Wants=local-fs.target network.target
 After=local-fs.target network.target
 
 [Install]
 WantedBy=multi-user.target
 
 [Service]
 Type=simple
 User=root
 Group=root
 WorkingDirectory=/opt/parsoid
 ExecStart=/usr/bin/node /opt/parsoid/bin/server.js
 KillMode=process
 Restart=on-success
 PrivateTmp=true
 StandardOutput=syslog
```

Then:  
```
systemctl daemon-reload
systemctl start parsoid
systemctl enable parsoid
firewall-cmd --add-port=8000/tcp --permanent
firewall-cmd --reload
```

Add this to your MediaWiki’s LocalSettings:  
```
wfLoadExtension( 'VisualEditor' );
 
 // Enable by default for everybody
 $wgDefaultUserOptions['visualeditor-enable'] = 1;
 
 // Optional: Set VisualEditor as the default for anonymous users
 // otherwise they will have to switch to VE
 // $wgDefaultUserOptions['visualeditor-editor'] = "visualeditor";
 
 // Don't allow users to disable it
 $wgHiddenPrefs[] = 'visualeditor-enable';
 
 // OPTIONAL: Enable VisualEditor's experimental code features
 #$wgDefaultUserOptions['visualeditor-enable-experimental'] = 1;
 
 
 $wgVirtualRestConfig['modules']['parsoid'] = array(
     // URL to the Parsoid instance
     // Use port 8142 if you use the Debian package
     'url' => 'http://wiki.hakase-labs.me:8000',
     // Parsoid "domain", see below (optional)
     'domain' => 'wiki.hakase-labs.me',
     // Parsoid "prefix", see below (optional)
     'prefix' => 'wiki.hakase-labs.me'
 );
```

Remember to change ‘url’, ‘domain’ and ‘prefix’.  	

Enter your wiki http. Click to edit post and voilá. :)

ref:   
[Download from Git - MediaWiki](https://www.mediawiki.org/wiki/Download_from_Git#Fetch_external_libraries)   
[How to Install VisualEditor for MediaWiki on CentOS 7](https://www.howtoforge.com/tutorial/how-to-install-visualeditor-for-mediawiki-on-centos-7/)



## Other useful extensions
Mind checking:
* PdfHandler
* MediaViewer
* SimpleBatchUpload

## Known caveats
Remember to disable SELINUX  
`chmod -R 777 images` for file upload 




