# Tomcat

Apache Tomcat, often referred to simply as Tomcat, is an open-source web server and servlet container developed by the Apache Software Foundation. It's used to deploy Java-based web applications, especially those built using Java Servlets and JavaServer Pages (JSP). Tomcat's primary purpose is to provide a platform for running Java web applications and handling HTTP requests.

Here's a breakdown of the key concepts and components of Tomcat:

1. **Servlet Container**: Tomcat functions as a servlet container, which means it provides an environment for running Java Servlets and JSPs. Servlets are Java classes that handle incoming HTTP requests, process them, and generate dynamic responses. JSPs, on the other hand, are text-based templates that allow embedding Java code within HTML to create dynamic web pages.

2. **HTTP Server**: Tomcat can also serve as a web server, handling HTTP requests and responses. This is especially useful for simple web applications or static content. However, it's important to note that Tomcat's web server capabilities are not as extensive as dedicated web servers like Apache HTTP Server or Nginx.

3. **Architecture**: Tomcat consists of several core components that work together to provide its functionality:
   - **Catalina**: This component implements the servlet container and is responsible for managing the lifecycle of servlets, handling HTTP requests, and managing the execution of servlets and JSPs.
   - **Coyote**: Coyote is responsible for handling the low-level network communication. It listens for incoming HTTP requests and passes them to the appropriate servlets or resources.
   - **Connector**: Connectors are responsible for managing communication protocols. For example, the HTTP Connector handles HTTP communication, while other connectors can handle different protocols like AJP (Apache JServ Protocol) for connecting to other web servers.
   - **Host**: A host represents a virtual host within Tomcat. It can contain multiple web applications, each with its own context path.
   - **Context**: A context represents a specific web application within a host. It holds information about the web application, such as its deployment settings and the location of its servlets and JSPs.

4. **Deployment**: Tomcat allows you to deploy web applications as "WAR" (Web Application Archive) files. A WAR file is essentially a packaged version of your web application containing all the necessary files, including servlet classes, JSPs, static resources (HTML, CSS, JS), and configuration files. When you deploy a WAR file to Tomcat, it automatically unpacks the contents and sets up the application for execution.

5. **Configuration**: Tomcat's configuration involves XML files that define settings for various components and behaviors. The main configuration files include `server.xml` (for global settings), `context.xml` (for context-specific settings), and others for connectors and logging.

6. **Management and Monitoring**: Tomcat provides a web-based management console known as the Tomcat Manager. It allows administrators to deploy, undeploy, and manage web applications, as well as monitor server status, resources, and active sessions.

7. **Security**: Tomcat offers various security features such as user authentication, secure socket layer (SSL) support, and access controls to protect your web applications and server.

In summary, Apache Tomcat is a popular Java-based web server and servlet container that facilitates the deployment and execution of Java web applications. It plays a crucial role in serving dynamic content, handling HTTP requests, and managing the lifecycle of servlets and JSPs.

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

## Configure Pipeline As in the video


```groovy
pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your_repo.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Tests') {
            steps {
                sh "mvn test"
            }
        }

        stage('Package') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('Deploy') {
            steps {
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'tomcat-credential',
                        path: '',
                        url: 'http://13.233.144.57:8080/'
                    )
                ],
                contextPath: 'Planview',
                onFailure: false,
                war: 'target/*.war'
            }
        }
    }
}
```

This script defines a Jenkins Declarative Pipeline with several stages. Each stage corresponds to a specific step in your build and deployment process. The script checks out code from a Git repository, compiles it, runs tests, packages the application, and finally deploys it to an Apache Tomcat server.

Make sure to have the necessary Jenkins plugins installed for Git integration, Maven, and the Tomcat deployment adapter (`tomcat9`).

Remember to adjust any configuration values such as Git repository URL, Tomcat server URL, context path, and credentials as needed for your setup.


