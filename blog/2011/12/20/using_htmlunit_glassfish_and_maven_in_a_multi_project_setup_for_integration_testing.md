# Using HtmlUnit, Glassfish and Maven in a multi-project setup for integration testing

In the previous article we added the Glassfish application server into the mix for testing. This looks all nice, but one might wonder how this would work in a parent project (POM) and child projects setup. Well it is fairly easy to accomplish.

We will move all the bits first over to the new parent POM we are going to use. See below for what the parent POM will look like.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project 
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.manorrock.demo.integration</groupId>
    <artifactId>integration-pom</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>pom</name>
    <build>
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
                                <integration.base.url>${integration.container.url}</integration.base.url>
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
                            <extractDir>${integration.container.root}</extractDir>
                        </zipUrlInstaller>
                    </container>
                    <configuration>
                        <type>standalone</type>
                        <home>${integration.container.home}</home>
                    </configuration>
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
    <modules>
        <module>integration-htmlunit3</module>
        <module>integration-htmlunit3b</module>
    </modules>
    <properties>
        <integration.skip>true</integration.skip>
        <integration.container.home>target/container.home</integration.container.home>
        <integration.container.id>glassfish3x</integration.container.id>
        <integration.container.downloadDir>C:/Temp/downloads</integration.container.downloadDir>
        <integration.container.downloadUrl>http://download.java.net/glassfish/3.1.1/release/glassfish-3.1.1.zip</integration.container.downloadUrl>
        <integration.container.root>target/container.install</integration.container.root>
        <integration.container.skip>true</integration.container.skip>
        <integration.container.url>http://localhost:8080/${project.build.finalName}/</integration.container.url>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>    
    <reporting>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-report-plugin</artifactId>
                <version>2.11</version>
                <reportSets>
                    <reportSet>
                        <id>integration-tests</id>
                        <reports>
                            <report>failsafe-report-only</report>
                        </reports>
                    </reportSet>
                </reportSets>
            </plugin>
        </plugins>
    </reporting>
</project>
```

The parent POM file looks quite similar to the POM file of the previous article. Now lets introduce the POM file for the child project.

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd"> 
    <modelVersion>4.0.0</modelVersion> 
    <parent> 
        <groupId>com.manorrock.demo.integration</groupId> 
        <artifactId>integration-pom</artifactId> 
        <version>1.0.0-SNAPSHOT</version> 
    </parent> 
    <groupId>com.manorrock.demo.integration</groupId> 
    <artifactId>integration-htmlunit3</artifactId> 
    <version>1.0.0-SNAPSHOT</version> 
    <packaging>war</packaging> 
    <name>htmlunit3</name> 
    <build> 
        <finalName>htmlunit3</finalName> 
    </build> 
</project>
```

Note that the parent POM actually has 2 child projects, the POM mentioned above is one of them. The other POM is available in the [zip file](htmlunit3.zip).

To execute the integration tests for all the projects (the parent and child projects) you can use following command line at the parent level:

```sh
mvn -Dintegration.skip=false -Dintegration.container.skip=false clean verify
```

If you just want to test a specifc child project go into the child directory and use the same command line. See the example output below for what a test run against all the projects would look like. Note the output only shows the last few lines.

```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] ------------------------------------------------------------------------
[INFO] pom ................................................... SUCCESS [57.168s]
[INFO] htmlunit3 ............................................. SUCCESS [46.631s]
[INFO] htmlunit3b ............................................ SUCCESS [48.791s]
[INFO] ------------------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2 minutes 32 seconds
[INFO] Finished at: Tue Dec 20 23:53:57 CST 2011
[INFO] Final Memory: 51M/503M
[INFO] ------------------------------------------------------------------------
```

Posted December 20, 2011

[Up](../../../index.html)
