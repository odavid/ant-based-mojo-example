# Ant Based Maven Plugins

The following repository is dedicated to explain how to create Ant based Maven plugins.

## Why Ant Based Maven Plugin

Many legacy Java application build scripts are Ant based. 
When a decision is made to convert that code to Maven, usually we start by taking pieces of Ant code and using it internally using antrun plugin. This makes the Maven pom files verbose and hard to maintain. Re-implementing the Ant code as a Maven Plugin can be a tedious work. 
An Ant based mojo can ease the migration from Ant to Maven, by keeping the pom files clean and using declarative Maven plugins configuration, while the actual code can be still using Ant scripts.

## Getting started

A Maven plugin is a jar artifact with the the following convention: 

* artifactId: ${plugin-name}-maven-plugin
* packaging: maven-plugin

In order to start developing an Ant based mojo, you need to the following project structure:

```
pom.xml 									  
src/main/scripts/${plugin-name}.build.xml     (ant build script)
src/main/scripts/${plugin-name}.mojo-desc.xml (simple mojo descriptor)
```


## List of Maven Ant tasks

The following are useful ant tasks that let a plugin developer to exploit "Maven echosystem" within the context of Ant project.
In order to use these tasks, the plugin developer need to add the following taskdef within the build code

```
<project>
	<!-- antcontrib -->
	<taskdef resource="net/sf/antcontrib/antlib.xml"/>
	<!-- adding ant maven tasks to enable integration with maven -->
	<taskdef resource="org/apache/maven/ant/tasks/antlib.xml"/>

	<!-- Exposing Maven properties to ant project -->
	<mavenclasspath/>
	<importmavenprojectproperties/>	
	...
</project>
```

##### mavenclasspath

The mavenclasspath task exposes maven dependencies as path properties. 

* ${compile_classpath} - all compile dependencies
* ${plugin_classpath} - all plugin dependencies
* ${combined_classpath} - a combination of compile and plugin dependencies
* ${test_classpath} - all test dependencies

##### importmavenprojectproperties

By using this task, all maven project.* properties are available within the Ant build context:
${project.basedir}, ${project.build.directory} ${project.build.finalName}, etc...

