# Appsody Stack with Spring® Boot 2, Swagger, JPA

This Appsody Stack, namely `springboot-swagger-jpa`, is inspired by Appsody's Spring Boot 2 Stack, [incubator/java-spring-boot2](https://github.com/appsody/stacks/tree/master/), by Appsodizing [this sample project](https://github.com/brightzheng100/springboot-swagger-jpa-stack).


## Features / Highlights

The Stack, like what Spring Boot 2 Stack does, supports the development of [Spring Boot 2](https://spring.io/projects/spring-boot) applications using OpenJDK 8+ and OpenJ9 from [AdoptOpenJDK](https://adoptopenjdk.net/).

Moreover, it provides a series of integrated capabilities and plugins to streamline and simplify the development of web apps and RESTful APIs.

**The major features:**

- Fully managed by Maven;
- Integrated [Swagger](https://github.com/swagger-api/swagger-ui) so make exposing and documenting RESTful APIs fun;
- Integrated JPA because it's standard and makes talking to RDBMS easy;
- Integrated [H2](https://www.h2database.com) as an embedded database for local development and testing; while you can easily talk to external DB, e.g. MySQL, in production;
- Integrated [Flyway](https://flywaydb.org/) to make database upgrades / patches as code;
- Integrated [micrometer](http://micrometer.io/) to expose Prometheus-style metrics for app monitoring;
- Integrated Spring Boot Test Framework with Junit 4, [Rest Assured](https://github.com/rest-assured/rest-assured) for testing;
- Integrated with [Hazelcast](https://hazelcast.org/) which can be activated by adding a Spring profile named "hazelcast";
- etc.

> Note: Although Maven is provided by the Appsody stack container, allowing you to build, test, and debug your Java application without installing Maven locally, we still recommend installing Maven locally for the best development experience.


## Templates

Templates are used to initialize your application project to quickly start development.

For now, there is only one template named `default`, which is also the default template of the Stack.

Once initialized, by syntax of `appsody init [<REPO>/]<STACK> [<TEMPLATE>]` , the template will help you scaffold a runnable application with following file structure:

```
$ tree .
.
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── app
    │   │       ├── Application.java
    │   │       ├── config
    │   │       │   └── SwaggerConfig.java
    │   │       ├── controller
    │   │       │   ├── DefaultController.java
    │   │       │   ├── GreetingController.java
    │   │       │   └── StudentController.java
    │   │       ├── model
    │   │       │   ├── Greeting.java
    │   │       │   └── Student.java
    │   │       └── repository
    │   │           └── StudentRepository.java
    │   └── resources
    │       ├── application-default.yml
    │       ├── application-hazelcast.yml
    │       ├── application-prod.yml
    │       ├── application.yml
    │       ├── bootstrap.yml
    │       └── db
    │           └── migration
    │               ├── V1.0.0__initial.sql
    │               ├── V1.0.1__add_person.sql
    │               └── V1.0.2__alter_student.sql
    └── test
        └── java
            └── app
                └── ApplicationTest.java
```

Where:

- It provides a `pom.xml` file that references the parent POM defined by the Stack;
- Some end-to-end sample code, including JPA-integrated RESTful APIs, is included for developers to reference to;
- The practices of Spring Profiles are embedded;
- A popular way of DB schema upgrades / patches by using `Flyway` is also demonstrated;
- Some test cases are also provided for developers to start with;
- And you may find more.


## Getting Started

### Package Stack

Appsody supports building the Stack from source code, so let's start from scratch.

```sh
$ git clone https://github.com/brightzheng100/springboot-swagger-jpa-appsody-stack.git springboot-swagger-jpa
$ cd springboot-swagger-jpa

$ appsody stack package

$ appsody repo list
NAME      	URL
*dev.local	file:///Users/brightzheng/.appsody/stacks/dev.local/dev.local-index.yaml

$ appsody list dev.local
REPO     	ID                    	VERSION  	TEMPLATES	DESCRIPTION
dev.local	springboot-swagger-jpa	0.1.1    	*default 	A Stack managed by Maven, with Spring Boot, Swagger, JPA
```

Now the stack is ready to use, at `dev.local/springboot-swagger-jpa`.


### Init Application

```
$ cd .. && mkdir my-project && cd my-project

$ appsody init dev.local/springboot-swagger-jpa

$ ls -AL
.appsody-config.yaml .gitignore           .vscode              pom.xml              src
```

This will initialize a Spring Boot 2 project, managed by Maven and itegrated with `Swagger`, `JPA`, `H2`, `Flyway` etc., using the default template.


### Try It Through Using `appsody` Commands

**1. `appsody run`**: To run the application

```sh
$ appsody run
```

You should be able to try out the RESTful APIs exposed and visualized by `Swagger` at:
- http://localhost:8080/swagger/index.html

And Actuator endpoints:
- The health with DB info: http://localhost:8080/actuator/health
- The Flyway info: http://localhost:8080/actuator/flyway
- The Prometheus metrics: http://localhost:8080/actuator/prometheus
- The H2 Console: http://localhost:8080/h2-console

Please refer to [this sample application](https://github.com/brightzheng100/springboot-swagger-jpa-stack) for more details.

**2. `appsody debug`**: To start debugging the application. The debugger will listen at port 5005.

**3. `appsody test`**: To run test cases for the application.

**4. `appsody build`**: To build the Docker image and generate a Kubernetes manifest file.

```sh
$ appsody build -t registry:5000/my-project:latest --push
```

This will build a Docker image tagged as `registry:5000/my-project:latest` and push it to Docker registry at `registry:5000`.

Do change it accordingly for your env.

Meanwhile, it will create the `app-deploy.yaml` file which is the Kubernetes manifest.

If you want to activate `prod` Spring profile with a specified external MySQL URL, update the `app-deploy.yaml` with the `env` part:

```sh
apiVersion: appsody.dev/v1beta1
kind: AppsodyApplication
...
spec:
  ...
  env:
  - name: SPRING_PROFILES_ACTIVE
    value: prod
  - name: SPRING_DATASOURCE_URL
    value: jdbc:mysql://192.168.1.148:3306/testdb   # point it to an empty `testdb` at `192.168.1.148`
  ...
```

**5. `appsody deploy`**: To deploy it to Kubernetes.

```sh
$ appsody deploy -t registry:5000/my-project:latest
```

> Note: by default, `appsody deploy` will include the scope of `appsody build` at least as of current Appsody releases (e.g. v0.5.x). Adding flag `--no-build` can ignore the build process.

**6. `kubectl get`**: To check it out from Kubernetes.

```sh
$ kubectl get appsodyapplication,svc,pod
NAME                                        IMAGE                            EXPOSED   RECONCILED   AGE
appsodyapplication.appsody.dev/my-project   registry:5000/my-project:0.1.0   true      True         3m23s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    88m
service/my-project   ClusterIP   10.106.100.39   <none>        8080/TCP   3m23s

NAME                                   READY   STATUS    RESTARTS   AGE
pod/appsody-operator-595d74b67-l8qhk   1/1     Running   0          85m
pod/my-project-85b768977d-4dgr2        1/1     Running   0          3m23s
```

**7. Access Application**: By default, it's exposed by `ClusterIP` service type.

```sh
$ kubectl port-forward service/my-project 8080:8080
```

Now open your browser and navigate to http://localhost:8080/.

Yep, you should be redirected to the Swagger UI, with the sample APIs listed there:

![swagger-ui](https://raw.githubusercontent.com/brightzheng100/springboot-swagger-jpa-stack/master/misc/screenshot-swagger.png "Swagger UI")

So it works end to end. Enjoy!


## Advanced Topics

### Enable Hazelcast

The project generated by `appsody init dev.local/springboot-swagger-jpa` already has `Hazelcast` client integrated.

Hazelcast can be activated by enabling one extra profile named `hazelcast`.

Please refer to [here](https://github.com/brightzheng100/springboot-swagger-jpa-stack#run-it-with-hazelcast-integrated) for the detailed steps:

1. Setup a Hazelcast cluster by Docker Compose

2. Run it with extra Spring profile: `SPRING_PROFILES_ACTIVE=default,hazelcast mvn clean spring-boot:run`

3. Optionally, setup the Hazelcast management center to manage and visualize Hazelcast clusters.


If you want to run it by using the same `appsody run` experience, instead of `SPRING_PROFILES_ACTIVE=default,hazelcast mvn clean spring-boot:run`, follow below steps:

1. Setup a Hazelcast cluster by Docker Compose -- which is the same;

2. Build a `hazelcast.yaml` and put it under `src/main/resources/` folder with the IPs we got:

```sh
IP_ADDR="192.168.56.1"      # CHANGE ME TO YOUR MACHINE'S IP ADDRESS, e.g. 192.168.56.1
cat > src/main/resources/hazelcast.yaml <<EOF
hazelcast-client:
  cluster-name: dev
  network:
    cluster-members:
    - $IP_ADDR:5701
    - $IP_ADDR:5702
EOF
``` 

3. Run `appsody run` with extra parameters to enable right Spring profiles:

```sh
appsody run --docker-options "-e spring_profiles_active=default,hazelcast"
```


## License

This Stack is licensed under the [Apache 2.0](./image/LICENSE) license.
