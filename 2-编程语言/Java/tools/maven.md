# Maven

## Core concept

### 1.[Archetype mechanism](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html)

Archetype is a Maven project templating toolkit.An archetype is defined as an original pattern or model from which all other things of the same kind are made.Archetype will help authors create Maven project templates for users, and provides users with the means to generate parameterized versions of those project templates.

### 2.[Build lifestyle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

The default lifecycle handles your project deployment, the clean lifecycle handles project cleaning, while the site lifecycle handles the creation of your project's web site.

- default lifetyle
- cleaning lifestyle
- site lifestyle

#### a majority list of default build lifestyle

- validate - validate the project is correct and all necessary information is available
- compile - compile the source code of the project
- test - test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
- package - take the compiled code and package it in its distributable format, such as a JAR.
- verify - run any checks on results of integration tests to ensure quality criteria are met
- install - install the package into the local repository, for use as a dependency in other projects locally
- deploy - done in the build environment, copies the final package to the remote repository for sharing with other developers and projects.

### 3.[POM(Project Object Model)](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)

1. Project Inheritance
    The Super POM is Maven's default POM. All POMs extend the Super POM unless explicitly set, meaning**the configuration specified in the Super POM is inherited** by the POMs you created for your projects.
    If parent POM wasn`t in that specific directory structure (parent pom.xml is one directory higher than that of the module's pom.xml),just as follows:

    ```
    .
     |-- my-module
     |   `-- pom.xml
     `-- parent
         `-- pom.xml
    ```

    The solutions is to address this directory structure (or any other directory structure), we would have to add the `<relativePath>` element to our parent section.

    ```
    <project>
      <modelVersion>4.0.0</modelVersion>
      <parent>
        <groupId>com.mycompany.app</groupId>
        <artifactId>my-app</artifactId>
        <version>1</version>
        <relativePath>../parent/pom.xml</relativePath>
      </parent>
      <artifactId>my-module</artifactId>
    </project>
    ```

2. Project Aggregation
Project Aggregation is similar to Project Inheritance. But instead of specifying the parent POM from the module, it specifies the modules from the parent POM. By doing so, the parent project now knows its modules, and **if a Maven command is invoked against the parent project, that Maven command will then be executed to the parent's modules as well.** To do Project Aggregation, you must do the following:
    - Change the parent POMs packaging to the value "pom".
    - Specify in the parent POM the directories of its modules using `<modules/>` (children POMs).

    If we change the directory structure to the following:

    ```
    .
     |-- my-module
     |   `-- pom.xml
     `-- parent
         `-- pom.xml
    ```

    The answer is by specifying the path to the module.

     ```
     <project>
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.mycompany.app</groupId>
      <artifactId>my-app</artifactId>
      <version>1</version>
      <packaging>pom</packaging>
      <modules>
        <module>../my-module</module>
      </modules>
    </project>
     ```

3. Project Interpolation and Variables
    One of the practices that Maven encourages is don't repeat yourself. However, there are circumstances where you will need to use the same value in several different locations. To assist in ensuring the value is only specified once, Maven allows you to use both your own and pre-defined variables in the POM.
    For example, to access the project.version variable, you would reference it like so:

    ```
     <version>${project.version}</version>
    ```

    One factor to note is that **these variables are processed after inheritance as outlined above**. This means that if a parent project uses a variable, then its definition in the child, not the parent, will be the one eventually used.

### 4.[Plugin Configuration](https://maven.apache.org/guides/mini/guide-configuring-plugins.html)

By default, plugin configuration should be propagated to child POMs, so to break the inheritance, you could use the <inherited> tag.

## Practices

#### 1. Transitive dependencies/direct dependencies

- Although transitive dependencies can implicitly include desired dependencies, it is a good practice to explicitly specify the dependencies your source code uses directly.
    For example, assume that your project A specifies a dependency on another project B, and project B specifies a dependency on project C. If you are directly using components in project C, and you don't specify project C in your project A, it may cause build failure when project B suddenly updates/removes its dependency on project C.
- Maven also provides `dependency:analyze` plugin goal for analyzing the dependencies: it helps making this best practice more achievable.
- Another reason to directly specify dependencies is that it provides better documentation for your project: one can learn more information by just reading the POM file in your project, or by executing `mvn dependency:tree`.

### 2. Reference a property defined in your pom.xml/external file

process-resources is the build lifecycle phase where the resources are copied and filtered.And in this phrase maven will put our new property value into.`mvn process-resources`
And We can reference the properties in the application.properties defined as follow.

- (1)the xml elements of pom.xml
- (2)an external file, need to include it in the pom.xml
- (3)defined with the tag `<properties/>`
- (4)the command line arguments,using the `-Dxxx` parameter.

```
# application.properties
application.name=${project.name}
message=${my.filter.value}
cliPro=${command.line.prop}
```

```
# filter.properties
my.filter.value=hello!
```

```
# pom.xml

(1)
<name>test</name>

<build>
    <filters>
    (2)
      <filter>src/main/filters/filter.properties</filter>
    </filters>
<build>

# (3)
 <properties>
    <my.filter.value>hello</my.filter.value>
 </properties>
```

```
mvn process-resources "-Dcommand.line.prop=hello again"
```

### 3. Build multiple modules(projects)
