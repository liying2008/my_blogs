---
title: 使用Gradle打包Kotlin项目代码、生成Kotlin代码文档
date: 2017-11-19
tags: [Kotlin, Android, Gradle, javadoc, dokka]
categories: 相关技能
---
# Kotlin项目
在 **Root Project** 下的 <code>build.gradle</code> 文件中 <code>buildscript</code> 下的 <code>dependencies</code> 中添加：

```gradle
classpath "org.jetbrains.dokka:dokka-gradle-plugin:0.9.15"
```

在 **module** 下的 <code>build.gradle</code> 文件中添加：

```gradle
apply plugin: 'org.jetbrains.dokka'


task generateSourcesJar(type: Jar) {
    group = 'jar'
    from sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task dokkaJavadoc(type: org.jetbrains.dokka.gradle.DokkaTask) {
    outputFormat = "javadoc"
    outputDirectory = javadoc.destinationDir
}

task generateJavadoc(type: Jar, dependsOn: dokkaJavadoc) {
    group = 'jar'
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives generateJavadoc
    archives generateSourcesJar
}
```

其中， <code>generateSourcesJar</code> Task 中的  <code>from sourceSets.main.java.srcDirs</code>  需要根据项目实际源码目录填写。

同步之后，在 <code>gradle</code> 的任务列表中的 <code>jar</code> 分组下就可以看到 <code>generateSourcesJar</code> 和  <code>generateJavadoc</code> 两个任务了。
双击这两个任务，<code>sources.jar</code> 和 <code>javadoc.jar</code> 就会生成，位置在 <code>build/libs</code> 目录下。

# 使用Kotlin开发的Android项目
在 **Root Project** 下的 <code>build.gradle</code> 文件中 <code>buildscript</code> 下的 <code>dependencies</code> 中添加：

```gradle
classpath 'org.jetbrains.dokka:dokka-android-gradle-plugin:0.9.15'
```

在 **module** 下的 <code>build.gradle</code> 文件中添加：

```gradle
apply plugin: 'org.jetbrains.dokka-android'


task generateSourcesJar(type: Jar) {
    group = 'jar'
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task dokkaJavadoc(type: org.jetbrains.dokka.gradle.DokkaTask) {
    outputFormat = "javadoc"
    outputDirectory = javadoc.destinationDir
}

task generateJavadoc(type: Jar, dependsOn: dokkaJavadoc) {
    group = 'jar'
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives generateJavadoc
    archives generateSourcesJar
}
```

其中， <code>generateSourcesJar</code> Task 和 <code>javadoc</code> Task 中的  <code>sourceSets.main.java.srcDirs</code>  需要根据项目实际源码目录填写。

同步之后，在 <code>gradle</code> 的任务列表中的 <code>jar</code> 分组下就可以看到 <code>generateSourcesJar</code> 和  <code>generateJavadoc</code> 两个任务了。
双击这两个任务，<code>sources.jar</code> 和 <code>javadoc.jar</code> 就会生成，位置在 <code>build/libs</code> 目录下。

# 相关参考链接

> [Documenting Kotlin Code](http://kotlinlang.org/docs/reference/kotlin-doc.html)
> 
> [Github Kotlin/dokka](https://github.com/Kotlin/dokka)
