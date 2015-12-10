title: Maven 依赖包打包
date: 2015-10-12 20:19:56
tags:
category: java
---

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.4</version>
    <dependencies>
        <dependency>
            <groupId>com.travelsky.tdp.pkgstock</groupId>
            <artifactId>stock-assembly-descriptor</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <!-- 绑定到maven的package命令 -->
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <ignoreMissingDescriptor>true</ignoreMissingDescriptor>
        <descriptorRefs>
            <descriptorRef>zipAll</descriptorRef>
            <descriptorRef>zipFilterConf</descriptorRef>
            <descriptorRef>zipJsCssOnly</descriptorRef>
            <descriptorRef>zipPicOnly</descriptorRef>
        </descriptorRefs>
    </configuration>
</plugin>
```
