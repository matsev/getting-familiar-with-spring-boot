# Getting Familiar with Spring Boot

Spring provides plenty of getting started guides at https://spring.io/guides/ . If there is some specific technology that
you would like to explore, or if you find yourself idling, find a guide of your choice and follow the instructions there.
There are several references to different parts of the reference documentation in this lab, however if you miss some 
information you can find it online at http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/  

## Create a project

- Go to the __Spring Initializr__ page: http://start.spring.io/
  - Fill in the text fields according to your preference.
  - Set `Type` to be `Maven Project`.
  - Set `Packaging` to `jar`
  - Set `Java Version` according to your preferences.
  - Set `Language` as `Java`.
  - Set `Spring Boot Version` to the latest, released, version, i.e. >= 1.1.6.RELEASE.
  - Click `Generate`.

- Unzip the generated file in a directory of your choice
  - Open the project in your favorite IDE.
  - Look at the generated `pom.xml` file.
    
**Question:** 
- What dependencies are used?
- How is version management handled?


There are three ways to execute your application:
- Open the `Application` class and locate the `main` method of your program. All Spring Boot program (including web 
applications) will use a main method as entry point. Consequently, you can debug your program by pointing your debugger 
to this method before you execute it.
- There is also a Maven plugin (`mvn spring-boot:run`) that can start your application. The Maven plugin is useful if 
you are working with a webapp and change static assets, it will it will be automatically be picked up without the need 
of rebooting the app.
- Lastly, if you build your application `mvn package` you will get an executable jar file that can be started from the 
command line `$ java -jar [your project]-0.0.1-SNAPSHOT.jar`.

Two more files have been generated:
- Open the `ApplicationTests` file to see what annotations are used to create a Spring Boot integration test. 
- Open the `application.properties` file. This is the default configuration file of your project. It is currently empty,
but Spring Boot offers plenty of [configuration properties](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#common-application-properties) 
(it should be noticed that it is also possible to use [external config files](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#boot-features-external-config),
but that is beyond the scope of this lab).


## Create a JSON Controller

First of all, we need to add a suitable _Starter POM_ to our project to get the required dependencies. A Starter POM is 
a convenient, opinionated, dependency declaration that allows you to easily extend your app with new features including 
all transitive dependencies. Take a moment to familiarize yourself with the Starter POM listed in the 
[reference docs](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#using-boot-starter-poms). As you may
have noticed, there is a one-to-one mapping between the available Starter POMs and the `Styles` listed at 
[Spring Initializr](http://start.spring.io/) page, and if you had selected one or more of them the corresponding Starter
 POM would have been added to the dependencies in the generated pom file.

**Question:** 
Which Starter POM do you need to add Tomcat, Spring MVC, FasterJackson and other useful dependencies for a web project? 
Hint, you only need one. 

Create a simple `Greeting` object:
```java
public class Greeting {

    private final long id;
    private final String content;
    
    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }
    
    public long getId() {
        return id;
    }
    
    public String getContent() {
        return content;
    }
}
```

Now, create a `GreetingController` that has a single `/greeting` resource that accepts an `content` request parameter. When 
requested, it returns a `Greeting` instance. In this example, the `id` can be generated directly in the controller, 
e.g. by using adding `AtomicLong` as a counter.

Hints, use the [@RestController](http://docs.spring.io/spring-framework/docs/4.0.x/spring-framework-reference/html/mvc.html#mvc-ann-restcontroller),
[@RequestMapping](http://docs.spring.io/spring-framework/docs/4.0.x/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping) 
and [@RequestParam](http://docs.spring.io/spring-framework/docs/4.0.x/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html) 
annotations.

- Verify that the response body defaults to JSON.
- Verify that the request parameter is picked up correctly.
- _Optional:_ create an integration test for the resource, see http://www.jayway.com/2014/07/04/integration-testing-a-spring-boot-application/
for implementation details and explanation.


**Question:**
How can we make the `content` parameter optional (and default to `Hello World`)?  

_Optional:_ 
What needs to be changed in order to get an XML payload?  


See the _Rest Service guide_ for solution: https://spring.io/guides/gs/rest-service/


## Actuator

Spring Boot has some [production ready features](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#production-ready-endpoints) 
that can easily by added by adding a dependency.

- Add the `spring-boot-starter-actuator` to your project dependencies and restart your application. 

**Question:** 
- Take a closer look at the console log. If you have been working with log analytics, you will appreciate that Spring 
Boot has already provided a nice format for you, but it can easily be customized to your needs. See the [reference docs](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#boot-features-logging)
for details.
- Which endpoints are available? Hint: look for `EndpointHandlerMapping` in the log.
- What is the purpose of the following endpoints: 
  - http://localhost:8080/env
  - http://localhost:8080/configprops

_Optional:_
Customize which endpoints that should be enabled, see the [reference docs](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#production-ready-customizing-endpoints).


_Optional:_
Did you also notice in the log that a series of MBeans where created and exported using the `EndpointMBeanExporter`?
- Start Visual VM (or jconsole), connect to your app,and see what is provided by the MBeans: 
```sh
$ jvisualvm
```


### Metrics

Make a few requests to your http://localhost:8080/greeting endpoint and then take a look at the http://localhost:8080/metrics 
endpoint. 

**Question:**
- What is the difference between the `counter.status.200.greeting: [nbr]` and `gauge.response.greeting: [nbr]`?
Hint, make another request to http://localhost:8080/greeting and reload the `/metrics` endpoint.


_Optional:_ 
Create your own custom metrics by following the instructions in the [reference docs](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#production-ready-recording-metrics).
Implement either a [GaugeService](http://docs.spring.io/spring-boot/docs/1.1.x/api/org/springframework/boot/actuate/metrics/GaugeService.html) 
or create an instance of the [DefaultGaugeService](http://docs.spring.io/spring-boot/docs/1.1.x/api/org/springframework/boot/actuate/metrics/writer/DefaultGaugeService.html)
that can be used to measure snapshot values (e.g. feed it with a random value if you do not come up with anything useful).
Alternatively, implement a [CounterService](http://docs.spring.io/spring-boot/docs/1.1.x/api/org/springframework/boot/actuate/metrics/CounterService.html)
or instantiate [DefaultCounterService](http://docs.spring.io/spring-boot/docs/1.1.x/api/org/springframework/boot/actuate/metrics/writer/DefaultCounterService.html)
and start counting (implement your own trigger(s) to increment and / or decrement the counter, e.g. by creating a 
[Scheduled Task](https://spring.io/guides/gs/scheduling-tasks/)).
- Revisit http://localhost:8080/metrics and observe your metric(s).


### Port configuration

A common requirement is that the information provided by the actuator should not be accessible by the user. One solution 
is to configure different ports for the application and the actuator endpoints, and then a firewall in front of the 
application to protect the application.
 
- Open `application.properties`
- Add the following lines:
```.properties
server.port=8081
management.port=8082
```
- Make requests to http://localhost:8081/greeting and http://localhost:8082/metrics 



## Security

- Add the `spring-boot-starter-security` to your project dependencies, restart your application and try to access the 
`/greeting` resource. By default, you have now enabled basic auth on your application. The default username is `user`
and you will find a generated password in the console log.
- Find the password and make a request to the `/greeting` resource. 

You can also provide a username and password of your choice, open the `application.properties` file and add the following
properties:
```.properties
security.user.name=[your preferred user name]
security.user.password=[your secret password]
```

Reboot the application and login again using the new credentials.

_Optional:_ 
If you added the actuator feature described above, you should be aware of that enabling security also affects the 
actuator endpoints. 
- Try accessing http://localhost:8080/metrics (or whatever port number you have configured)
- You can disable the security checks for the actuator endpoints by configuring `management.security.enabled=false` (I 
only recommend this alternative if you have configured different ports for the application and the actuator endpoints) 
- Reboot your app.

_Optional:_ 
It is not uncommon that you would like another authentication mechanism than basic auth. The [Securing a Web Application](https://spring.io/guides/gs/securing-web/)
guide show you how you can extend the `WebSecurityConfigurerAdapter` to use form based login. The same example also shows 
you how the GlobalAuthenticationConfigurerAdapter` can be extended to support in memory authentication, giving you an idea 
how other authentication mechanism can be implemented. Another security example is the [Authenticating a User with LDAP](https://spring.io/guides/gs/authenticating-ldap/)
guide that shows you how Spring Security can interact with LDAP.                    
If you have enabled the actuator, it is possible to 
[configure a specific role](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#production-ready-sensitive-endpoints) 
so that only users who have the specific role are authorized to access the actuator endpoints.



## Persistence

Add the `spring-boot-starter-data-jpa` Starter POM to your dependencies. Additionally, add HSQLDB (or H2 or Derby which 
are two other in-memory databases that are supported) that will be used for testing:
```xml
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <!-- no version required -->
</dependency>
```

Update the implementation of the `Greeting` class to 

```java
@Entity
public class Greeting {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private long id;
    private String content;
    
    /**
     * Default constructor required by JPA
     */
    protected Greeting() {}
    
    public Greeting(String content) {
        this.content = content;
    }
    
    // Getters, setters, equals, hashCode, etc
}
```

Implement a `GreetingRepository` by extending Spring Data's [PagingAndSortingRepository](http://docs.spring.io/spring-data/data-commons/docs/1.8.x/api/org/springframework/data/repository/PagingndSortingRepository.html)
or [CrudRepository](http://docs.spring.io/spring-data/data-commons/docs/1.8.x/api/org/springframework/data/repository/CrudRepository.html)
```java
public interface GreetingRepository extends PagingAndSortingRepository<Greeting, Long> {
}
```

- What methods are provided by the interfaces?

_Optional:_ Add a method that returns a list of `Greeting`s matching exact content:
```java
List<Greeting> findByContent(String content);
```
Behind the scenes, Spring Data will automatically generate an implementation of any `findBy*` method if the corresponding
entity has a matching accessor method.
 

_Optional:_ Add a method that returns a list of `Greeting`s matching string contains of content:
```java
@Query("SELECT g FROM Greeting g where g.content like %?1%")
List<Greeting> findByContentContains(String content);
```
Create an integration test to verify that your query works:
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
public class GreetingRepositoryTest {

    @Autowired
    GreetingRepository repo;

    @Before
    public void setUp() {
        repo.deleteAll();
    }

    @Test
    public void findsByContentSubString() {
        Greeting expected = repo.save(new Greeting("This is a string"));

        List<Greeting> actual = repo.findByContentContains("is");
        assertThat(actual, contains(expected));
    }
}
```
More information about `@Query` can be found in the [Spring Data JPA reference docs](http://docs.spring.io/spring-data/data-jpa/docs/current/reference/html/jpa.repositories.html#jpa.query-methods.at-query)



### Add a MySQL database for production

Install MySQL:
- OS X: `$ brew install mysql`
- Linux: [apt-get or yum](http://dev.mysql.comdoc/refman/5.6/en/linux-installation-native.html)
- Windows: download [installer](http://dev.mysql.com/downloads/installer/)

Verify that the installation works:
```
$ mysql --version
mysql  Ver 14.14 Distrib 5.6.20, for osx10.9 (x86_64) using  EditLine wrapper
```
Start the MySQL server:
```
$ mysql.server start
Sarting MySQL
 SUCCESS!
```

Add the following dependency to the pom:
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <!-- no version required -->
</dependency>
```

**Question:**
Now that we have two databases, which one is used?
Hint, restart your app and check the log.

By changing some properties, we can configure Spring Boot to connect to MySQL. Open the `application.properties` and add
the following:

```.properties
spring.datasource.platform=mysql
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=
spring.datasource.password=
spring.datasource.driverClassName=com.mysql.jdbc.Driver
```

Add the MySQL connector to your list of dependencies:

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <!-- no version required -->
</dependency>
```

If you restart your app, you will find that it connects to MySQL, but an exception is thrown:

```sh
MySQLSyntaxErrorException: Table 'test.greeting' doesn't exist
```
To overcome this issue, Spring Boot provides [database initialization](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#howto-intialize-a-database-using-spring-jdbc)
By convention, a file named `schema.sql` that is located in the classpath root will be executed. By adding the persistence
platform to the file name, e.g. `schema-${platform}.sql` the file will only be executed by for that platform. 

- Create a new file called `schema-mysql.sql` in the project `/resource` folder.
- Open the file and add the following script: 
```sql
CREATE TABLE IF NOT EXISTS Greeting (
  id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  content VARCHAR (255)
);
```
- Restart the app.



### Health Check

The actuator also provide a health check endpoint, intended to be used by load balancers and similar tools.
- Start the application using MySQL for persistence.
- Goto http://localhost:8080/health (or to whatever port you have configured the `management.port` to).
- Stop MySQL
```sh
$ mysql.server stop
Shutting down MySQL
.... SUCCESS!
```
- Reload http://localhost:8080/health



### Profiles

Now we have two databases, but we currently only use MySQL. What we would like is two create two [profiles](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#boot-features-profiles),
one `test` profile that uses HSQLDB, and one `prod` profile that uses MySQL. Since it is only properties that differs, we 
can create an `application-${profile}.properties` specific file for each profile, see the [reference docs](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#boot-features-external-config-profile-specific-properties).

- Create a new `application-prod.properties` file in your `/resources` folder. 
- Cut and paste the following properties from the `application.properties` file to the new `application-prod.properties`
file:
```.properties
spring.datasource.platform=mysql
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=
spring.datasource.password=
spring.datasource.driverCassName=com.mysql.jdbc.Driver
```
- _Optional:_ Create a new `application-test.properties` file in your `/resources` folder that is left empty

**Question:**
Which database is used when you restart the application?

IMHO, it is better to make the production configuration the default configuration, and add any overrides in test or stage
environment, because the consequences of making a mistake is much greater.

Add the following line to the `application.properties` and restart the application:
```.properties
spring.profiles.default=prod
```
This configures the application to start using the `default` profile, however there is a [bug](https://github.com/spring-projects/spring-boot/issues/1219)
(Spring Boot version 1.1.2.RELEASE) that prevents Spring Boot from picking up the correct settings from the 
`application-prod.properties` file.

So far so good, but how about overriding the profile to enable the `test` profile? The answer depends on what you would 
like to do:
- If you would like to add the `spring.profiles.active=test` as an argument to the `main()` method.
- If you have built the app, you can add it as an command line argument, e.g.
```sh
$ java -jar -Dspring.profiles.active=test your-app-0.0.1-SNAPSHOT.jar
```

- If you would like to use a specific profile for a test, you can use the [@ActiveProfile](http://docs.spring.io/spring-framework/docs/4.0.x/javadoc-api/org/springframework/test/context/ActiveProfiles.html)
annotation.
```java
@ActiveProfiles("test")
```


_Optional:_ 
I do not recommend that JPA is used for SQL schema generation as it makes migrations cumbersome. Using the `schema.sql` 
approach is better, but for production I recommend using a tool such as Flyway or Liquibase from day one, see the 
[reference guide](http://docs.spring.io/spring-boot/docs/1.1.x/reference/htmlsingle/#howto-use-a-higher-level-database-migration-tool) 
Implement a migration based on the [Flyway sample](http://github.com/spring-projects/spring-boot/tree/v1.1.5.RELEASE/spring-boot-samples/spring-boot-sample-flyway) 
or the [Liquibase sample](http://github.com/spring-projects/spring-boot/tree/v1.1.5.RELEASE/spring-boot-samples/spring-boot-sample-liquibase). 


_Optional:_ 
- `@Autowire` the `GreetingRepository` in the `GreetingController`. 
- Create and map the following resources to the repo:
  - Return all `Greeting`s from the `/greetings` resource.
  - Post a new `Greeting` to the `/greetings` resource.  
  - Fetch a single `Greeting` from the `/greetings/{id}` resource.
  
Hints: use the `@PathVariable` and the `@RequestMapping` annotations.
