# gradle

## 常用设置

### 修改maven源

在gradel安装目录下的init.d目录创建init.gradle文件，指定以下内容：

```text
allprojects {
    repositories {
        mavenLocal()
        maven { name "Alibaba" ; url 'https://maven.aliyun.com/repository/public/' }
        maven { name "Bstek" ; url 'https://nexus.bsdn.org/content/groups/public/' }
        mavenCentral()
    }
}

buildscript {
    repositories {
        maven { name "Alibaba" ; url 'https://maven.aliyun.com/repository/public/' }
        maven { name "Bstek" ; url 'https://nexus.bsdn.org/content/groups/public/' }
        maven { name "M2" ; url 'https://plugins.gradle.org/m2/' }

    }
}
```

## 常用指令

gradle clean

gradle classes

gradle package

gradle build [-x test]

### wrapper

![wrapper](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221118180917.png)

`gradle  wrapper --gradle-version=xxx` 修改gradle.properties中的wrapper版本，但是并未实际下载

## gradle任务

## gradle插件

>[官方内部插件](https://docs.gradle.org/current/userguide/plugin_reference.html)

![](https://cdn.jsdelivr.net/gh/wshtx/personal_settings/myImageHosting/20221122105553.png)

### 使用自定义插件/脚本

在build.gradle文件中通过`apply from :""`引入本地或者远程脚本文件。

### 使用gradle内部插件

- plugin DSL方式

```gradle
plugins {
    id 'java'
    id 'java-library'    
}
```

- apply具名参数方式

```gradle
//键值对，value：插件id，插件全类名，插件简类名
apply plugin:'value'

apply{
    plugin:'value'
}
```

### 使用第三方插件

- 先引入插件依赖，再apply应用插件：

```gradle
dependencies {
    classPath('org.springframework.boot:spring-boot-gradle-plugin:x.x.x')
}

apply plugin:'org.springframework.boot'
```

- plugin DSL方式，使用此方法引入的插件必须已经托管到[官方网站](https://plugins.gradle.org)上：

```gradle
plugins {
    id 'org.springframework.boot'
}
```

### 使用用户自定义插件

[用户自定义插件](https://docs.gradle.org/current/userguide/custom_plugins.html)

## 依赖

### 依赖类型

由java插件提供:

- compileOnly
- runtimeOnly
- implementation
- testcompileOnly
- testruntimeOnly
- testimplementation
  
war插件提供支持:

- providedCompile
编译，测试阶段需要依赖此类jar包，而运行阶段容器已经提供相应支持，所以无需将这些文件打包进war包中，如servlet-api.jar

由java-library插件提供:

- api
为了避免多模块的重复依赖，使用api类型。但是大量使用api，会显著增加构建时间。
- compileOnlyApi

### 动态版本声明

不建议使用。

- `implementation 'org.slf4j:slf4j-api:+'`
- `implementation 'org.slf4j:slf4j-api:latest.integration'`

### 依赖冲突

当一个项目中依赖了同一个库的不同版本，就会出现依赖冲突。解决办法：

- 默认情况下，gradle使用最新版本的jar包解决冲突。推荐使用
- 使用exclude排除指定依赖包，然后再手动引入指定版本的依赖

```gradle
dependencies{
    implementation('org.hibernate:hibernate-core:3.6.3-final'){
        exclude group: 'org.slf4j' module: 'slf4j-api'
    }
}
```

- 强制使用指定版本号

```java
//第一种写法!!
implementation('org.slf4j:slf4j-api:1.4.0!!')

//第二种写法version
dependencies{
    implementation('org.slf4j:slf4j-api:1.4.0'){
        version{
            strictly('1.4.0')
        }
    }
}
```

### 查看依赖冲突

配置如下时，当gradle构建时遇到依赖冲突，则立即构建失败

```gradle
configurations.all{
    Configuration configuration -> configuration.resolutionStrategy.failOnVersionConflict()
}
```
