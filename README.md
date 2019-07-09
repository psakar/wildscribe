Wildscribe Documentation Generator
==================================

This project is a documentation generator for Wildfly/JBoss EAP. Basically takes the self describing management model
and turns it into HTML. It consists of two parts, the model dumper and the site generator.

See models/README.md for details on dumping models.

To generate the site:

Add any new model versions to models/standalone/versions.txt

Then run (for example):

$ java -Durl=https://wildscribe.github.io -jar site-generator/target/site-generator.jar models/standalone/ ../wildscribe.github.io/


To generate data for single version
-----------------------------------
example for Wildfly 17

cd /tmp
VERSION=17.0.0.Final
# download zip distribution
wget https://download.jboss.org/wildfly/${VERSION}/wildfly-${VERSION}.zip -O /tmp/wildfly-${VERSION}.zip
unzip wildfly-${VERSION}.zip -d /tmp
JBOSS_HOME=/tmp/wildfly-${VERSION}
# generate messages
mvn clean -pl message-dumper package exec:java -Dexec.args="$JBOSS_HOME models/standalone/Wildfly-${VERSION}.messages"
# generate management model
#  start server
$JBOSS_HOME/bin/standalone.sh -c standalone-full-ha.xml &
sleep 4
#  activate extra modules
$JBOSS_HOME/bin/jboss-cli.sh -c --commands=/extension=org.wildfly.extension.rts:add,/extension=org.jboss.as.xts:add,/extension=org.wildfly.extension.datasources-agroal:add
#  dump management model
mvn clean -pl model-dumper package exec:java -Dexec.args="models/standalone/JBoss-EAP-${VERSION}.dmr"
#  kill server
kill $(jps | grep jboss-modules.jar | grep -v -w grep | awk '{print $1}')
rm -rf /tmp/jboss-eap-7.3
# generate site
mvn clean verify -pl site-generator exec:java -Dversions.txt.dir=models/standalone/WildFly-${VERSION}.dmr -Dsite.url="" -Dserver.version=WildFly-${VERSION}
# create zip file
(cd site-generator/target/wildscribe-generated; zip -r ../wildfly-${VERSION}-management-model-reference.zip . )
# to view generated site open in browser file site-generator/target/wildscribe-generated/index.html eg. 
# firefox file://$(pwd)/site-generator/target/wildscribe-generated/index.html &


example for EAP CD17

VERSION=7.3.0.CD17
# download jboss-eap from https://developers.redhat.com/products/eap/download
unzip /tmp/jboss-eap-${VERSION}.zip -d /tmp
JBOSS_HOME=/tmp/jboss-eap-7.3
# generate messages
mvn clean -pl message-dumper package exec:java -Dexec.args="$JBOSS_HOME models/standalone/JBoss-EAP-${VERSION}.messages"
# generate management model
#  start server
$JBOSS_HOME/bin/standalone.sh -c standalone-full-ha.xml &
sleep 4
#  activate extra modules
$JBOSS_HOME/bin/jboss-cli.sh -c --commands=/extension=org.wildfly.extension.rts:add,/extension=org.jboss.as.xts:add,/extension=org.wildfly.extension.datasources-agroal:add
#  dump management model
mvn clean -pl model-dumper package exec:java -Dexec.args="models/standalone/JBoss-EAP-${VERSION}.dmr"
# kill server
kill $(jps | grep jboss-modules.jar | grep -v -w grep | awk '{print $1}')
rm -rf /tmp/jboss-eap-7.3
# generate site
mvn clean verify -pl site-generator exec:java -Dversions.txt.dir=models/standalone/JBoss-EAP-${VERSION}.dmr -Dsite.url="" -Dserver.version=JBoss-EAP-${VERSION}
# create zip file
(cd site-generator/target/wildscribe-generated; zip -r ../jboss-eap-${VERSION}-management-model-reference.zip . )
# to view generated site open in browser file site-generator/target/wildscribe-generated/index.html eg. 
# firefox file://$(pwd)/site-generator/target/wildscribe-generated/index.html &
