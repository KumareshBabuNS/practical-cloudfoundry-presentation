<!SLIDE cover>

# Practical Cloud Foundry #

## Phil Webb ##

<!SLIDE>

# Welcome #

* https://github.com/philwebb/practical-cloudfoundry
* Apache 2.0 licensed
* twitter: @phillip_webb

.notes provide a brief overview of the contents

<!SLIDE>

# WaveMaker #

.notes TODO wavemaker screen shot.  Aquired 2011 100K+ java

<!SLIDE>

# WaveMaker #

* Local Web Application
* Projects saved on disk
* Uses JDK Java Compiler
* Deploys to local tomcat


<!SLIDE>

# WaveMaker #

* <del>Local</del> Web Application
* Projects saved <del>on disk</del>
* Uses <del>JDK</del> Java Compiler
* Deploys <del>to local tomcat</del>

<!SLIDE>

# Overview #

.notes TODO should we put an overview here?

<!SLIDE>

# Cloud PITA Factor #

* local is PITA to setup but good to develop
* cloud is opposite

<!SLIDE>

# General Tips #

* Use logging
* JUnit

<!SLIDE subsection>

# Refactor &#8594; Extract Interface #

<!SLIDE>

# Example #

	@@@ java
	@Controller
	public class MainController {

		@RequestMapping({ "/", "/index.html" })
		public ModelAndView index() {
			return new ModelAndView("index", 
				Collections.singletonMap("username", 
					System.getProperty("user.name")));
		}
	}
	
<!SLIDE>

# Example #

	@@@ java







					System.getProperty("user.name")));
	
<!SLIDE>

# Example #

	@@@ java
	public interface UserDetails {
		String getUsername();
	}

<!SLIDE>

# Example #

	@@@ java
	public interface UserDetails {
		String getUsername();
	}


	@Component
	public class LocalUserDetails implements UserDetails {

		@Override
		public String getUsername() {
			return System.getProperty("user.name");
		}
	}

<!SLIDE>

# Example #

	@@@ java
	@Controller
	public class MainController {

		@Autowired
		private UserDetails userDetails;

		@RequestMapping({ "/", "/index.html" })
		public ModelAndView index() {
			return new ModelAndView("index", 
				Collections.singletonMap("username", 
					userDetail.getUsername()));
		}
	}


<!SLIDE subsection>

# New &#8594; Class #

<!SLIDE>

# Example #

	@@@ java
	@Component
	public class CloudUserDetails implements UserDetails {

		@Override
		public String getUsername() {
			return CloudEnvironment.current().getUser();
		}
	}

<!SLIDE>

# #FAIL #

<pre>
org.springframework.beans.factory.NoSuchBeanDefinitionException: 
<strong style="font-size: 2em;">No unique bean</strong> of type [org.cloudfoundry.practical.
demo.core.UserDetails]  is defined: expected single matching 
bean but <strong style="font-size: 2em;">found 2</strong>: [<em>localUserDetails</em>, <em>cloudUserDetails</em>]

at org.springframework.beans.factory.support....
</pre>

<!SLIDE>

# Never the twain shall meet #

* Local or Cloud Strategy
* Use Build System 
	* e.g. Maven Dependencies
* Use Spring Profiles

<!SLIDE>

# Spring profiles in XML #

	@@@ xml
	<beans profile="default">
		<bean class="org.cloudfoundry.practical.demo.
			local.LocalUserDetails">
	</beans>

	<beans profile="cloud">
		<bean class="org.cloudfoundry.practical.demo.
			cloud.CloudUserDetails"
	</beans>

<!SLIDE>

# Spring profiles in code #

	@@@ java
	package org.cloudfoundry.practical.demo.local;

	@Configuration
	@Profile("default")
	@ComponentScan
	public class LocalConfiguration {
	}


	package org.cloudfoundry.practical.demo.cloud;

	@Configuration
	@Profile("cloud")
	@ComponentScan
	public class CloudConfiguration {
	}

<!SLIDE>

# Gotcha #

* Cloud foundry Servlet 2.5 means web.xml

<p/>
	
	@@@ xml
	<param-name>contextClass</param-name>
	<param-value>
		org.springframework.web.context.support.
			AnnotationConfigWebApplicationContext
	</param-value>

	<param-name>contextConfigLocation</param-name>
	<param-value>
		org.cloudfoundry.practical.demo.RootConfiguration
	</param-value>

<!SLIDE subsection>

# Profiles Demo #

<!SLIDE>

.notes picture here

<!SLIDE>

# Environment Variables #

	$ vmc env-add myapp key=value

<p/>

	@@@ java
	public void example() {
		String value = System.getEnv("key")
		// ... do something with value
	}

	// or if a spring bean

	@Value("#{systemEnvironment['key']}")
	private String value;

<!SLIDE subsection>

# File &#8594; Open File... #

<!SLIDE>

# File System #

* Local storage will disappear when you application stops, crashes or moves
* Use Mongo GridFS
* There are limits
	* 2GB Disk
	* 240MB Mongo
* Blobstore is coming

<!SLIDE>

# Resource Abstraction #

	@@@ java
	package org.cloudfoundry.tools.io;


	public interface Resource {
		// ...
	}

	public interface Folder extends Resource {
		// ...
	}

	public interface File extends Resource {
		// ...
	}

<!SLIDE>

# Local implementation #

<!SLIDE>

# Temp Disk #

<!SLIDE>

# Embrace the change #

* Differences that help
* It you need to change it, make it better

<!SLIDE>

# How to @Configure mongo #


<!SLIDE >

# Demo File System #

<!SLIDE>

# Chunked Problem #

* Wireshark

<!SLIDE>

# 504 Gateway Timeouts #

* Default
* Drip
* Shim

<!SLIDE>

# Java Compiler #

<!SLIDE>

# Tomcat Stand Alone #

<!SLIDE>

# Tunnel Debug #

<!SLIDE>

# Micro Cloudfoundry Tips #

* Add memory
* Start with -debug
* Disk space
* Cookie Timeouts
* VCAP.me + DNS + Tunnel
* SSH to play
* nginx timeouts


