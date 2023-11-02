# Tomcat

Apache Tomcat, often referred to simply as Tomcat, is an open-source web server and servlet container developed by the Apache Software Foundation. It is designed to host and execute Java-based web applications, specifically those that use Java Servlet and JavaServer Pages (JSP) technologies.


## Install Tomcat

```bash
sudo su
cd /opt
sudo wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.tar.gz
sudo tar -xvf apache-tomcat-9.0.65.tar.gz
```

Navigate to Tomcat's configuration directory:

```bash
cd /opt/apache-tomcat-9.0.65/conf
```

Edit the `tomcat-users.xml` file and add the following line at the end (2nd-last line):

```xml
<user username="admin" password="admin1234" roles="admin-gui, manager-gui, manager-script"/>
```

Create symbolic links for Tomcat startup and shutdown scripts:

```bash
sudo ln -s /opt/apache-tomcat-9.0.65/bin/startup.sh /usr/bin/startTomcat
sudo ln -s /opt/apache-tomcat-9.0.65/bin/shutdown.sh /usr/bin/stopTomcat
```

Edit the `context.xml` files for the manager and host-manager web applications to comment out a section:

```bash
sudo vi /opt/apache-tomcat-9.0.65/webapps/manager/META-INF/context.xml
```

Comment out the following section:

```xml
<!-- Valve className="org.apache.catalina.valves.RemoteAddrValve"
allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
```

Similarly, edit the `context.xml` file for the host-manager:

```bash
sudo vi /opt/apache-tomcat-9.0.65/webapps/host-manager/META-INF/context.xml
```

Comment out the same section as above:

```xml
<!-- Valve className="org.apache.catalina.valves.RemoteAddrValve"
allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" -->
```

To stop and start Tomcat, use the following commands:

```bash
sudo stopTomcat
sudo startTomcat
```

Remember to replace the username and password in the `tomcat-users.xml` file with your preferred values for added security.

```

