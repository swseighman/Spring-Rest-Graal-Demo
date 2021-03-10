## Spring Boot REST Demo with GraalVM Native Image
    

![spring](images/spring.png)

With this basic Spring Boot REST example, we demonstrate three options for building a Spring Boot native application:

* Using GraalVM **native image** **Maven plugin** support to generate a native executable
* Using Spring Boot **Maven Buildpacks** support to generate a container running a native executable
* Using a **multi-stage Dockerfile** to build a container to generate a container running a native executable

**Note**: Throughout this exercise, when you see a ![red computer](images/userinput.png) icon, it indicates a command that you'll need to enter in your terminal. ### CreditsThe contents of this tutorial is based on a Spring Native for GraalVM example [here](https://repo.spring.io/milestone/org/springframework/experimental/spring-graalvm-native-docs/0.8.5/spring-graalvm-native-docs-0.8.5.zip!/reference/index.html#_graalvm).

----

### Let's get started!

![user input](images/userinput.png)

```
$ git clone https://github.com/swseighman/Spring-Rest-Graal-Demo.git
```

### Build a Native Image Executable

To build a native image locally, execute this command:

![user input](images/userinput.png)

```
$ cd Spring-Rest-Graal-Demo
$ mvn -DskipTests -Pnative clean package
```

You can then run the executable:

![user input](images/userinput.png)

```
$ target/graalvm-restservice
```

Now that the service is up, visit **localhost:8080/greeting** or `curl` where you should see:

![user input](images/userinput.png)

```
$ curl localhost:8080/greeting
{"id":1,"content":"Hello, World!"}
```

And you also could run the `jar` example:

![user input](images/userinput.png)

```
java -jar target/rest-service-0.0.1-SNAPSHOT.jar
```


### Building a container

If you're on a Linux system, you can quickly build a container by copying the native image executable you just created in the steps above (see `Dockerfile.distroless`):

```
$ docker build -f Dockerfile.distroless -t graalvm-restservice:distro .
```
```
$ docker images
REPOSITORY              TAG        IMAGE ID       CREATED          SIZE
graalvm-restservice     distro     3db75db72e1f   12 minutes ago   82.4MB
```
```
$ docker run -p 8080:8080 graalvm-restservice:distro
```

If you're not using a Linux system, there are two options available for demonstrating a container build:

* Using Spring Boot Maven Buildpacks support to generate a lightweight container containing a native executable
* Using a multi-stage Dockerfile to build a container to generate a container containing a native executable

In the first example, we'll use buildpacks.

Edit the `pom.xml` file and uncomment the `build` section (lines 54-77):

```
<build>
   <plugins>
      <plugin>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <plugin>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-maven-plugin</artifactId>
         <configuration>
            <image>
               <builder>paketobuildpacks/builder:tiny</builder>
               <env>
                  <BP_BOOT_NATIVE_IMAGE>true</BP_BOOT_NATIVE_IMAGE>
                  <BP_BOOT_NATIVE_IMAGE_BUILD_ARGUMENTS>
                     -Dspring.native.remove-yaml-support=true
                     -Dspring.spel.ignore=true
                 </BP_BOOT_NATIVE_IMAGE_BUILD_ARGUMENTS>
               </env>
            </image>
         </configuration>
       </plugin>
   </plugins>
</build>
```

To build a container, execute this command:

![user input](images/userinput.png)

```
$ mvn spring-boot:build-image
```

This will create a container using buildpacks and will take a few minutes to build (3-5 minutes).

```
$ docker images
REPOSITORY              TAG              IMAGE ID       CREATED         SIZE
rest-service            0.0.1-SNAPSHOT   aef518db86d9   41 years ago    79.3MB
```
To run the container, execute:

```
$ docker run -p 8080:8080 docker.io/library/rest-service:0.0.1-SNAPSHOT
```
Now that the service is up, visit **localhost:8080/greeting** or `curl` where you should see:

![user input](images/userinput.png)

```
$ curl localhost:8080/greeting
{"id":1,"content":"Hello, World!"}
```

For option number two, you can build the container using a multi-stage Dockerfile.

Edit the `pom.xml` file and uncomment the `profile` section (lines 79-109) and comment the `build` section (lines 54-77):

```
<profiles>
   <profile>
      <id>native</id>
      <build>
         <plugins>
            <plugin>
               <groupId>org.graalvm.nativeimage</groupId>
               <artifactId>native-image-maven-plugin</artifactId>
               <version>21.0.0</version>
               <configuration>
                  <mainClass>com.example.restservice.RestServiceApplication</mainClass>
                  <imageName>graalvm-restservice</imageName>
                  <buildArgs>-Dspring.native.remove-yaml-support=true -Dspring.spel.ignore=true</buildArgs>
               </configuration>
               <executions>
                  <execution>
                     <goals>
                        <goal>native-image</goal>
                     </goals>
                     <phase>package</phase>
                  </execution>
               </executions>
            </plugin>
            <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
         </plugins>
     </build>
  </profile>
</profiles>
```

To build a container, execute this command:

![user input](images/userinput.png)

```
$ docker build . -t graalvm-restservice:multi
```

This will create a container using a multi-stage build process (feel free to examine the Dockerfile).  It will take a few minutes to build (3-5 minutes).

Check that the image is available:

![user input](images/userinput.png)

```
$ docker images
REPOSITORY              TAG           IMAGE ID           CREATED          SIZE
graalvm-restservice     multi        d4d1cdc7a246       9 minutes ago    81.2MB
```

Then you can run the container:

![user input](images/userinput.png)

```
$ docker run -p 8080:8080 graalvm-restservice:multi
```

Now that the service is up, visit **localhost:8080/greeting** or `curl` where you should see:

![user input](images/userinput.png)

```
$ curl localhost:8080/greeting
{"id":1,"content":"Hello, World!"}
```

### Summary

Congratulations! You successfully created a GraalVM native image executable and explored different options (buildpacks, native image Maven profile) for deploying the executable in a container. The `pom.xml` file is easily configurable and supports your different native image/container requirements and use cases. 
