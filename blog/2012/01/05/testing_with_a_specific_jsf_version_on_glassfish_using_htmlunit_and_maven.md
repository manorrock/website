# Testing with a specific JSF version on Glassfish using HtmlUnit and Maven

In the previous article we described how to do testing with multiple containers. But since joining the Glassfish team I have wondered: how can I test applications against different versions of JSF on Glassfish?

As it turns out it is actually quite straightforward. This entry shows you how to do it. It will allow you to test using a 2.1.2 version and up, all by using properties passed on the command line.

Download the [zip file](htmlunit5.zip) and follow along. We are going to use the `maven-dependency-plugin` to copy the `javax.faces.jar` into the Glassfish installation we are using for testing.

```xml
<plugin>
    <groupid>org.apache.maven.plugins</groupid>
    <artifactid>maven-dependency-plugin</artifactid>
    <version>2.4</version>
    <executions>
        <execution>
            <id>copy</id>
            <phase>package</phase>
            <goals>
                <goal>copy</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>${jsf.groupId}</groupId>
                        <artifactId>${jsf.artifactId</artifactId>
                        <version>${jsf.version}</version>
                        <type>jar</type>
                        <overwrite>true</overwrite>
                        <outputDirectory>${integration.container.installDir}/glassfish-${glassfish.version}/glassfish3/glassfish/modules</outputDirectory>
                        <destFileName>javax.faces.jar</destFileName>
                    </artificatItem>
                </artificatitems>
                <overWriteReleases>true</overWriteReleases>
                <overWriteSnapshots>true</overWriteSnapshots>
            </configuration>
        </execution>
    </executions>
</plugin>
```

The above plugin execution copies the `javax.faces.jar` file into the Glassfish modules directory during the `package` phase. This is well before the integration phase so we are sure we have the JSF runtime we want to use.

By using `-Djsf.version=2.1.6` on the command line you would test against Mojarra version 2.1.6. Note that the POM file also requires you to pass in the Glassfish version you are using. This is a little quirk in how Cargo installs the container. So a complete command line could look like the one below:

```bash
mvn -Dintegration.skip=false -Dintegration.container.skip=false \
  -Dintegration.container.downloadUrl=http://dlc.sun.com.edgesuite.net/glassfish/3.1.2/promoted/glassfish-3.1.2-b15.zip \
  -Dintegration.container.installDir=/tmp/cargo/install/glassfish3.1.2 \
  -Dintegration.container.runtimeDir=/tmp/cargo/runtime/glassfish3.1.2 \
  -Dintegration.container.type=existing -Dglassfish.version=3.1.2-b15 \
  -Djsf.version=2.1.6 clean verify
```

I hope you liked this article! Let me know what you think, or if you have any suggestions about other content, or if you just want to drop me a note, feel free to do so :)

Posted January 5, 2012

[Up](../../../index.html)
