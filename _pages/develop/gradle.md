---
mediawiki: Gradle
title: Gradle
---

[Gradle](http://gradle.org/getting-started-gradle-java/) is an open-source build system using a Groovy-based script syntax as a succinct alternative to the XML of [Ant](http://ant.apache.org/) or [Maven](/develop/maven).

The ImageJ core artifacts are built with Maven, but can also be consumed in a Gradle build script.

## Sample ImageJ build.gradle

Contributed<sup>[1](https://github.com/imagej/tutorials/issues/24)</sup> by {% include person id='reckbo' %}

```groovy
buildscript {
    repositories {
        maven {
            url "https://plugins.gradle.org/m2/"
        }
    }
    dependencies {
        classpath "io.spring.gradle:dependency-management-plugin:0.5.4.RELEASE"
    }
}
apply plugin: "io.spring.dependency-management"

repositories {
    jcenter()
    maven {
            url "https://maven.scijava.org/content/groups/public/"
    }
}
dependencyManagement {
    imports {
        mavenBom 'net.imagej:pom-imagej:14.1.0'
    }
}

dependencies {
    compile 'net.imagej:imagej'
}
```
