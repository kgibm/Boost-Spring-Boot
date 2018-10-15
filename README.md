# Boost your Spring Boot app with Open Liberty

Spring Boot is a popular programming model for building cloud native Java applications. Open Liberty is a highly efficient and high performing open source runtime designed to run cloud native Java applications. Together, Spring Boot and Open Liberty combine a cloud native programming model with a cloud native runtime. Docker has emerged as the cloud native deployment technology of choice because Docker provides a highly efficient packaging and deployment solution for applications. Its easy to build Docker images, but to build efficient Docker images, which minimize deployment time and maximize reuse, care needs to be given to how the Docker image is constructed. Enter the `boost-maven-plugin`. The `boost-maven-plugin` helps you package a Spring Boot application with Open Liberty as an uber jar and to build Docker images.

## Introduction

This QuickLab shows you how to package Spring Boot applications with Open Liberty as an uber jar and as a Docker image. The lab uses [Spring Petclinic](https://github.com/spring-projects/spring-petclinic) to demonstrate a Spring Boot application running on Open Liberty.

Open Liberty is a tiny but powerful. Designed from the ground up, Open Liberty is a small, fast, lightweight runtime for cloud native Java applications. Open Liberty's auto-tuning threadpool and advanced security integrations for social login and enterprise security solutions means Open Liberty is enterprise-grade open source.

### Getting started

To get started, clone the Boost Spring Boot sample repository.

Run `git clone https://github.com/barecode/Boost-Spring-Boot.git`

There are three main parts to the sample:

1. Maven build - `pom.xml`
2. Maven wrapper - `mvnw`
3. Spring Petclinic application - `src/...`

The `boost-maven-plugin` is already configured in the `pom.xml` with the `package` execution goal. The `boost-maven-plugin` is included after the `spring-boot-maven-plugin` as it uses the output from the Spring Boot build.

```xml
  <plugin>
    <groupId>io.openliberty.boost</groupId>
    <artifactId>boost-maven-plugin</artifactId>
    <version>0.1</version>
    <executions>
      <execution>
        <goals>
          <goal>package</goal>
        </goals>
     </execution>
   </executions>
  </plugin>
```

The `boost-maven-plugin` packages the Spring Boot application with embedded Open Liberty as an uber jar. Spring Boot applications require an embedded servlet container. The embedded servlet container provides the core technologies needed to host a Java web app. Open Liberty provides the servlet container, along with other capabilities such as SSL/TLS support and websocket support.

## Create an uber jar

### The build

First, build the Spring Petclinic application.

Run `./mvnw package`

This builds the Spring Boot application with an embedded Open Liberty server. Next, run the application and explore Spring Petclinic.

### Run the app

Now run the application.

Run `java -jar target/spring-petclinic-2.0.0.BUILD-SNAPSHOT.jar`

You will notice that Liberty starts and then loads the Spring Boot application.

Browse to `http://localhost:8080/`

You now have a Spring Boot application running on Open Liberty packaged as an uber jar.

## Create a Docker image

Docker is a very popular deployment technology. Docker enables developers to package applications into a deployable image. The Docker image behaves consistently no matter where it is deployed. The consistent behaviour makes Docker ideal for continuous delivery pipelines, because the application behaves the same between development, QA and production. Since the Docker image works the same way everywhere, it makes Docker ideal for delivering on the age old saying 'it works on my machine!' :)

The `boost-maven-plugin` makes it easy to build a Docker image for your Spring Boot application with minimal knowledge of Docker. All that is required is to have Docker installed. Follow the Docker [Getting Started instructions](https://www.docker.com/get-started) to install Docker on your system.

### The build

Configure the `boost-maven-plugin` in the `pom.xml` to include the `docker-build` execution goal. The `docker-build` goal instructs the `boost-maven-plugin` to build a Docker image, creating a Dockerfile if one does not already exist.

```xml
  <plugin>
    <groupId>io.openliberty.boost</groupId>
    <artifactId>boost-maven-plugin</artifactId>
    <version>0.1</version>
    <executions>
      <execution>
        <goals>
          <goal>package</goal>
          <goal>docker-build</goal>
        </goals>
     </execution>
   </executions>
  </plugin>
```

Run `./mvnw clean package`

This produces a Docker image for your Spring Boot application.

### Run the app

Now run the application.

Run `docker run -p 9080:9080 spring-petclinic`

You will notice that Liberty starts and then loads the Spring Boot application.

Browse to `http://localhost:9080/`

The Docker image is [optimized](https://openliberty.io/blog/2018/06/29/optimizing-spring-boot-apps-for-docker.html) to separate the Spring Boot application from the Spring Boot and third-party libraries. These [two layers](https://openliberty.io/blog/2018/09/12/build-and-push-spring-boot-docker-images.html) reduce iterative build and deployment time, and storage cost. The base image is [Open Liberty](https://openliberty.io/) with [Eclipse OpenJ9](https://www.eclipse.org/openj9/), both are designed and optimized for the cloud.

## Docker layers matter

You might be wondering why an 'optimized' Docker image matters. A Docker image is comprised of layers. Each layer adds content to the final Docker image. These are they layers for a Spring Petclinic application built with a simple Dockerfile:

```
$ docker history springio/spring-petclinic
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
94b0366d5ba2        52 seconds ago      /bin/sh -c #(nop)  ENTRYPOINT ["java" "-Djav…   0B                  
213dff56a4bd        53 seconds ago      /bin/sh -c #(nop) COPY file:d3551559c2aa35af…   38.3MB              
bc453a32748e        6 minutes ago       /bin/sh -c #(nop)  ARG JAR_FILE                 0B                  
7fe0bb0d8026        6 minutes ago       /bin/sh -c #(nop)  VOLUME [/tmp]                0B                  
cc2179b8f042        8 days ago          /bin/sh -c set -x  && apk add --no-cache   o…   97.4MB              
<missing>           8 days ago          /bin/sh -c #(nop)  ENV JAVA_ALPINE_VERSION=8…   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV JAVA_VERSION=8u151       0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV PATH=/usr/local/sbin:…   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/lib/jv…   0B                  
<missing>           8 days ago          /bin/sh -c {   echo '#!/bin/sh';   echo 'set…   87B                 
<missing>           8 days ago          /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B                  
<missing>           5 months ago        /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B                  
<missing>           5 months ago        /bin/sh -c #(nop) ADD file:093f0723fa46f6cdb…   4.15MB              
```

The second layer (`213dff56a4bd`) is the entire application fat jar (38.3MB). The Docker image runs exactly the way you’d expect a Spring Boot app to run, however, it suffers from the layering efficiency problem. Every time the application is changed, the entire application JAR is rebuilt and the entire layer is replaced. This happens even if only one line of source code was modified.

Now, inspect the Docker image you just built.

Run `docker history spring-petclinic`.

You will see output similar to this:

```
illarian:Boost-Spring-Boot mcthomps$ docker history spring-petclinic 
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9fbf6d908507        23 seconds ago      /bin/sh -c #(nop) COPY file:9b0b2e869f03a431…   389kB               
24a72a8dc383        3 weeks ago         /bin/sh -c #(nop) COPY dir:d3aa26aeba13c79a8…   34.6MB              
27e322887790        3 weeks ago         /bin/sh -c #(nop)  ARG APP_FILE                 0B                  
95fc6fb1ac6e        3 weeks ago         /bin/sh -c /opt/ol/wlp/bin/server start && /…   63.7MB              
<missing>           3 weeks ago         /bin/sh -c cp /opt/ol/wlp/templates/servers/…   596B                
<missing>           3 weeks ago         /bin/sh -c mkdir -p /opt/ol/wlp/usr/shared/r…   48B                 
<missing>           3 weeks ago         /bin/sh -c #(nop)  CMD ["/opt/ol/wlp/bin/ser…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENTRYPOINT ["/opt/ol/dock…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  EXPOSE 9080/tcp 9443/tcp     0B                  
<missing>           3 weeks ago         /bin/sh -c /opt/ol/wlp/bin/server create    …   623B                
<missing>           3 weeks ago         /bin/sh -c mkdir /logs     && mkdir -p $WLP_…   82B                 
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PATH=/opt/ol/wlp/bin:…   0B                  
<missing>           3 weeks ago         /bin/sh -c apt-get update     && apt-get ins…   142MB               
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV LIBERTY_VERSION=18.0.…   0B                  
<missing>           5 weeks ago         /bin/sh -c #(nop) COPY file:a6eac68a3be2db32…   382B                
<missing>           5 weeks ago         /bin/sh -c #(nop)  LABEL maintainer=Alasdair…   0B                  
<missing>           5 weeks ago         /bin/sh -c #(nop)  ENV JAVA_HOME=/opt/ibm/ja…   0B                  
<missing>           5 weeks ago         /bin/sh -c set -eux;     ARCH="$(dpkg --prin…   192MB               
<missing>           5 weeks ago         /bin/sh -c #(nop)  ENV JAVA_VERSION=1.8.0_sr…   0B                  
<missing>           5 weeks ago         /bin/sh -c apt-get update     && apt-get ins…   6.95MB              
<missing>           5 weeks ago         /bin/sh -c #(nop)  MAINTAINER Dinakar Gunigu…   0B                  
<missing>           5 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           5 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B                  
<missing>           5 weeks ago         /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$…   2.76kB              
<missing>           5 weeks ago         /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
<missing>           5 weeks ago         /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   745B                
<missing>           5 weeks ago         /bin/sh -c #(nop) ADD file:a83ab1826f43e88bc…   115MB   
```

Notice the second layer (`24a72a8dc383`) is now much smaller: 34.6MB down from 38.3MB. This is because the application classes were moved to the top-most layer (`9fbf6d908507`) at 389kB and the embedded application server moved down to a lower layer (`95fc6fb1ac6e`).

This is where layering matters - by changing how the Docker image is built and layered we minimize the amount of time needed to rebuild and re-deploy an image. In the output above, notice that the application layer was built 23 seconds ago, while the rest of the layers are weeks old. These old layers are re-used with the new image, and the delta to push over the network and redeploy is the tiny 389kB application layer.

### Summary

You've seen how easy it is to use Liberty in your Spring Boot project and to build Docker images based on Open Liberty and OpenJ9.

`Liberty ♥ Spring`
