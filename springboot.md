

## 自动配置

**pom.xml**

- spring-boot-dependency : 核心依赖在父工程中
- 引入依赖，不需要指定版本，依赖仓库

**启动器**

- springboot的启动场景；比如spring-boot-starter-web,自动导入web环境所有的依赖。
- 我们需要什么功能，就只需要找到对应的启动器，starter

> 参考

https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot



**主程序**

```java
@SpringBootApplication //注解

 @SpringBootConfiguration //springboot的配置
   @Configuration //spring配置


 @EnableAutoConfiguration  //自动配置
   @AutoConfigurationPackage   //自动配置包
     @Import(AutoConfigurationPackages.Registrar.class) // 自动配置 包注册
   @Import(AutoConfigurationImportSelector.class)       //自动配置导入选择

//获取所有候选配置
List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
```



```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),      getBeanClassLoader());
    
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                    + "are using a custom packaging, make sure that file is correct.");
    
    return configurations;
}

protected Class<?> getSpringFactoriesLoaderFactoryClass() {
    return EnableAutoConfiguration.class;
}

protected ClassLoader getBeanClassLoader() {
    return this.beanClassLoader;
}
```



结论：springboot所有自动配置都是在启动的时候扫描并加载，META-INF/Spring.factories所有的自动配置类都在这个里面，但是不一定生效，需要判断条件是否成立，只要导入了对应的starter，就有对应的启动器了，对应的自动装配就会生效，然后配置成功。

1. springboot启动时，从类路径下META-INF/Spring.factories获取指定的值
2. 将这些自动配置的类导入容器，并生效
3. 以前我们需要配置的，现在springboot帮我们做了
4. 整合javaEE，解决方案和自动配置的东西都在spring-boot-autoconfigure-2.2.0RELEASE.jar这个包下
5. 它会吧所有需要导入的组件，以类名方式返回，这些组件就会添加到容器
6. 配置文件中都是XXXAutoConfiguration的文件（Bean），并自动配置@Configuration。
7. 有了自动配置类，免去手动编写配置文件。
8. 自动配置组件的时候，会从properties类中获取某些属性，我们只需要在配置文件中指定这些属性的值即可。XXXProperties封装配置文件的相关属性
9. debug=true查看哪些自动配置类生效



## Spring Security 整合

https://docs.spring.io/spring-security/site/docs/current/reference/html5/

依赖管理

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```



官网实例代码：

```JAVA
@EnableWebSecurity
public class MultiHttpSecurityConfig {
    @Bean                                                             
    public UserDetailsService userDetailsService() throws Exception {
        // ensure the passwords are encoded properly
        UserBuilder users = User.withDefaultPasswordEncoder();
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(users.username("user").password("password").roles("USER").build());
        manager.createUser(users.username("admin").password("password").roles("USER","ADMIN").build());
        return manager;
    }

    @Configuration
    @Order(1)                                                        
    public static class ApiWebSecurityConfigurationAdapter extends WebSecurityConfigurerAdapter {
        protected void configure(HttpSecurity http) throws Exception {
            http
                .antMatcher("/api/**")                               
                .authorizeRequests(authorize -> authorize
                    .anyRequest().hasRole("ADMIN")
                )
                .httpBasic(withDefaults());
        }
    }

    @Configuration                                                   
    public static class FormLoginWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http
                .authorizeRequests(authorize -> authorize
                    .anyRequest().authenticated()
                )
                .formLogin(withDefaults());
        }
    }
}
```


## Shiro整合

At the highest conceptual level, Shiro’s architecture has 3 primary concepts: the `Subject`, `SecurityManager` and `Realms`. The following diagram is a high-level overview of how these components interact, and we’ll cover each concept below:


- **Subject**: As we’ve mentioned in our [Tutorial](http://shiro.apache.org/tutorial.html), the `Subject` is essentially a security specific ‘view’ of the the currently executing user. Whereas the word ‘User’ often implies a human being, a `Subject` can be a person, but it could also represent a 3rd-party service, daemon account, cron job, or anything similar - basically anything that is currently interacting with the software.

  `Subject` instances are all bound to (and require) a `SecurityManager`. When you interact with a `Subject`, those interactions translate to subject-specific interactions with the `SecurityManager`.

- **SecurityManager**: The `SecurityManager` is the heart of Shiro’s architecture and acts as a sort of ’umbrella’ object that coordinates its internal security components that together form an object graph. However, once the SecurityManager and its internal object graph is configured for an application, it is usually left alone and application developers spend almost all of their time with the `Subject` API.

  We will talk about the `SecurityManager` in detail later on, but it is important to realize that when you interact with a `Subject`, it is really the `SecurityManager` behind the scenes that does all the heavy lifting for any `Subject` security operation. This is reflected in the basic flow diagram above.

- **Realms**: Realms act as the ‘bridge’ or ‘connector’ between Shiro and your application’s security data. When it comes time to actually interact with security-related data like user accounts to perform authentication (login) and authorization (access control), Shiro looks up many of these things from one or more Realms configured for an application.

  In this sense a Realm is essentially a security-specific [DAO](https://en.wikipedia.org/wiki/Data_access_object): it encapsulates connection details for data sources and makes the associated data available to Shiro as needed. When configuring Shiro, you must specify at least one Realm to use for authentication and/or authorization. The `SecurityManager` may be configured with multiple Realms, but at least one is required.

  Shiro provides out-of-the-box Realms to connect to a number of security data sources (aka directories) such as LDAP, relational databases (JDBC), text configuration sources like INI and properties files, and more. You can plug-in your own Realm implementations to represent custom data sources if the default Realms do not meet your needs.

  Like other internal components, the Shiro `SecurityManager` manages how Realms are used to acquire security and identity data to be represented as `Subject` instances.

官网10分钟实例

```JAVA
public class Quickstart {

    private static final transient Logger log = LoggerFactory.getLogger(Quickstart.class);


    public static void main(String[] args) {

        // The easiest way to create a Shiro SecurityManager with configured
        // realms, users, roles and permissions is to use the simple INI config.
        // We'll do that by using a factory that can ingest a .ini file and
        // return a SecurityManager instance:

        // Use the shiro.ini file at the root of the classpath
        // (file: and url: prefixes load from files and urls respectively):
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager = factory.getInstance();

        // for this simple example quickstart, make the SecurityManager
        // accessible as a JVM singleton.  Most applications wouldn't do this
        // and instead rely on their container configuration or web.xml for
        // webapps.  That is outside the scope of this simple quickstart, so
        // we'll just do the bare minimum so you can continue to get a feel
        // for things.
        SecurityUtils.setSecurityManager(securityManager);

        // Now that a simple Shiro environment is set up, let's see what you can do:

        // get the currently executing user:
        Subject currentUser = SecurityUtils.getSubject();

        // Do some stuff with a Session (no need for a web or EJB container!!!)
        Session session = currentUser.getSession();
        session.setAttribute("someKey", "aValue");
        String value = (String) session.getAttribute("someKey");
        if (value.equals("aValue")) {
            log.info("Retrieved the correct value! [" + value + "]");
        }

        // let's login the current user so we can check against roles and permissions:
        if (!currentUser.isAuthenticated()) {
            UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
            token.setRememberMe(true);
            try {
                currentUser.login(token);
            } catch (UnknownAccountException uae) {
                log.info("There is no user with username of " + token.getPrincipal());
            } catch (IncorrectCredentialsException ice) {
                log.info("Password for account " + token.getPrincipal() + " was incorrect!");
            } catch (LockedAccountException lae) {
                log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                        "Please contact your administrator to unlock it.");
            }
            // ... catch more exceptions here (maybe custom ones specific to your application?
            catch (AuthenticationException ae) {
                //unexpected condition?  error?
            }
        }

        //say who they are:
        //print their identifying principal (in this case, a username):
        log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");

        //test a role:
        if (currentUser.hasRole("schwartz")) {
            log.info("May the Schwartz be with you!");
        } else {
            log.info("Hello, mere mortal.");
        }

        //test a typed permission (not instance-level)
        if (currentUser.isPermitted("lightsaber:wield")) {
            log.info("You may use a lightsaber ring.  Use it wisely.");
        } else {
            log.info("Sorry, lightsaber rings are for schwartz masters only.");
        }

        //a (very powerful) Instance Level permission:
        if (currentUser.isPermitted("winnebago:drive:eagle5")) {
            log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
                    "Here are the keys - have fun!");
        } else {
            log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
        }

        //all done - log out!
        currentUser.logout();

        System.exit(0);
    }
}

```

1. Ensure you have JDK 1.8+ and Maven 3.0.3+ installed.

2. Download the lastest “Source Code Distribution” from the [Download](http://shiro.apache.org/download.html) page. In this example, we’re using the 1.7.1 release distribution.

3. Unzip the source package:

   ```
   $ unzip shiro-root-1.7.1-source-release.zip
   ```

4. Enter the quickstart directory:

   ```
   $ cd shiro-root-1.7.1/samples/quickstart
   ```

5. Run the QuickStart:

   ```
   $ mvn compile exec:java
   ```







## 任务

### 异步任务

```java
//开启异步注解功能
@EnableAsync
class Application{}

//告诉spring这是一个异步方法
@Async
XXXService
```



### 定时任务

```java
//任务调度器
TaskSchedule

//任务执行者
TaskExecute
    
//开启定时注解功能
@EnableScheduling
class Application{}    

//什么时候执行
//cron
//秒 分钟 小时 日 月 星期
@Scheduled(cron="0 * * * * 0-7")
XXXService
```

