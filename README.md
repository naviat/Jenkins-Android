# Jenkins-Android
===============

Tutorial how to configure Jenkins with Android Projects Using both Eclipse and Android Studio


Summary

The following repository is a technical reference for a continuous integration of Android projects using Jenkins.

We will go step by step with detailed description to integrate a sample project to Jenkins.
1- install and configure of Jenkins
2- A sample project using both ant and gradle build system
3- Steps to configure a various build variants for our sample project (dev/prod variants)
   * Configure Ant build files using Eclipse ==> (dev/prod variants)
   * Configure Ant build files using Android System ==> (dev/prod variants)


## 1- install of Jenkins

Download jenkins.war file from jenkins website : http://jenkins-ci.org/
You can use Tomcat server to launch Jenkins by saving jenkins.war under Tomcat webapps folder 
(Path exp :C:\Program Files\Apache Software Foundation\Tomcat 6.0\webapps )
Launch tomcat and access Jenkins from : http://localhost:8080/jenkins/

## 2- install Android plugins for Jenkins

To install a new plugin with Jenkins, go under Administrer Jenkins, then select Gestion des plugins.
All you have to do now is to search for plugin and install it.

The following jenkins plugins are necessary to use Android Projects with Jenkins
*Android Emulator Plugin : allow us to launch Android emulator, install app and launch tests
*Ant Plugin : Necessary to compile projects using Ant
*Gradle Plugin : Necessary to compile projects using Gradle
*Subversion Plugin :allow us to get project code source using Tortoise SVN

## 3- Configure Jenkins for Android Project

To Add a new project to Jenkins, we have to create a New Item.
  *Steps to follow are :

  1-Name your Project
  2-Handle Source Code : Set up your repository URL using SVN or GIT 
  3-Build : Choose your android build system Ant/Gradle
  For project using Ant build system : Select option Add Build Step ==> Call Ant
                                       Then specify your ant target (defined in your build file)==>More details further
                                       Finally specify path to your build file. (Ex: Project Parent folder/build.xml)
  
  For project using Gradle build system : Select option Add Build Step ==> Invoke Gradle Script
                                          Select option Gradle Wrapper
                                          Finally specify the task to execute (Ex: clean build) 
                                          Gradle build file Path : app/build.gradle.==>More details further


In the next part, we will discuss more detailed configuration steps in both Ant and Gradle. The purpose of this configurations is to build multiple variant of our application without modifing any single line of code.  
The variants of our sample project is 2 apps , the first for devellopement called dev and the second app is for produciton called prod. The difference betweend these 2 apps will be : App Name,Image Icone,App Version, Package Name Webservice URL . 
After launching our build task, we will have to APK (dev/prod), with custom ressources for both of them.   
And the cool part, is having both flavor(dev/prod) installed at the SAME time in the SAME device, thanks to the configuration of different package name for each variant.  

Obviously, we can push further our customization, by having different behavior for each variant, like payed vs free, ads vs without ads ... depending in the project functionnality.  

Now, lets stop talking and start working. We will begin with Ant build system. Ofently used with Eclipse IDE.  

## 1- Configuration for Ant Build system : (dev vs prod)

To compile and build android projectwith ant, we need to generate build file (build.xml).  
Just go under your project root folder, and execute the following command :   
android update project --path .  
Or from any location :  
android update project --path <path to your project directory>   

This command will create our build file and update all proporties files of the project  
Now we can compile our project with this command : ant debug /or ant release  

To configure multiple variant of our application, we need to define different target in ant build file (build.xml) and add specefic properties file for each variant.  
Ex target "full-debug" for dev:   
```
<target name="full-debug" depends="read-debug-properties, process-template, debug" />
Ex target "full-release" for prod: 
<target name="full-release" depends="read-release-properties, process-template, clean, release" />
```
The release target depends on target read-release-properties, wich will read our custom properties for release target (prod variant)  
Ex variant properties file "release.properties" : under project root folder  
```
<target name="read-release-properties">
		<property file="release.properties" />
</target>
```

We need to create 2 properties file (release.properties and debug.properties)   
Ex of release.properties :  
```
/*** Declare keystore to use and relative informations ***/
key.store=testapp
key.alias=test
key.store.password=testapp
key.alias.password=testapp

/***  Define properties specefic for release target (prod variant)  ***/
app.version.packagename = com.test.app
app.version.code = 1
app.version.name = 1.0
app.name = Test-App-Prod
app.url = http://www.test-app.com/
```
We do the same in the debug.properties with specefic information for dev variant.   
Info : All details will be found under sample project source code.  

All these properties will be used during customization process. We will create custom file (under folder /custom) for all data to modify with each variant.   

#### -Create Manifest file : manifest.xml  
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="@app.version.packagename@"
    android:versionCode="@app.version.code@"
	android:versionName="@app.version.name@" >
	
	<....all permissions and activties ../>
<manifest/>
```
All the informations @...@ will be filled from target.properties files (Ex: release.properties)  

#### - Create Custom ressources files : custom_ressources.xml  
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">@app.name@</string>
    <string name="web_service_url">@app.url@</string>
</resources>
```
#### - Create drawable folders for all images to change: under folder /custom  
/drawable-hdpi  
/drawable-xhdpi  
...  
