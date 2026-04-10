# MAVEN_20260410
MAVEN_20260410


Maven is Build Automation tool for java project where we can add dependencies libraries to build and compile and deploy the code.


Java Code Example :

For Example in Java Code their are below dependiceis 

MYSQL
AWS CLI
JDBC 

so for maven also we have remote repositiories called MVN repo from here we can get any type of dependcoiies related so single repo and can manage everything
Here MVN repo manage all dependencies for java we can download any plugin and dependencies from central repo maven 

Build means Create a final package by adding all dependencies and make it final package like executable format 

Compile is required 

test case is required 

Deploy in required server 



MVN Repo(Central) ==> (mysql,awscli,jdbc) ==> pom.xml (dependencies ) ==> Source java code ==> Build ==> Compile (class file ) ==> Test ==> Deploy ==> Server ec2 ==> Users 

Maven Alternate tools Gradle ,ANT


To Execute this process in maven we have basic commands to manage projects

TO manage maven project we have to run few goals 

While Running Build process maven download the plugin from cental repo so here request server required internet provisioning to download packages so we can manage the process with out internet also by  creating dedicated serve to integrate artifact toolls
by integrating to manage dependencies internally without even nat gateway or we can download required packages into s3 bucket we can give provision to source java private version

when ever we install mavem .m2 folder will create into server home directory (EC2)

Here .m2 is local repo where dependencies stores locally while taking from central if we run second time request no need to go central repo it takes from .m2 directory 

Python --> pip --> requirements.txt -- >pypi.org repo
nodejs --> npm  -->package.json -->  NPM Repo 
Java -- > mvn  --> pom.xml --> MVN central Repo


Maven Components 
Project Object Model 
pom.xml --> where we can add all dependencies for java project 


TO manage maven project we have to run few goals 
mvn clean : it will clean entrie previous build process files 
mvn compile : convert source code into class files (compilation )
mvn test : Test the test cases 

mvn package it created a final package (jar-java,war-web archive) into current project directory 
mvn install : it creates final package into current directory and .m2 

