---
title: "SpringMVC testing with JUnit and MockMvc"
author: sham
date: 2016-07-14 19:00:00 +0800
categories: [tutorials]
tags: [spring, java, web]
---

[In the last tutorial](https://ihtasham.com/devblog/posts/getting-started-with-spring), we covered how to build a simple SpringMVC web-application using Spring Security and Boot. In this tutorial we will be testing the controllers that we created with very easy to implement [MockMvc](https://docs.spring.io/autorepo/docs/spring-security/4.0.0.RELEASE/reference/html/test-mockmvc.html) tests.

# MockMvc
MockMvc is a tool that we can use to essentially mock HTTP requests with some data and see how our web-application reacts. To do this, we will use the MockMvc class provided by the Spring Framework and it will enable us to test the expected functionality of our application. Testing is extremely important in web-development as sometimes things tend to go south without us even expecting it. So writing integration tests for your application can be very useful not only for being satisfied with the *test passing* output but also because it enables you to notice problems if they may occur later on in the development. 

# build.gradle
To setup JUnit, we need to make sure that it is in the dependencies list. In the last tutorial I did include it but if you don't have it then add the following to your dependencies block:
```
    testCompile("junit:junit:4.11")
```
To get a friendly output for the tests I also have added the following code to this file so that we can see how many tests are passing:
```
test {
    testLogging {
        events "passed", "skipped", "failed", "standardOut", "standardError"
        showExceptions true
        exceptionFormat "full"
        showCauses true
        showStackTraces true
        
        afterSuite { desc, result ->
            if (!desc.parent) { // will match the outermost suite
                def output = "Results: ${result.resultType} (${result.testCount} tests, ${result.successfulTestCount} successful, ${result.failedTestCount} failed, ${result.skippedTestCount} skipped)"
                def startItem = '|  ', endItem = '  |'
                def repeatLength = startItem.length() + output.length() + endItem.length()
                println('\n' + ('-' * repeatLength) + '\n' + startItem + output + endItem + '\n' + ('-' * repeatLength))
            }
        }
        
    }
}
```
This just shows us at the end of the testing process, how many tests ran, passed, failed or skipped and can be useful.

# Building the test file

To start writing tests we will create the file: `/src/test/java/webAppProject/AppTests.java`.

Before we can begin writing tests it is important to consider how the testing process works. 

In Spring Testing, we first define our configurations which allow us to use existing app classes to use for requests and entities. To do this we can use the following annotations:
```
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@SpringApplicationConfiguration(classes={
		webAppProjectApplication.class, 
		SecurityConfig.class, 
		WebConfig.class,
		})
public class AppTests {
```
Here we are requiring the [SpringJUnit4ClassRunner](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/context/junit4/SpringJUnit4ClassRunner.html) which provides functionality of the Spring TestContext Framework to standard JUnit tests. We are also using `WebAppConfiguration` which uses the configurations we have set for the application. The `@SpringApplicationConfiguration` allows us to essentially require some classes that our app depends on to run and these classes are helpful for later use of authentication and requests.

`@SpringApplicationConfiguration` is the most important annotation of all, this specifies that we are testing a Sprint **boot** application and instead of using the standard `@ContextConfigurations` we will use this annotations which will invoke the application to boot before it starts testing. What this means is that the `ApplicationRunner` class will be invoked and so all of our pre-defined entities will build allowing us to be able to test those entities without loading in a `.sql` test scheme or create test entities. I've had huge issues with this before so it's very important that you treat Spring annotations with care and understand the reason for their use.

In Spring contexts are a very important aspect and provide interfaces depending on what type of context you're looking for. We will use the `WebApplicationContext` which allows us to communicate with the servlet and container allowing us to manage the application. 

>A filter is an object that performs filtering tasks on either the request to a resource (a servlet or static content), or on the response from a resource, or both.

So to be able to use requests and receive responses we will require the use of filters in the context of MockMvc. So now we can build our MockMvc object up like this:
```
public class AppTests {
	
	@Autowired
	private WebApplicationContext wac;
	  
	@Autowired
	private Filter springSecurityFilterChain;
		
	private MockMvc mockMvc;
	
	@Before
	public void setup(){		
		this.mockMvc = MockMvcBuilders.
						webAppContextSetup(this.wac)
						.addFilter(springSecurityFilterChain)
						.apply(springSecurity())
						.build();
	}
```

#@Test

Now we can start writing tests and it's very simple to do so, for instance, we can test a successful login by doing the following:
```
@Test
public void loggingInWithCorrectDetails() throws Exception {
	mockMvc.perform(post("https://localhost/login")
			.param("username", "alice")
			.param("password", "password")
			.with(csrf()))
			.andExpect(status().is(302))
			.andExpect(redirectedUrl("/success-login"))
			.andDo(print());
}
```
Firstly we use the `@Test` annotation to specify that the body method is a test. The test **must** have the `throws Exception` clause because the test will throw an exception if it fails, and we need to handle this.

The MockMvc format is very simple. First we use the `perform` method for performing a request and then specify the request type. In this case we are *posting* data to the login form. Inside the post param we specify the URL that we are posting to and then we can use the `param()` function to specify post parameters that we need to send as a header. Because our web-application is enforcing the use of `csrf()` in the `SecurityConfig` we need to send a csrf token also with the request otherwise the server will not register the request. Following this we can use the `andExpect()` function to specify the expected behaviour of the test and if everything is consistent with the controller and the code you're using to deal with these requests, the tests will pass.

Here are the set of tests I've written for the application:
```
package webAppProject;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;


import javax.servlet.Filter;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.SpringApplicationConfiguration;
import org.springframework.security.core.Authentication;
import org.springframework.security.test.context.support.WithUserDetails;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import webAppProject.persistence.repository.UserRepository;

import static org.springframework.security.test.web.servlet.setup.SecurityMockMvcConfigurers.springSecurity;
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@SpringApplicationConfiguration(classes={
		webAppProjectApplication.class, 
		SecurityConfig.class, 
		WebConfig.class,
		})
public class AppTests {
	
	@Autowired
	private WebApplicationContext wac;
	  
	@Autowired
	private Filter springSecurityFilterChain;
		
	private MockMvc mockMvc;
	
	@Before
	public void setup(){		
		this.mockMvc = MockMvcBuilders.
						webAppContextSetup(this.wac)
						.addFilter(springSecurityFilterChain)
						.apply(springSecurity())
						.build();
	}

	@Test
	public void cantViewPageIfNotLoggedIn() throws Exception {
		mockMvc.perform(get("https://localhost/"))
			   .andExpect(status().is(302))
			   .andExpect(redirectedUrl("https://localhost/user-login"))
			   .andDo(print());
	}
	
	@Test
	public void loggingInWithCorrectDetails() throws Exception {
		mockMvc.perform(post("https://localhost/login")
				.param("username", "alice")
				.param("password", "password")
				.with(csrf()))
				.andExpect(status().is(302))
				.andExpect(redirectedUrl("/success-login"))
				.andDo(print());
	}
	
	@Test
	public void loggingInWithIncorrectDetails() throws Exception {
		mockMvc.perform(post("https://localhost/login")
				.param("username", "alice")
				.param("password", "wrong")
				.with(csrf()))
				.andExpect(status().is(302))
				.andExpect(redirectedUrl("/error-login"))
				.andDo(print());	
		}
	
	@Test
	@WithUserDetails("alice")
	public void canAccessUserPageIfUser() throws Exception {
		mockMvc.perform(get("https://localhost/UserPage"))
			   .andExpect(status().isOk())
			   .andExpect(view().name("userpage"))
			   .andDo(print());
	}
	
	@Test
	@WithUserDetails("alice")
	public void cantAccessAdminPageIfUser() throws Exception {
		mockMvc.perform(get("https://localhost/AdminPage"))
			   .andExpect(status().is(403))
			   .andExpect(forwardedUrl("/user-error"))
			   .andDo(print());
	}
	
	@Test
	@WithUserDetails("bob")
	public void cantAccessUserPageIfNotUser() throws Exception {
		mockMvc.perform(get("https://localhost/UserPage"))
		   .andExpect(status().is(403))
		   .andExpect(forwardedUrl("/user-error"))
		   .andDo(print());	
	}
	
	@Test
	@WithUserDetails("bob")
	public void canAccessAdminPageIfAdmin() throws Exception {
		mockMvc.perform(get("https://localhost/AdminPage"))
		   .andExpect(status().isOk())
		   .andExpect(view().name("adminpage"))
		   .andDo(print());
	}
	
}

```

I've tested all the expected functionality that my application has outlined in the model and that's what you should be aiming to do also. It is very tedious sometimes to write tests and also boring but once you get into the habit of doing it, it's very useful for debugging code later and finding out what still works later on in the development.

#Running the tests
To run the tests we can use the following commands in the terminal:
```
$ gradle clean cleanCheck
$ gradle check
```

If your application behaves as expected, you should see near the end that all the tests are passing.

# Conclusion

I hope that this tutorial has helped you understand the important of testing as well as how tests can be implemented in Spring. It is very simple process but can be a complete mindf**k at times when you can't figure out where you're going wrong. As always the code for this tutorial is available on github and I've updated the original repo to contain the tests: https://github.com/redrails/SpringMVCBasicProject