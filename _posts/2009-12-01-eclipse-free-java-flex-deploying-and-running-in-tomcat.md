---
layout: post
title: ! 'Eclipse-free Java & Flex: Deploying and Running in Tomcat'
tags:
- Flex
- Java
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Earlier, I (briefly) described [my long-running battle with Eclipse, which Eclipse finally won by dropping the proverbial a-bomb and refusing to start](http://vikinghammer.com/2009/12/01/eclipse-free-java-flex-mxmlactionscript-in-macvim/). If you want to read that first, be my guest. Here, I describe the next step in setting up my current environment.

Part of the deal with escaping from Eclipse is that I no longer get the benefit of its automatic-publishing to a Tomcat container; from now on, I'm going to have to do that myself. Here's how I'm doing it.

First, you'll need an installation of Tomcat. I already had this, and I'd put it in my user-specific Applications directory. It's located at:

    ~/Applications/apache-tomcat-6.0.20/

Yours may differ depending on the version ... but I _do_ recommend putting it in your home directory rather than setting it up in a global location.

Since we're using HTTPS for our applications, the first thing you need to do is go into conf/server.xml and uncomment the Connector for port 8443:

    <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" />

Mine was on line 81 ... yours might be in a different place, but it should be around there.

The next thing you need to do is increase the memory allotted to Tomcat to run; this _may_ not be totally necessary, but if you're running a handful or more services you'll probably get some Out Of Memory / PermGen space errors. I know I did.

Go edit bin/catalina.sh, and put the following line around line 200 (right before it sets the JAVA_OPTS variable):

    JAVA_OPTS="$JAVA_OPTS -server -ms512M -mx1024M 
        -XX:NewSize=256M -XX:MaxNewSize=256M -XX:PermSize=256M 
        -XX:MaxPermSize=256M -Dcom.sun.management.jmxremote 
        -Dcom.sun.management.jmxremote.port=7799 
        -Dcom.sun.management.jmxremote.authenticate=false 
        -Dcom.sun.management.jmxremote.ssl=false "

_* Newlines added for pretty-printing purposes._

What this does is take any existing value of JAVA_OPTS and tacks some more options onto it. I set it to a minimum of 512M and a maximum of 1024M, and then set the PermGen space to 256M (the -XX:NewSize, -XX:MaxNewSize, -XX:PermSize, and -XX:MaxPermSize options are responsible for this). You might be able to get away with less than 256M of PermGen space, but I couldn't, and this works. I initially had my minimum memory set to 256M, but that failed to start up because it claimed it couldn't fit its PermGen space into the heap; that's why I bumped the minimum up so high.

The remaining options are there for enabling JMX on the server. I have some JMX-enabled services working on there, so that's necessary for my purposes. If you need it too, it's worth noting that this is solely for development; if you're putting this into production _you absolutely should **not** turn off the authentication and SSL options_.

The final step is getting your WAR files into the Tomcat webapps directory. I'm using Maven to compile and build, so I've got my WAR files available in each project's target directory. So, to deploy the WARs, I created some scripts in my workspace. The first one deploys the base services, all together:

    echo "base-service-one"
    rm -rf ~/Applications/apache-tomcat-6.0.20/webapps/base-service-one*
    cp base-service-one/web-app/target/base-service-one-webapp-*.war
        ~/Applications/apache-tomcat-6.0.20/webapps/base-service-one.war
    echo "base-service-two"
    rm -rf ~/Applications/apache-tomcat-6.0.20/webapps/base-service-two*
    cp base-service-two/target/base-service-two-*.war 
        ~/Applications/apache-tomcat-6.0.20/webapps/base-service-two.war

_* Newlines added for pretty-printing purposes._

And so on. (Note that your target directories may be in different locations per project, as mine are, depending on what type of project you're dealing with. Be sure to check out each directory structure after Maven's done its thing.) The reason I'm doing "base-service-two-*.war" is because Maven will actually create a file like "base-service-two-0.0.1-SNAPSHOT.war" or something like that, and I don't really care what version it currently is.

Then for each individual project, I have a _separate_ deploy script:

    echo "project-one"
    rm -rf ~/Applications/apache-tomcat-6.0.20/webapps/project-one*
    cp project-one/web-app/target/project-one-webapp-*.war 
        ~/Applications/apache-tomcat-6.0.20/webapps/project-one.war

_* Newlines added for pretty-printing purposes._

The reason for deleting from webapps _before_ copying the WAR over is so that Tomcat knows to re-deploy. I had some trouble with Tomcat detecting new WAR files, so I'm just deleting the old WAR _and_ exploded directory to be safe.

Run your deploy scripts after every time you successfully run "mvn install" ... and you're ready to start Tomcat:

    ~/Applications/apache-tomcat-6.0.20/bin/startup.sh

And, bam. Your services are starting. And you didn't need to touch Eclipse.
