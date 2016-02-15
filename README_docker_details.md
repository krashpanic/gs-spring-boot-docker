#Sample Spring boot Docker :


* clone from github

```
git clone https://github.com/spring-guides/gs-spring-boot-docker.git
```

* `cd initial/src/main/java/hello`

* sample `Application.java` file

```
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.bind.RelaxedPropertyResolver;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class Application {

	@RequestMapping("/")
	public String home() {
		return "Hello Docker World";
	}


	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```


* test locally :

```
mvn package && java -jar target/gs-spring-boot-docker-0.1.0.jar
```

and go to `localhost:8080` to see your "Hello Docker World" message

###Build a Docker Image with Maven

The configuration specifies 3 things:

1. The image name (or tag), which will end up here as `springio/gs-spring-boot-docker`

2. The directory in which to find the Dockerfile

3. The resources (files) to copy from the target directory to the docker build (alongside the Dockerfile) - we only need the jar file in this example

`pom.xml` snippet :

```
...
<properties>
      <docker.image.prefix>springio</docker.image.prefix>
</properties>
<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>0.2.3</version>
            <configuration>
                <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                <dockerDirectory>src/main/docker</dockerDirectory>
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
            </configuration>
        </plugin>
    </plugins>
</build>
...
```

* Now on to the `Dockerfile` :


```
FROM java:8
VOLUME /tmp
ADD gs-spring-boot-docker-0.1.0.jar app.jar
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```



* Before proceeding with the following steps (which use [Docker’s tools](https://docs.docker.com/v1.8/installation/mac/)), make sure `docker` is properly running on your system by typing `docker ps`. If you get an error message, something may not be set up correctly

```
docker-machine create --driver virtualbox spring-boot
docker-machine env spring-boot`
export DOCKER_TLS_VERIFY="1"`
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/joe/.docker/machine/machines/spring-boot"
export DOCKER_MACHINE_NAME="spring-boot"
eval "$(docker-machine env spring-boot)”
```

* Maven steps :
```
mvn package && java -jar target/gs-spring-boot-docker-0.1.0.jar
mvn package docker:build
```

* verify docker your docker image is running locally :

```
docker images

$ docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
krashpanic/gs-spring-boot-docker   latest              c523bba54501        48 minutes ago      842.5 MB
gs-spring-boot-docker              latest              85b0ea154d0f        53 minutes ago      842.5 MB
springio/gs-spring-boot-docker     latest              700aee5bb66a        About an hour ago   842.5 MB
java                               8                   4cd29d33e3f4        13 days ago         817.5 MB
```
 
* test locally :

```
docker run -p 8080:8080 -t springio/gs-spring-boot-docker
```

* push image to repo :


```
docker push krashpanic/gs-spring-boot-docker
The push refers to a repository [docker.io/krashpanic/gs-spring-boot-docker] (len: 1)
c523bba54501: Image successfully pushed
700aee5bb66a: Image successfully pushed
44c250bfb43c: Image successfully pushed
2ccf22dc8ea8: Image successfully pushed
57c5723fbe03: Image successfully pushed
4cd29d33e3f4: Image successfully pushed
623f68952214: Image successfully pushed
35c51c011946: Image already exists
9798a430a6cc: Image already exists
b7e87190a39a: Image already exists
7203bee160e2: Image already exists
49e5d222aba6: Image successfully pushed
dc24080994f1: Image successfully pushed
2cd6a1879c96: Image successfully pushed
3f8d2e13b904: Image successfully pushed
58016a5acc80: Image already exists
e2a4fb18da48: Image successfully pushed
latest: digest: sha256:f1410addb199278805a48b779abfa969ceb35ba98adb08911d1fc9d7083f5b11 size: 32852
```

###Reference :

https://spring.io/guides/gs/spring-boot-docker/

