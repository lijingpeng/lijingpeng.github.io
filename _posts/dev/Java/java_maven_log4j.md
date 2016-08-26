title: log4j POM依赖问题
date: 2016-8-2 11:19:56
tags:
category: java
---

配置log4j
```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.14</version>
</dependency>
```

错误如下：
- Missing artifact javax.jms:jms:jar:1.1:compile
- Missing artifact com.sun.jmx:jmxri:jar:1.2.1:compile
- Failure to transfer com.sun.jmx:jmxri:jar:1.2.1 from https://maven-repository.dev.java.net/nonav/repository was cached in the local
- Missing artifact com.sun.jdmk:jmxtools:jar:1.2.1:compile
- Failure to transfer javax.jms:jms:jar:1.1 from https://maven-repository.dev.java.net/nonav/repository was cached in the local repository
- Failure to transfer com.sun.jdmk:jmxtools:jar:1.2.1 from https://maven-repository.dev.java.net/nonav/repository was cached in the local

解决：
```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.15</version>
    <exclusions>
        <exclusion>
            <groupId>javax.mail</groupId>
            <artifactId>mail</artifactId>
        </exclusion>
        <exclusion>
            <groupId>javax.jms</groupId>
            <artifactId>jms</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jdmk</groupId>
            <artifactId>jmxtools</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jmx</groupId>
            <artifactId>jmxri</artifactId>
        </exclusion>
    </exclusions>
    <scope>runtime</scope>
</dependency>
```