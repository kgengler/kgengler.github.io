---
layout: post
comments: true
title: "Basic Spring Web Application With No XML"
date: 2017-02-07 07:12:00 -0600
categories: java spring
---

Something I always end up needing to reference is building a new application
from scratch. It seems like it's just not something I ever need to do that
often. So here is a guide on how to get a very basic "Hello World!" Spring 4
application with a Thymeleaf view resolver with absolutely no XML.

## The Source

First things first, I put the source used in the examples together in a Github
repository. 

[https://github.com/kgengler/basic-spring-web-application](https://github.com/kgengler/basic-spring-web-application)

If you just want to start with that, all you need to do is clone the
repository and build with your preferred tool. I've included configurations for
both [Maven](https://maven.apache.org/), and [Gradle](https://gradle.org/).

## The Container

It seems like every guide I find on the Internet nowadays for spring ALWAYS uses
Spring Boot. While Spring Boot is a great project, some of use still deploy to
application containers like [Wildfly](http://wildfly.org/) or 
[tc Server](http://www.vmware.com/products/pivotal-tcserver.html). I still like
to use [Tomcat](http://tomcat.apache.org/) for basic things. Basically, pick
your poison and make sure you have it set up and running according to your given
application server's installation instructions.

## The Build Tool

Gradle seems to be really popular now, but I seem to always reach for Maven
still. Either way, I've detailed both below as it doesn't really matter.

### Maven

[Maven](https://maven.apache.org/) uses an XML configuration to set up the
project. In your project directory, create a file `pom.xml`. Put the following
configuration in your `pom.xml` file.

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.kylegengler</groupId>
    <artifactId>BasicWebApp</artifactId>
    <version>1.0.0</version>
    
    <!-- Tell maven to build this as a war -->
    <packaging>war</packaging>

    <build>

        <!-- Set the final artifact name of the project -->
        <finalName>${project.artifactId}</finalName>
        <plugins>

            <!-- Allow the build to pass without a web.xml -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>

            <!-- Compile for Java 8 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <!-- Project Dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring4</artifactId>
            <version>3.0.3.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

To build the project you will run

```
mvn clean package
```

The resulting file will be the `target/BasicWebApp.war`

### Gradle

[Gradle](https://gradle.org/) uses Groovy to write its configuration. In your
project root directory, create a file `build.gradle`. Use the below
configuration

```
// Build a war
apply plugin: 'war'

group = 'com.kylegengler'
version = '1.0.0'

description = "A Basic Spring Application"

// Build for Java 8
sourceCompatibility = 1.8
targetCompatibility = 1.8

// Set the final name for the artifact
war {
    archiveName 'BasicWebApp.war'
}

repositories {
     mavenCentral()
}

// Declare the project dependencies
dependencies {
    compile group: 'org.springframework', name: 'spring-webmvc', version:'4.3.6.RELEASE'
    compile group: 'org.thymeleaf', name: 'thymeleaf-spring4', version:'3.0.3.RELEASE'
}
```

To build the application you will run:

```
gradle clean war
```

The resulting file will be available at `build/libs/BasicWebApp.war`.

## The Web App Configuration

Traditionally Java applications will use a `web.xml` for their configuration.
Now with servlet 3.0+ environments, we can write out the configuration in Java.

```
package com.kylegengler.basic;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

/**
 * This class configures the web application. This replaces the traditional
 * web.xml file in servlet 3.0+ environments.
 * 
 * @author kyle
 */
public class WebApplicationConfiguration extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * Specify the root configuration classes. This is used for the business
     * logic, and persistence classes. Since this application is just a basic
     * "Hello World!" application, we'll be omitting it here.
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    /**
     * Configuration classes for the servlet environment. This would include the
     * Controllers and view resolvers.
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { ServletConfiguration.class };
    }

    /**
     * Specify the url mappings that will be handled by the servlet.
     */
    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

The magic here is that by extending the
`AbstractAnnotationConfigDispatcherServletInitializer` class, this file will be
picked up automatically by the Servlet contatiner and used to initialize the
application.

## The Servlet Configuration

Here is where we configure the Spring Dispatcher Servlet. This is also where you
would configure the controllers, and view resolvers.

```
package com.kylegengler.basic;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.thymeleaf.spring4.SpringTemplateEngine;
import org.thymeleaf.spring4.templateresolver.SpringResourceTemplateResolver;
import org.thymeleaf.spring4.view.ThymeleafViewResolver;
import org.thymeleaf.templatemode.TemplateMode;

/**
 * Configure the Spring dispatcher servlet
 * 
 * The configuration annotation designates this as a Spring configuration.
 * EnableWebMvc - tells spring to import and set up any configuration needed for
 * the controllers and any other mvc components. ComponentScan - Tells spring
 * the base package(s) to scan on the classpath for any spring @Components, or
 * any of its derivitives such as @Service, @Controller, or @Repository
 *
 * This file will specify any @Beans that need to be managed by the Spring IoC
 * container for Dependency Injection
 *
 * @author kyle
 */
@Configuration
@EnableWebMvc
@ComponentScan({ "com.kylegengler" })
public class ServletConfiguration extends WebMvcConfigurerAdapter {

    @Autowired
    private ApplicationContext ctx;

    /**
     * Set up the template resolver. This will look for templates in the
     * /WEB-INF/templates directory, ending with html. This is used to find the
     * template files needed to generate the view
     */
    @Bean
    public SpringResourceTemplateResolver templateResolver() {
        SpringResourceTemplateResolver templateResolver = new SpringResourceTemplateResolver();
        templateResolver.setApplicationContext(ctx);
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        templateResolver.setCacheable(true);
        return templateResolver;
    }

    /**
     * The engine used to render the view using the template file and model.
     */
    @Bean
    public SpringTemplateEngine templateEngine() {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver());
        templateEngine.setEnableSpringELCompiler(true);
        return templateEngine;
    }

    /**
     * The view resolver receives the name of the view, and is responsible for
     * creating the corresponding View object to go with that view. It ties
     * together the Controller logic and template engine.
     */
    @Bean
    public ThymeleafViewResolver viewResolver() {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setTemplateEngine(templateEngine());
        return viewResolver;
    }

}
```

## The Controller

This is where we create a basic Controller that will be picked up by Spring's
classpath scanning, and managed by the IoC container.

```
package com.kylegengler.basic;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * Set up a simple controller for the application
 * 
 * @author kyle
 */
@Controller
public class IndexController {

    /**
     * This will use the template at /WEB-INF/templates/index.html for the root
     * and /home url paths.
     */
    @RequestMapping({ "/", "/home" })
    public String getIndex() {
        return "index";
    }
}
```

## The Template

Now we just need to create the template file that will be used by Thymeleaf to
generate the view. As shown in the controller, this will use the
`/WEB-INF/templates/index.html` file.

```
<!DOCTYPE html>
<html>
<head>
    <title>Hello World!</title>
</head>
<body>
    <h1>Hello World!</h1>
    <p th:text="${#dates.format(#dates.createNow(), 'dd MMM yyyy HH:mm')}">Current Timestamp</p>
</body>
</html>
```

## The Deployment

Build the application using your preferred build tool and deploy the resulting
war to the application server. For Tomcat, you can just copy it into the
`webapps` directory. Once the application is deployed you can hit the `/` or
`/home` paths of the servlet. For Tomcat this might be
[http://localhost:8080/BasicWebApp/](http://localhost:8080/BasicWebApp/).

And you should see:

![Basic Web App Home](/assets/posts/2017/02/07/basic-web-app-home.png)

