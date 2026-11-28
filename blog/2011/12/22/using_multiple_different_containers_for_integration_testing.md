# Using multiple different containers for integration testing

In the previous article we have worked out how to run the integration tests against Glassfish, but what about other versions of Glassfish or even other servers?

Well, the infrastructure groundwork has been laid for that. Before we are going to reap the fruits we are going to explain the difference between downloading, installing and configuring your server within Cargo.

The main reason for explaining it now is that we want to decouple the installing and configuring a bit, so that you understand how you can customize a particular server to suit your needs.

For this article we are going to assume you have downloaded the [zip file](htmlunit4.zip) and that you are following along. Lets download and install Glassfish 3.1.1 one more time.

Use the following commandline at one of the CHILD projects:

```
mvn -Dintegration.skip=false -Dintegration.container.skip=false 
    -Dintegration.container.downloadUrl=http://dlc.sun.com.edgesuite.net/glassfish/3.1.1/release/glassfish-3.1.1.zip 
    -Dintegration.container.downloadDir=/tmp/cargo/downloads
    -Dintegration.container.installDir=/tmp/cargo/install/glassfish3.1.1 
    -Dintegration.container.runtimeDir=/tmp/cargo/runtime/glassfish3.1.1 
    -Dintegration.container.type=standalone cargo:install
```

Note if you do not do it at the CHILD project level it will install the container multiple times. While it works it takes quite a bit longer as it is going to install it n + 1 times, where n is the number of child projects.

After issueing this commandline you should have a directory /tmp/cargo/downloads which will contain the glassfish-3.1.1.zip file. And a directory /tmp/cargo/install/glassfish3.1.1 that contains the extracted zip file in Cargo layout. You have now installed a server as far as Cargo is concerned. The next step is to configure it, we like to call that setting up the runtime.  Use the following commandline at the CHILD level again:

```
mvn -Dintegration.skip=false -Dintegration.container.skip=false 
    -Dintegration.container.downloadUrl=http://dlc.sun.com.edgesuite.net/glassfish/3.1.1/release/glassfish-3.1.1.zip 
    -Dintegration.container.downloadDir=/tmp/cargo/downloads
    -Dintegration.container.installDir=/tmp/cargo/install/glassfish3.1.1 
    -Dintegration.container.runtimeDir=/tmp/cargo/runtime/glassfish3.1.1 
    -Dintegration.container.type=standalone cargo:configure
```

You should now see a directory /tmp/cargo/runtime/glassfish3.1.1. This contains the runtime configuration for the Glassfish runtime you are going to use. If you need to do anything specific to the Glassfish domain this is directory you will do the customization at.

The next step is now to do the actual testing, use the following commandline at the PARENT level:

```
mvn -Dintegration.skip=false -Dintegration.container.skip=false 
    -Dintegration.container.downloadUrl=http://dlc.sun.com.edgesuite.net/glassfish/3.1.1/release/glassfish-3.1.1.zip 
    -Dintegration.container.downloadDir=/tmp/cargo/downloads
    -Dintegration.container.installDir=/tmp/cargo/install/glassfish3.1.1 
    -Dintegration.container.runtimeDir=/tmp/cargo/runtime/glassfish3.1.1 
    -Dintegration.container.type=existing clean verify
```

This will execute the tests for all the child projects. Note that the integration.container.type is set to existing here. This is so it reuses the configuration you setup in the previous steps. You will see output similar to the output below:

```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] ------------------------------------------------------------------------
[INFO] pom ................................................... SUCCESS [31.105s]
[INFO] htmlunit4 ............................................. SUCCESS [28.045s]
[INFO] htmlunit4b ............................................ SUCCESS [28.312s]
[INFO] ------------------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1 minute 27 seconds
[INFO] Finished at: Fri Dec 23 00:01:37 CST 2011
[INFO] Final Memory: 51M/504M
[INFO] ------------------------------------------------------------------------
```

You'll notice that the actual build time for testing is down a bit from the previous article.

Now it is time to use another version of Glassfish, lets use a promoted build of 3.1.2. Actually since the article shows you the steps I am just going to give you each of the commandline properties you need to adjust and leave the rest to you as an exercise. I will also give you the same for a promoted build of Glassfish 4.0 and Tomcat 7.0.29.

## Properties for Glassfish 3.1.2

```
integration.container.id=glassfish3x 
integration.container.downloadUrl=http://dlc.sun.com.edgesuite.net/glassfish/3.1.2/promoted/glassfish-3.1.2-b15.zip 
integration.container.installDir=/tmp/cargo/install/glassfish3.1.2
integration.container.runtimeDir=/tmp/Cargo/runtime/glassfish3.1.2
```

## Properties for Glassfish 4.0

```
integration.container.id=glassfish3x 
integration.container.downloadUrl=http://dlc.sun.com.edgesuite.net/glassfish/4.0/promoted/glassfish-4.0-b16.zip 
integration.container.installDir=/tmp/cargo/install/glassfish4.0 
integration.container.runtimeDir=/tmp/cargo/runtime/glassfish4.0 
```

## Properties for Tomcat 7.0.23

```
integration.container.id=tomcat7x 
integration.container.downloadUrl=http://apache.mirrors.pair.com/tomcat/tomcat-7/v7.0.23/bin/apache-tomcat-7.0.23.zip 
integration.container.installDir=/tmp/cargo/install/tomcat7.0.23 
integration.container.runtimeDir=/tmp/cargo/runtime/tomcat7.0.23 
```

As you can see supporting another version or a different application server is now quite easy to accomplish!

Let me know what you think of this article. Note if you went through all the steps for Tomcat 7.0.23 you will notice it fails the tests. This is by design. We did not ship the required JSF jars. Our next article will touch on that ;)

Posted December 22, 2011

[Up](../../../index.html)
