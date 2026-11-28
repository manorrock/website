---
title: "HtmlUnit and Maven for integration testing"
date: 2011-12-13
---

# HtmlUnit and Maven for integration testing

If you want to do integration testing using HtmlUnit and Maven the following sets you up for easy testing.

1. Create a Maven web application project.
2. Add index.jsp page (or whatever flavor of framework you want to use).
3. Add HtmlUnit dependency.
4. Write integration test for the index.jsp page.
5. Add the profile to support running of the integration tests separate from the main build cycle.

## Create a Maven web application project

First create the project. Below is how the POM file looks like that I started with. Note that there is nothing specific to HtmlUnit yet.

```
<project 
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.manorrock.demo.maven</groupId>
    <artifactId>htmlunit</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>htmlunit</name>
    <build>
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
    </dependencies>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
</project>
```

## Add index.jsp page

Below the very simple JSP page used for setting up integration testing, add it to `$PROJECT_ROOT/src/main/webapp/`:

```
<%@page contentType="text/html" pageEncoding="UTF-8"%>

<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Testing with HtmlUnit</title>
    </head>
    <body>
        <h1>Testing with HtmlUnit + Maven!</h1>
    </body>
</html>
```

## Add HtmlUnit dependency

The next step is to add the HtmlUnit dependency so we can write our integration test. Add the dependency snippet below to your POM file:

```
<dependency>
    <groupId>net.sourceforge.htmlunit</groupId>
    <artifactId>htmlunit</artifactId>
    <version>2.9</version>
    <scope>test</scope>
</dependency>
```

## Write integration test for the index.jsp page

And now it is time to write the actual integration test, see the code below:

```java
package com.manorrock.demo.htmlunit;

import com.gargoylesoftware.htmlunit.WebClient;
import com.gargoylesoftware.htmlunit.html.HtmlPage;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import static org.junit.Assert.*;

public class HtmlUnitIT {

    private String webUrl;
    private WebClient webClient;

    @Before
    public void setUp() {
        webUrl = System.getProperty("integration.base.url");
        webClient = new WebClient();
    }

    @After
    public void tearDown() {
        webClient.closeAllWindows();
    }

    /**
     * Test /index.html
     */
    @Test
    public void testIndexHtml() throws Exception {
        System.out.println("Connecting to: " + webUrl);
        HtmlPage page = webClient.getPage(webUrl);
        assertTrue(page.getBody().asText().indexOf("Testing with HtmlUnit + Maven!") != -1);
    }
}
```

## Add the integration test profile

To support running the integration test outside of the main build cycle add the following profile:

```
<profiles>
    <profile>
        <id>integration</id>
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
                    <artifactId>maven-failsafe-plugin</artifactId>
                    <version>2.10</version>
                    <executions>
                        <execution>
                            <goals>
                                <goal>integration-test</goal>
                                <goal>verify</goal>
                            </goals>
                            <configuration>
                                <systemPropertyVariables>
                                    <integration.base.url>http://localhost:8080/htmlunit</integration.base.url>
                                </systemPropertyVariables>
                            </configuration>
                        </execution>
                    </executions>                 
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

Note that in order for the integration profile to work you will have to deploy the web application to the application server of your choice.

To run the integration tests use the following command line:

```
mvn -Pintegration verify
```

If you did not deploy the application at http://localhost:8080/htmlunit you will need to set the integration.base.url.

If your application is deployed at http://www.myhost.com/htmlunit, the command line would be:

```
mvn -Dintegration.base.url=http://www.myhost.com/htmlunit -Pintegration verify
```

More about integration in the next article!

[Up](../../../index.md)
