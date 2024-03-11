---
title: "Simple Spring project using Spring Boot and Security"
date: 2016-07-11 19:00:00 +0800
categories: [tutorials]
tags: [spring, java, web]
---
[Spring](https://spring.io/) is a web-development framework for Java which allows you to create web-applications in a very organised and intuitive manner.

The Spring framework provides many many features that I was actually surprised they even exist when I came to look more into the functions for the purposes of my development. In this little guide I will give you some very basic startup tips and code to get a basic Spring development setup going and I hope it will be useful for you all!

# Prerequisites
I will be using Gradle to develop this project and it's very important to understand the [differences](https://gradle.org/maven_vs_gradle/) between [Gradle](https://gradle.org/) and [Maven](https://maven.apache.org/). Personally, I prefer Gradle because of my background with developing Android applications and the fact that Gradle is very well developed in my opinion and provides all the resources you need. Most tutorials and guides that I've come across with Spring use Maven which can be a little confusing once your project starts to expand.  
  
Note: I am using Linux for development, so if I make some use of Linux then it's good to be aware that this may not work on all operating systems, though I think it's pretty similar on other OS's too.
  
- Java (I'm using `1.8.0_91`)
- Gradle
- Eclipse (Preferable for Syntax highlighting and libraries)
- MySQL (I don't use any other engines by Spring does have support for them)

# File Structure
The Spring file structure is very simple and quite important because of the way that Spring functions. I'm going to call my project webAppProject and below is how the tree will look for my project.

Note: You can use the [Spring Initializr](https://start.spring.io/) to init your file structure with the Gradle files if you wish and make it look like I have it, I don't prefer using the Initlializr so I usually create my packages myself. 

```
├── bin
├── gradle
│   └── wrapper
└── src
    ├── main
    │   ├── java
    │   │   └── webAppProject
    │   │       ├── controller
    │   │       ├── domain
    │   │       └── persistence
    │   │           ├── repository
    │   │           └── services
    │   ├── resources
    │   └── webapp
    │       ├── styles
    │       └── WEB-INF
    │           └── views
    └── test
        ├── java
        │   └── webAppProject
        └── resources
```

The `src` folder will include everything related to the project so you can ignore the other directories for now.

Explanation of the directories:

`/src/main/java` is the folder containing the source java files, the sub-packages are important in this directory.  
`/src/main/java/controller` includes all the files which handle requests to the web-application. All the files for functioning the web-application will be places here.  
`/src/main/java/domain` includes all the domain objects that will be used in the application, e.g. `User` or `Role`.  
`/src/main/java/persistence` includes all the classes (in `repository` which will be used for persisting data to our database.  It will also include the `DbConfig` file which is used for connecting to the database.  
`/src/main/java/resources` includes resource files for the application to use such as property files and other resources required.  
`/src/main/webapp/` will include the web-pages in `.jsp` that will be viewable. It can also contain all other content related to web views.  
  
In this application, we will also be utilising [JUnit](https://junit.org/junit4/) for testing our application and thus we have the `/src/test` directory which will contain those test files.

# build.gradle

To get started, we will need to start a Gradle project, you can do this by using the following command:
```
$ gradle init
```
This will create a basic Gradle file-structure for you with the files that Gradle uses to download dependencies and project-specific settings that you set.

The main file to focus on here is the `build.gradle` file. My file looks like the following after importing all the Spring stuff.
```
buildscript {
    ext {
        springBootVersion = '1.3.0.M2'
    }
    repositories {
        mavenCentral()
        maven { url "https://repo.spring.io/snapshot" }
        maven { url "https://repo.spring.io/milestone" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'spring-boot' 

jar {
    baseName = 'webAppProject'
    version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.7
targetCompatibility = 1.7

repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/snapshot" }
    maven { url "https://repo.spring.io/milestone" }
}


dependencies {
	compile("org.springframework.boot:spring-boot-starter-web")
	testCompile("org.springframework.boot:spring-boot-starter-test")
    // JPA
	compile("org.springframework.boot:spring-boot-starter-data-jpa")
	runtime("mysql:mysql-connector-java")
	// JSP
	compile("org.apache.tomcat.embed:tomcat-embed-jasper:8.0.23")
	compile("javax.servlet:jstl:1.2")
	// Security
    compile("org.springframework.boot:spring-boot-starter-security")
	testCompile("org.springframework.security:spring-security-test:4.0.1.RELEASE")
	// cucumber
	testCompile("info.cukes:cucumber-spring:1.2.3")
	testCompile("info.cukes:cucumber-java:1.2.3")
	testCompile("info.cukes:cucumber-junit:1.2.3")
	testCompile("junit:junit:4.11")
	    // presentation
    compile("org.webjars:handsontable:0.17.0")
    compile("org.webjars:jquery:2.0.3-1")
    compile("org.webjars:bootstrap:3.3.5")
   compile("com.google.code.gson:gson:1.7.2")
}

eclipse {
    classpath {
         containers.remove('org.eclipse.jdt.launching.JRE_CONTAINER')
         containers 'org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8'
    }
}

task wrapper(type: Wrapper) {
    gradleVersion = '2.3'
}
```

# Connecting to the database

It's time to connect to the database, to do this create the file `/src/main/java/persistence/DbConfig.java` and add the following to it:

```
package webAppProject.persistence;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

@Configuration
public class DbConfig {
	@Bean
	public DriverManagerDataSource dataSource() {		
		DriverManagerDataSource ds = new DriverManagerDataSource();
		ds.setDriverClassName("com.mysql.jdbc.Driver");
		ds.setUrl("jdbc:mysql://localhost:3306/webapp");
		ds.setUsername("root");
		ds.setPassword("");
    		return ds;
	}
}
```

You will need to fill in the information with your own to make sure that the database connects. 

# Generating SSL

Spring Security is very strict with how the security of the web-app works and I've had many issues in the past because of SSL so we will be using a simple generated SSL certificate for the application. To do this, first `cd` into your project root directory then run the following (please make sure you have the directory structure as I've mentioned above implemented).
```
$ cd src/main/resources/
$ keytool -genkeypair -alias tomcat -keyalg RSA -keystore ./keystore.jks
```
Go through generating a key and then the `keystore.jks` file will be generating which will be the SSL certificate we will be using.

# Spring Security

Spring Security is a very neat feature of Spring and offers a tonne of resources and functionality to make your web-application secure. It manages Beans, sessions and permissions and it's worth having a look at the [documentation](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/) for Spring Security to familiarise yourself with how it works.  
  
To get started, we will need to create the file `/src/main/java/webAppProject/SecurityConfig.java`.  
Note: I am using Java configuration files *NOT* XML. 

```
package webAppProject;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	protected void configure(HttpSecurity http) throws Exception {
		http
    			.authorizeRequests()
					.antMatchers("/", "/login-form", "/logout", "/success-login", "/error-message")
						.hasAnyRole("USER","ADMIN")
	    			.and()
	    		.authorizeRequests()
					.antMatchers("/", "/UserPage")
						.hasAnyRole("USER")
	    			.and()
	    		.authorizeRequests()
					.antMatchers("/", "/AdminPage")
						.hasAnyRole("ADMIN")
	    			.and()
			    .formLogin()
					.loginPage("/user-login") 
					.defaultSuccessUrl("/success-login",true) // the second parameter is for enforcing this url always
					.loginProcessingUrl("/login")
					.failureUrl("/error-login")
					.permitAll()
					.and()
				.logout()
					.logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
					.logoutSuccessUrl("/user-logout")
			        .and()
			    .requiresChannel()
			    	.anyRequest()
			    	.requiresSecure()
	    			.and()
	   			.exceptionHandling().accessDeniedPage("/user-error");
		}

	@Autowired 
	private UserDetailsService userDetailsService; 	

	@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
		BCryptPasswordEncoder pe = new  BCryptPasswordEncoder();
		auth.userDetailsService(userDetailsService).passwordEncoder(pe);
	}
}
```

Using `HttpSecurity` allows us to create permissions for URL and request-mappings which is very useful to define which user can access what page depending on their role. This is very effective when building a role-driven application where many users will have different privileges allowing the application to be very secure without having to do a lot of work in the controllers. We are also going to be using the [@Autowired](https://docs.spring.io/spring-framework/docs/2.5.x/api/org/springframework/beans/factory/annotation/Autowired.html) annotation which allows us to inject a Bean into our class allowing us to use an interface within the class. I am also using [BCryptPasswordEncoder](https://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/crypto/bcrypt/BCryptPasswordEncoder.html) to securely store passwords in the database as hashed which is a very good practice and an important one at that if you're building a big application.
  
# UserDetailsService 

The UserDetailsService is the core interface used to load user-specific data. So if a user is logged in and we wish to retrieve information about them then this service is useful to use. In this case, Spring Security will use the interface to see user-roles before allowing them to continue onto a request.

We will be creating a CustomUserDetails service in `/src/main/java/persistence/services/CustomUserDetailsService.java` and adding the following to it.

```
package webAppProject.persistence.services;

import static webAppProject.webAppProjectApplication.ROLE_USER;
import static webAppProject.webAppProjectApplication.ROLE_ADMIN;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import webAppProject.persistence.repository.UserRepository;

@Service
@Transactional(readOnly=true)
public class CustomUserDetailsService implements UserDetailsService {
	
	@Autowired
	private UserRepository userRepo;	

	public UserDetails loadUserByUsername(String login)
			throws UsernameNotFoundException {
		
		webAppProject.domain.User domainUser = userRepo.findByLogin(login);
		
		boolean enabled = true;
		boolean accountNonExpired = true;
		boolean credentialsNonExpired = true;
		boolean accountNonLocked = true;

		return new User(
				domainUser.getLogin(), 
				domainUser.getPassword(), 
				enabled, 
				accountNonExpired, 
				credentialsNonExpired, 
				accountNonLocked,
				getAuthorities(domainUser.getRole().getId())
		);
	}
	
	private Collection<? extends GrantedAuthority> getAuthorities(Integer role) {
		List<GrantedAuthority> authList = getGrantedAuthorities(getRoles(role));
		return authList;
	}
	
	private List<String> getRoles(Integer role) {

		List<String> roles = new ArrayList<String>();

		if (role.intValue() == ROLE_USER) {
			roles.add("ROLE_USER");
		} else if (role.intValue() == ROLE_ADMIN) {
			roles.add("ROLE_ADMIN");
		}
		return roles;
	}
	
	private static List<GrantedAuthority> getGrantedAuthorities(List<String> roles) {
		List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
		
		for (String role : roles) {
			authorities.add(new SimpleGrantedAuthority(role));
		}
		return authorities;
	}
}
```

# Application Runner

An [ApplicationRunner](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationRunner.html) is used for initialising Beans and data when you boot the application. In this file, we will create basic data-entries to the database to test and see the functionality of the application as well as set some attributes for our user roles.  
`/src/main/java/webAppProject/webAppProjectApplicaton.java`  
```
package webAppProject;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import webAppProject.domain.Role;
import webAppProject.domain.User;
import webAppProject.persistence.repository.UserRepository;

@SpringBootApplication
public class webAppProjectApplication implements ApplicationRunner {

    public static void main(String[] args) {
        SpringApplication.run(webAppProjectApplication.class, args);
    }

	@Autowired
	private UserRepository userRepo;
    
	public static final int ROLE_ADMIN = 1;
	public static final int ROLE_USER = 2;
	
    @Override
	public void run(ApplicationArguments args) throws Exception {
		BCryptPasswordEncoder pe = new  BCryptPasswordEncoder();

		User user = new User();
		user.setLogin("alice");
		user.setPassword(pe.encode("password"));
		Role role = new Role();
		role.setId(ROLE_USER);
		role.setRole("USER");
		user.setRole(role);
		userRepo.save(user);
		
		user = new User();
		user.setLogin("bob");
		user.setPassword(pe.encode("admin"));
		role = new Role();
		role.setId(ROLE_ADMIN);
		role.setRole("ADMIN");
		user.setRole(role);
		userRepo.save(user);
		
    }
    
}

```

# WebConfig

Now we can configure some of the web stuff we'll be using. To do this we can go ahead and create the following.
`/src/main/java/webAppProject/WebConfig.java` 
```
package webAppProject;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;

@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

	// Handles HTTP GET requests for /resources/** by efficiently serving up static 
	// resources in the ${webappRoot}/resources/ directory
	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
	}
	  
	//Resolves views selected for rendering by @Controllers to .jsp resources in the 
	// /WEB-INF/views directory
	@Bean
	public InternalResourceViewResolver viewResolver() {
		InternalResourceViewResolver viewResolver = 
                        new InternalResourceViewResolver();
		viewResolver.setViewClass(JstlView.class);
		viewResolver.setPrefix("/WEB-INF/views/");
		viewResolver.setSuffix(".jsp");
		viewResolver.setOrder(2);
		return viewResolver;
	}
}

```

Here we just outline some of the structure we are using and also where our views are stored. We will also have no `.jsp` extension for our views and they'll be accessible by just going to `https://localhost:8090/login` etc.

# Domains

Domains are the entities in our database and to construct them we will use a [POJO](https://stackoverflow.com/questions/3527264/how-to-create-a-pojo?answertab=votes#tab-top). For our case we will be creating two domains, `User` and `Role` and to do this we will use the following files.

`/src/main/java/webAppProject/domain/User.java`
```
package webAppProject.domain;

import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.JoinTable;
import javax.persistence.OneToOne;
import javax.persistence.Table;

@Entity
@Table(name="users")
public class User {
	
	@Id
	@GeneratedValue
	private Integer id;
	
	private String login;
	
	private String password;
	
	private boolean enabled;
	
	@OneToOne(cascade=CascadeType.ALL)
	@JoinTable(name="user_roles",
		joinColumns = {@JoinColumn(name="user_id", referencedColumnName="id")},
		inverseJoinColumns = {@JoinColumn(name="role_id", referencedColumnName="id")}
	)
	private Role role;

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getLogin() {
		return login;
	}

	public void setLogin(String login) {
		this.login = login;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public Role getRole() {
		return role;
	}

	public void setRole(Role role) {
		this.role = role;
	}

	public boolean isEnabled() {
		return enabled;
	}

	public void setEnabled(boolean enabled) {
		this.enabled = enabled;
	}	
	
}
```

`/src/main/java/webAppProject/domain/Role.java`
```
package webAppProject.domain;

import java.util.Set;

import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.JoinTable;
import javax.persistence.OneToMany;
import javax.persistence.Table;

@Entity
@Table(name="roles")
public class Role {
	
	@Id
	private Integer id;
	
	private String role;
	
	@OneToMany(cascade=CascadeType.ALL)
	@JoinTable(name="user_roles", 
		joinColumns = {@JoinColumn(name="role_id", referencedColumnName="id")},
		inverseJoinColumns = {@JoinColumn(name="user_id", referencedColumnName="id")}
	)
	private Set<User> userRoles;

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getRole() {
		return role;
	}

	public void setRole(String role) {
		this.role = role;
	}

	public Set<User> getUserRoles() {
		return userRoles;
	}

	public void setUserRoles(Set<User> userRoles) {
		this.userRoles = userRoles;
	}
	
	
}
```
  
In these entities we are using annotations such as `@OneToOne` and `@OneToMany`, and these annotations describe the relationship of one attribute with another in *some* other table. When using these annotations it's extremely important to take care with not only the attributes of all the tables you're dealing with in the relationship but also the format of them.

`@OneToOne` - An annotation describing a one-to-one relationship that is formed by the columns as stated in the `@JoinTable` annotation.

`@JoinTable` - Defines an abstract link table for describing relationships such as `OneToOne`, `OneToMany` and `ManyToMany`. The format is specified as follows: 
```
@JoinTable(name="jointablename", 
	joinColumns = {@JoinColumn(name="nameOfFirstColumnInJoinTable", referencedColumnName="actualColumnFromCurrentClass")},
	inverseJoinColumns = {@JoinColumn(name="nameOfFirstColumnInJoinTable", referencedColumnName="actualColumnFromReferencingClass")}
) private ReferencingClass refc;
```

Sometimes you may come across `@JoinColumn` on it's own and this is usually used for `@ManyToOne` relationships where there is no need for an extra table to represent the relationship. Consider reading through some [content](https://www.javaworld.com/article/2077817/java-se/understanding-jpa-part-1-the-object-oriented-paradigm-of-data-persistence.html) on how JPA's work and how these relationships are formed in hibernate.

# Repository

A repository is similar to a *DatabaseAccessObject* and will be used for saving, retrieving, updating and deleting entries from the database. For this we will be using the *CrudRepository* interface which provides us a good set of functionality to work with Hibernate.

We will need to create repositories for both of our domains to interact with those objects in the persistence scope, so we create the following two files.

`/src/main/java/webAppProject/persistence/repository/UserRepository.java`
```
package webAppProject.persistence.repository;
import org.springframework.data.repository.CrudRepository;
import webAppProject.domain.User;

public interface UserRepository extends CrudRepository<User, Integer> {
	public User findByLogin(String login);
}
```

`/src/main/java/webAppProject/persistence/repository/RoleRepository.java`
```
package webAppProject.persistence.repository;
import org.springframework.data.repository.CrudRepository;
import webAppProject.domain.Role;

public interface RoleRepository extends CrudRepository<Role, Integer> {
	public Role findById(int id);
}
```

Now when we want to access objects from our database we can do things like:
```
@Autowired UserRepository userRepo;

String encryptedPasswordForBob = userRepo.findByLogin("bob").getPassword();
```

This will find the user object with the name "bob" in the database and then return the object, we can then call our POJO methods on this to retrieve information for that object.

# Checking if it builds

Now that you have the main code setup you can check that everything builds and that there are no issues, to do this run the following command:
```
$ gradle build
```

You should see a success message if everything goes smoothly.

# Controllers

Controllers in Spring are used to handle requests from the user. These requests are HTTP requests that a user sends, either GET or POST. So to handle these requests we need to know what to do and what to show when a user goes on a certain page. Let's start off by handling the request to the login page, to the logout URL and to the index page when you're successfully logged in. We can do this by using the following:

`/src/main/java/webAppProject/controller/AuthorizationController.java`
```
package webAppProject.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class AuthorizationController {
	
	@RequestMapping(value="/user-login", method=RequestMethod.GET)
	public ModelAndView loginForm() {
		return new ModelAndView("login-form");
	}

	@RequestMapping(value="/error-login", method=RequestMethod.GET)
	public ModelAndView invalidLogin() {
		ModelAndView modelAndView = new ModelAndView("login-form");
		modelAndView.addObject("error", true);
		return modelAndView;
	}
	
	@RequestMapping(value="/success-login", method=RequestMethod.GET)
	public ModelAndView successLogin() {
		return new ModelAndView("redirect:/");
	}

	@RequestMapping(value="/user-logout", method=RequestMethod.GET)
	public ModelAndView logout() {
		ModelAndView modelAndView = new ModelAndView("login-form");
		modelAndView.addObject("logout", true);
		return modelAndView;
	}
	
	@RequestMapping(value="/user-error", method=RequestMethod.GET)
	public String error() {
		return "/error-message";
	}
}
```

Using `@RequestMapping` we can find the mapping that a user is requesting and handle it accordingly. In addition to this we are using a `ModelAndView` which will render a view that we want for that request. I.e. if I want to access `https://localhost:8090/user-login` I won't be shown `login.jsp`, instead I will be shown `login-form.jsp` as specified in the mapping for that URL.

Now we can make another controller to handle our pages as specified in the `SecurityConfig.java` file earlier for the mappings `/UserPage` and `/AdminPage`.

`/src/main/java/webAppProject/controller/PageController.java`
```
package webAppProject.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class PageController {

	@RequestMapping(value="/", method=RequestMethod.GET)
	public ModelAndView index(){
		return new ModelAndView("index");
	}

	@RequestMapping(value="/UserPage", method=RequestMethod.GET)
	public ModelAndView loginForm() {
		return new ModelAndView("userpage");
	}

	@RequestMapping(value="/AdminPage", method=RequestMethod.GET)
	public ModelAndView invalidLogin() {
		return new ModelAndView("adminpage");
	}
}
```

Now that we have created our mappings it's time to create the views in jsp.

# JSP Views

For the sake of speeding up this process I'll just paste my jsp files below because they're essentially just HTML but you can use JSTL tags like I have in the `index.jsp` file. 

`/src/main/webapp/WEB-INF/views/login-form.jsp`
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "https://www.w3.org/TR/html4/loose.dtd">
<%@taglib uri="https://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
<title>Login</title>
</head>
	<c:if test="${logout == true}">
		<b class="logout">You have been logged out.</b>
	</c:if>
	<c:url value="/login" var="loginUrl"/>
	<form action="${loginUrl}" method="post" modelAttribute="user">
		Username: <input type="text" id="username" name="username" placeholder=""><br>
		Password: <input type="password" id="password" name="password" placeholder=""><br>
		
		<input type="hidden"
		name="${_csrf.parameterName}"
		value="${_csrf.token}"/>
		<button type="submit">Login</button>
	        <c:if test="${error == true}">
				    <font color="red">Invalid username or password</font>
			</c:if>
	</form>
</body>
</html>
```

`/src/main/webapp/WEB-INF/views/error-message.jsp`
```
<h1>ERROR 403</h1>
```

`/src/main/webapp/WEB-INF/views/index.jsp`
```
You are logged in as ${pageContext.request.userPrincipal.name}
<br><br>
<a href="UserPage">Click here to go to the user page</a><br>
<a href="AdminPage">Click here to go to the admin page</a>
```

`/src/main/webapp/WEB-INF/views/userpage.jsp`
```
<h1>this is the user page, an admin should not be able to see this</h1>
```

`/src/main/webapp/WEB-INF/views/adminpage.jsp`
```
<h1>this is the admin page, a user should not be able to see this</h1>
```

# Running the application

Now that we have everything we need, we can run the application. First make sure that no program or service is already utilising your port `8090` and then go to the shell and in the root of the project run the following command:
```
$ gradle clean && gradle cleanCheck
$ gradle bootRun
```

If everything works correctly you should see a bunch of green writing and after a few seconds you should be able to access `https://localhost:8090`. When you go on the URL for the first time you'll be asked about the SSL, just *proceed anyway` and then you should be able to login. 

As defined above, if you login as `alice` and `password` you should be logged in as alice and should **only** be able to access `/UserPage`, however if you try to access `/AdminPage` as alice, you will be prompted with a `403` page. Vice versa for the admin! 

# Conclusion

In this lengthy guide, I hope that you have learnt a good amount on the basic Spring functionality and how to setup your own project from scratch. In my next Spring guide I will cover the testing methods using `MockMvc` and `JUnit`. So stay tuned for that and please leave commends below if you face problems or are seeking advice. 

Check out the entire code in this article on my repository: https://github.com/redrails/SpringMVCBasicProject

Feel free to clone and go ham with it!