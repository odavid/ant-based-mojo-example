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

## The pom.xml file

The Ant Based Plugin must have few dependencies and few plugins defined in order to work.
I've created an [example parent pom](../../blob/master/ant-mojos-parent/pom.xml) and a [child maven-plugin](../../blob/master/helloworld-maven-plugin/pom.xml) that contains the minimal settings that are needed in order to create Ant based Maven Plugins

## The ${plugin-name}.mojo-desc.xml file

Any Maven plugin is composed of Mojos. Each Mojo implements a single goal and gets its own configuration parameters.
Each parameter has a name, default value and an expression that is bounded to a property to allow configuration of the mojo parameter from the command line by referencing a system property that the user sets.

A mojo definition can also inherit from an abstract mojo definition, which allow several mojos to share the same parameters.

An example for the mojo descriptor file can be found in [here](../../blob/master/helloworld-maven-plugin/src/main/scripts/helloworld.mojo-desc.xml)

## The ${plugin-name}.build.xml file

This file is actually a pure Ant build script. Each mojo should have a corresponding Ant target with the same name.
There is a boilerplate code that needs to be set in order to get connectivity with Maven properties and API

```
<project>
	<!-- *************************************************************************** -->
	<!-- Boilerplate - Add this to any Ant based Plugin -->
	<taskdef resource="net/sf/antcontrib/antlib.xml"/>
	<taskdef resource="org/apache/maven/ant/tasks/antlib.xml"/>
	<mavenclasspath/>
	<importmavenprojectproperties/>	
	<!-- *************************************************************************** -->

	<!-- Mojo Target - target name equals to the mojo name, all mojo parameters are exposed as ant properties -->
	<target name="hello-world">
		<echo>The following is a basic ant based mojo, it takes to parameters and prints them to the screen...</echo>
		<echo>${message} ${name}</echo>
	</target>
</project>
```

An example for the mojo build.xml file can be found in [here](../../blob/master/helloworld-maven-plugin/src/main/scripts/helloworld.build.xml)


## List of Maven Ant tasks

The following are useful ant tasks that let a plugin developer to exploit "Maven echosystem" within the context of Ant project.
In order to use these tasks, the plugin developer need to add the following taskdef within the build code


##### mavenclasspath

The mavenclasspath task exposes maven dependencies as path properties. This task is part of the boilerplate code

* ${compile_classpath} - all compile dependencies
* ${plugin_classpath} - all plugin dependencies
* ${combined_classpath} - a combination of compile and plugin dependencies
* ${test_classpath} - all test dependencies

##### importmavenprojectproperties

By using this task, all maven project.* properties are available within the Ant build context such as ${project.basedir}, ${project.build.directory} ${project.build.finalName}, etc...
This task is part of the boilerplate code

##### attachartifact

Attaching a file as additional artifact to the project running the plugin

```
<attachartifact file="" type="" classifier=""/>
```

##### export-pom-property

Exports a property to Maven context, so it can be used by other plugin configuration

```
<export-pom-property property="<the-name-of-the-property>" value="<value-from-ant>" overwrite="true|false (default=false)"/>
```

##### execute-maven-plugin

Sometimes you want to execute a goal of other plugin within your plugin, so the operation of the plugin will be atomic.

For example
```
<target name="my-mojo">
	...
	...
	<echo>Copying some dependencies to ${project.build.directory}/alternateLocation</echo>
	<execute-maven-plugin artifactId="maven-dependency-plugin" groupId="org.apache.maven.plugins" version="2.10" goal="copy-dependencies">
		<configuration>
			<includeGroupIds>${project.groupId}</includeGroupIds>
			<outputDirectory>${project.build.directory}/alternateLocation</outputDirectory>
		</configuration>
	</execute-maven-plugin>
	...

</target>
```