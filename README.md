# simple-java-maven-app

This repository is for the
[Build a Java app with Maven](https://jenkins.io/doc/tutorials/build-a-java-app-with-maven/)
tutorial in the [Jenkins User Documentation](https://jenkins.io/doc/).

The repository contains a simple Java application which outputs the string
"Hello world!" and is accompanied by a couple of unit tests to check that the
main application works as expected. The results of these tests are saved to a
JUnit XML report.

Bellow are described all the steps for setup
=============================================
               CI/CD pipeline on Jenkins 

Bellow, we will give details for building the infrastructure, using CI/CD pipelines from Dev to Test of any artefact (JAR/WAR).
1. Create infrastructure on Cloud AWS.
First, we need to create 2 Infrastructure “Linux AWS” on “console.aws.amazon.com/ec2”, that we have to use for test & deployed
    • previously we need to create a “key pairs” for access Amazone with fingerprint

devops
6d:b2:12:7d:23:07:e3:de:65:ea:4c:fc:dc:e1:16:67:c2:6f:cb:f8
  Key “devops.ppk” Is generated for password-less Instance access, and with default user Instance “em2-user”
    • first instance. we will  use for “Jenkins” , Public IP and DNS

ec2-3-15-215-141.us-east-2.compute.amazonaws.com
3.15.215.141

    • second instance, we will use for “Tomcat & Maven”, Public IP and DNS:

ec2-18-218-175-232.us-east-2.compute.amazonaws.com
18.218.175.232
    • In order to manage access, we need to adapt and create “security groups”
first by default is created ssh :22 port incoming access, for accessing by ssh instances.
Second we need to create other security group for access requirements between instances.
We have to create “webaccess “ access group. For allow “incoming inbound” port 8080, 8090, tomcat & jenkins. And also attach to the respective instance.

2. Jenkins Installation

Firs we have to login with ssh 3.15.215.141 with “devops.ppk” key.
Download and Install JenkinsTo download and install Jenkins:
1.To ensure that your software packages are up to date on your instance, use the following command to perform a quick software update:
[ec2-user ~]$ sudo yum update –y 
2.Add the Jenkins repo using the following command: 
[ec2-user ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo 
3.Import a key file from Jenkins-CI to enable installation from the package: 
[ec2-user ~]$ sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key 
4.Install Jenkins: [ec2-user ~]$ sudo yum install jenkins -y
5.Start Jenkins as a service: [ec2-user ~]$ sudo service jenkins start




Configure Jenkins:

Configure JenkinsJenkins is now installed and running on your EC2 instance. To configure Jenkins: 
1.Connect to http://3.15.215.141:8080 from your favorite browser. You will be able to access Jenkins through its management interface: 
2.As prompted, enter the password found in /var/lib/jenkins/secrets/initialAdminPassword. Use the following command to display this password:[ec2-user ~]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword 
3.The Jenkins installation script directs you to the Customize Jenkinspage. Click Install suggested plugins.
 4.Once the installation is complete, enter Administrator Credentials, click Save Credentials, and then clickStart Using Jenkins. 
Credentials : devops/*****
5.On the left-hand side, click Manage Jenkins, and then clickManage Plugins. 6.Click on the Available tab, and then enterAmazon EC2 plugin at the top right.
7.Select the checkbox next to Amazon EC2 plugin,and thenclick Install without restart.”deploy “ , “github”, “maven”.
8.Once the installation is done, click Go back to the top page. 
9.Click on Manage Jenkins, and then Configure System. 
10.Scroll all the way down to the section that says Cloud. 
11.Click Add a new cloud, and select Amazon EC2. A collection of new fields appears.
12.Fill out all the fields. (Note: You will have to Add Credentials of the kind AWS Credentials.)We are now ready to use EC2 instances as Jenkins build slaves 



3 Installation web serverive Tomcat9

Next step is to prepare web server, where the .war code will be deployed, and the application has to be accessed.
http://18.218.175.232:8090
Installation & configuration
make a update on the instance and install java8
    • sudo yum update && yum install java-1.8.0
    • wget packages “apache-tomcat-9.0.33.tar.gz” on directory 
    • useradd tomcat && su - tomcat
      usr/local/tomcat9/  and unzip tar -xvf  apache-tomcat-9.0.33.tar.gz
    • echo "export CATALINA_HOME='/opt/tomcat/'" >> ~/.bashrc && source ~/.bashrc
    • Configuration :
create user in tomcat (user tomcat bellow will be used on jenkins automation)
<!-- User tomcat who can access only manager section -->
<role rolename="manager-gui" />
<role rolename="manager-script" />
<user username="tomcat" password="tomcat" roles="manager-gui,manager-script" />

<!-- User Admin Who can access manager and admin section both -->
<role rolename="admin-gui" />
<user username="admin" password="tomcat" roles="admin-gui" />

Allow permission for any external IP for /manager and other hosts in /webapp
<Context>

    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>WEB-INF/tomcat-web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
    <Manager pathname="" />
    -->

<Context antiResourceLocking="false" privileged="true" >
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
 allow=".*" />

</Context>
    • Configuration credential on Maven & install sudo yum install maven
%MAVEN_PATH%/conf/settings.xml
<?xml version="1.0" encoding="UTF-8"?>
<settings ...>
        <servers>
           
                <server>
                        <id>TomcatServer</id>
                        <username>tomcat</username>
                        <password>*****</password>
                </server>

        </servers>
</settings>


4. Create GITHup account and repository

https://github.com/denis2286/simple-devops.git
This repo is public and is for demonstrating the process of testing and deploy on to web server .war file.
For jenkins integration we need to create an 
Create a GitHub Webhook
In order to integrate the GitHub repository into Jenkins, a webhook can be used to run the Jenkins build whenever a code commit is made in GitHub. To create the GitHub webhook, execute the following steps:
    • In Github, move to your repository and select “Settings”. 
    • On the left-hand side, select “Webhooks”. 
    • Click on the “Add webhook” button. 
    • In the “Payload URL”, enter http://3.15.215.141:8080>/github-webhook/.
    • Leave the options with the default selection and click on the “Add webhook”. 


Pom.xml is crated. In our case I have use plugins like bellow:

<plugin>
<groupId>org.apache.tomcat.maven</groupId>
<artifactId>tomcat7-maven-plugin</artifactId>
<version>2.2</version>
<configuration>
<url>http://18.218.175.232:8090/manager/text</url>
<server>tomcat</server>
<path>/test1</path>
</configuration>
</plugin> 


reference link: https://maven.apache.org/guides/introduction/introduction-to-the-pom.html


Create Maven project “simple-java-maven”
->After installing necessary plugins, described previously. We are ready to create a new Project:
->As we can see the project in git is Maven. So we  can create directly maven project.

Putting parameter like bellow:
Since the access in git is public for this repo. We don’t use credential otherwise we must create credential for git access.

We should use pom.xml that is located in root of the repo git.


Adding credential for tomcat web server in order to build and send .war file.
Test building
========
<===[JENKINS REMOTING CAPACITY]===>channel started
Executing Maven:  -B -f /var/lib/jenkins/workspace/simple-java-maven/pom.xml clean install
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------------< tomcat:test1 >----------------------------
[INFO] Building test1 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ test1 ---
[INFO] Deleting /var/lib/jenkins/workspace/simple-java-maven/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ test1 ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /var/lib/jenkins/workspace/simple-java-maven/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ test1 ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /var/lib/jenkins/workspace/simple-java-maven/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ test1 ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /var/lib/jenkins/workspace/simple-java-maven/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ test1 ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /var/lib/jenkins/workspace/simple-java-maven/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ test1 ---
[INFO] Surefire report directory: /var/lib/jenkins/workspace/simple-java-maven/target/surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.mycompany.app.AppTest
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.023 sec

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

[JENKINS] Recording test results
[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ test1 ---
[INFO] Building jar: /var/lib/jenkins/workspace/simple-java-maven/target/test1-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- maven-install-plugin:2.4:install (default-install) @ test1 ---
[INFO] Installing /var/lib/jenkins/workspace/simple-java-maven/target/test1-1.0-SNAPSHOT.jar to /var/lib/jenkins/.m2/repository/tomcat/test1/1.0-SNAPSHOT/test1-1.0-SNAPSHOT.jar
[INFO] Installing /var/lib/jenkins/workspace/simple-java-maven/pom.xml to /var/lib/jenkins/.m2/repository/tomcat/test1/1.0-SNAPSHOT/test1-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  6.191 s
[INFO] Finished at: 2020-04-08T07:44:37Z
[INFO] ------------------------------------------------------------------------
[JENKINS] Archiving /var/lib/jenkins/workspace/simple-java-maven/pom.xml to tomcat/test1/1.0-SNAPSHOT/test1-1.0-SNAPSHOT.pom
[JENKINS] Archiving /var/lib/jenkins/workspace/simple-java-maven/target/test1-1.0-SNAPSHOT.jar to tomcat/test1/1.0-SNAPSHOT/test1-1.0-SNAPSHOT.jar
channel stopped
ERROR: Step ‘Deploy war/ear to a container’ aborted due to exception: 
java.lang.InterruptedException: [DeployPublisher][WARN] No wars found. Deploy aborted. %n
        at hudson.plugins.deploy.DeployPublisher.perform(DeployPublisher.java:107)
        at hudson.tasks.BuildStepCompatibilityLayer.perform(BuildStepCompatibilityLayer.java:78)
        at hudson.tasks.BuildStepMonitor$3.perform(BuildStepMonitor.java:45)
        at hudson.model.AbstractBuild$AbstractBuildExecution.perform(AbstractBuild.java:741)
        at hudson.model.AbstractBuild$AbstractBuildExecution.performAllBuildSteps(AbstractBuild.java:690)
        at hudson.maven.MavenModuleSetBuild$MavenModuleSetBuildExecution.post2(MavenModuleSetBuild.java:1074)
        at hudson.model.AbstractBuild$AbstractBuildExecution.post(AbstractBuild.java:635)
        at hudson.model.Run.execute(Run.java:1905)
        at hudson.maven.MavenModuleSetBuild.run(MavenModuleSetBuild.java:543)
        at hudson.model.ResourceController.execute(ResourceController.java:97)
        at hudson.model.Executor.run(Executor.java:428)
Finished: FAILURE
============
This task can be automated , for every minutes schedule , putting on SCM configuration
*/1 * * * * 









