setup-solr-magento-community
============================

How to setup Solr for Magento Community edition version 1.9

This How-To describes the necessary steps to setup Solr in Magento community edition 1.9 using the excellent module which integrates Solr into Magento by Jeroen Vermeulen: https://github.com/jeroenvermeulen/magento-solarium

I used Ubuntu 14 / 12 for this.

The following steps are needed:

* Install Java: ```sudo apt-get install openjdk-7-jdk```
* ```sudo apt-get update```
* Install Tomcat server to serve Solr: ```sudo apt-get install tomcat7 tomcat7-admin```
* Download Solr: ```wget http://archive.apache.org/dist/lucene/solr/4.10.1/solr-4.10.1.tgz``` (or other version)
* Extract it: ```tar -xzf solr-4.10.1.tgz```
* Create a directory for Solr: ```sudo mkdir /opt/solr```
* ```sudo cp -r solr-4.10.1/example/solr/* /opt/solr/```
* ```sudo cp solr-4.10.1/example/webapps/solr.war /opt/solr/```
* ```sudo cp -r solr-4.10.1/example/lib/ext/* /var/lib/tomcat7/shared/```
* Install Jeroen's extension into your magento store, e.g. using modman inside /var/www/magento: ```modman init``` ```modman clone https://github.com/jeroenvermeulen/magento-solarium```
* Copy necessary config files from the module: ```sudo cp /var/www/magento/.modman/magento-solarium/app/code/community/JeroenVermeulen/Solarium/docs/schema.xml /opt/solr/collection1/conf/.```
* ```sudo cp /var/www/magento/.modman/magento-solarium/app/code/community/JeroenVermeulen/Solarium/docs/solrconfig.xml /opt/solr/collection1/conf/.```
* Create data directory: ```sudo mkdir /opt/solr/data```
* Make it readable/writable for Tomcat: ```sudo chown tomcat7 /opt/solr/data```
* Add the data directory to Solr Config: ```sudo vi /opt/solr/collection1/conf/solrconfig.xml -> "<dataDir>${solr.data.dir:/opt/solr/data}</dataDir>"``` (With -> I will imply that you edit the appropriate line in vi text editor)

* Make Tomcat serve Solr: ```sudo vi /etc/tomcat7/Catalina/localhost/solr.xml``` -> 
```
    <?xml version="1.0" encoding="utf-8"?>
    <Context docBase="/opt/solr/solr.war" debug="0" crossContext="true">
    <Environment name="solr/home" type="java.lang.String" value="/opt/solr" override="true"/>.
    </Context>
```
* Setup the logging (otherwise Solr won't run):
  * ```sudo cp /solr-4.10.1/example/lib/ext/* /usr/share/tomcat7/lib```
  * ```sudo cp solr-4.10.1/example/resources/log4j.properties /usr/share/tomcat7/lib/```
  * ```sudo vi /usr/share/tomcat7/lib/log4j.properties -> solr.log=/var/log/solr```
  * ```sudo mkdir /var/log/solr```
  * ```sudo chown tomcat7:tomcat7 /var/log/solr```
  * ```sudo vi /etc/logrotate.d/solr -> /var/log/solr/solr.log { copytruncate daily rotate 5 compress missingok create 640 tomcat7 tomcat7 }``` (This will make sure the log file does not grow indefinitely)
* Restart Tomcat ```sudo /etc/init.d/tomcat7 restart```
* Optional: Secure your Tomcat to only use given IPs:
  * ```sudo vi /etc/tomcat7/context.xml``` -> Add ```<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.0\.0\.1|65\.182\.48\.139"/>``` within <Context> to allow only localhost and our IP address
  * Restart Tomcat again
* Check if Solr is working: http://localhost:8080/solr should show Solr admin panel
* Configure the module by inputting your Solr connection data in the Magento backend in System Configuration
* Afterwards reindex the search index
* Then check if it is working -> should have an auto suggest with images and should be error tolerant, e.g. search for Jacet instead of Jacket should give you results.
