# ---
title: "Using HtmlUnit, Glassfish and Maven for integration testing"
date: 2011-12-15
---
# Using HtmlUnit, Glassfish and Maven for integration testing

In the previous article we described a setup for integration testing. It did not explain how to do automatic deployment to an application server. Since we want to make it convenient to do integration tests we are now going to add Glassfish into the mix.

Note that in the previous article we put the integration testing into a separate profile. We are now going to put it back into the normal build cycle. Do not worry, with the use of command line parameters you will be able to influence what the build cycle will or will not do.

We will be using the following POM:

```
<project
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.manorrock.demo.integration</groupId>
    <artifactId>htmlunit2</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>htmlunit2</name>
    <build>
        <finalName>htmlunit2</finalName>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-site-plugin</artifactId>
                    <version>2.2</version>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.1.1</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>2.10</version>
                <configuration>
                    <skipTests>${integration.skip}</skipTests>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                        <configuration>
                            <systemPropertyVariables>
                                <integration.base.url>http://localhost:8080/htmlunit2/</integration.base.url>
                            </systemPropertyVariables>
                        </configuration>
                    </execution>
                </executions>                 
            </plugin>
            <plugin>
                <groupId>org.codehaus.cargo</groupId>
                <artifactId>cargo-maven2-plugin</artifactId>
                <executions>
                    <execution>
                        <id>start-container</id>
                        <phase>pre-integration-test</phase>
                        <goals>
                            <goal>start</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>stop-container</id>
                        <phase>post-integration-test</phase>
                        <goals>
                            <goal>stop</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <skip>${integration.container.skip}</skip>
                    <container>
                        <containerId>${integration.container.id}</containerId>
                        <zipUrlInstaller>
                            <downloadDir>${integration.container.downloadDir}</downloadDir>
                            <url>${integration.container.downloadUrl}</url>
                        </zipUrlInstaller>
                    </container>
                </configuration>
            </plugin>                
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>6.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>net.sourceforge.htmlunit</groupId>
            <artifactId>htmlunit</artifactId>
            <version>2.9</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <properties>
        <integration.skip>true</integration.skip>
        <integration.container.id>glassfish3x</integration.container.id>
        <integration.container.downloadDir>/tmp/downloads</integration.container.downloadDir>
        <integration.container.downloadUrl>http://download.java.net/glassfish/3.1.1/release/glassfish-3.1.1.zip</integration.container.downloadUrl>
        <integration.container.skip>true</integration.container.skip>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
</project>
```

To execute the integration tests against a running container, you can use the following commandline:

```
mvn -Dintegration.skip=false clean verify
```

This will execute the same build as the integration profile did in the previous article. To add more value, lets now automatically start Glassfish and run the integration tests again, use the following commandline:

```
mvn -Dintegration.skip=false  -Dintegration.container.skip=false clean verify
```

This will execute the integration tests against the Glassfish container that will be downloaded, installed and started automatically.

Note the integration.container.downloadDir property points to the directory where you want Cargo to store the zip file it downloads for the Glassfish release it uses. Note a subsequent invocation will use the downloaded zip file by looking in that directory so do NOT point it to ${project.build.directory}.

Let me know what you think about this article! Was it helpful, too long, boring or whatever comment you have I like to know!

Posted December 15, 2011

[Up](../../../index.html)
