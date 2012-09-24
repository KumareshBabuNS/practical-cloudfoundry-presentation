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

&lt; screen shot here &gt;

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

* Local is PITA to setup but good to develop
* Cloud is opposite

&lt; image here &gt;

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

# Recap #

<pre>

	+-----------------------+
	| Code          Feature |
	+-----------------------+
</pre>

<!SLIDE>

# Recap #

<pre>

	+------------||---------+
	| Code       || Feature |
	+------------||---------+
	          interface

</pre>

<!SLIDE>

# Recap #

<pre>
                  +---------+
	+------------|| Cloud   |
	| Code       |+---------+
	+------------|| Local   |
	              +---------+
</pre>


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
* Mongo GridFS is an alternative
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

	@@@ java
	Folder folder = new LocalFolder("/path/on/disk");

	File file = folder.getFile("subfolder/file.txt");
	InputSteam stream = file.getContent().asInputStream();
	try {
		// do something ...
	} finally {
		stream.close();
	}

<!SLIDE>

# Mongo implementation #

	@@@ java
	Folder folder = new MongoFolder(db, "bucket");

	File file = folder.getFile("subfolder/file.txt");
	InputSteam stream = file.getContent().asInputStream();
	try {
		// do something ...
	} finally {
		stream.close();
	}

<!SLIDE>

# Embrace the change #

* If you need to change your code, make it better
* Design your API around **your** needs
* Make it hard to do the wrong thing

<!SLIDE>

# Example #

	@@@ java
	import java.io.File;

	public void someMethod(File folder) {
		if(!folder.isDirectory()) {
			throw new IllegalArgumentException("Must be a folder");
		}
		// ...
	}

<!SLIDE>

# Example #

	@@@ java
	import org.cloudfoundry.tools.io.Folder;

	public void someMethod(Folder folder) {
		// ...
	}

<!SLIDE>

# Example #

	@@@ java
	import java.io.File;

	public void someMethod(File file) throw IOException {
		StringBuffer content = new StringBuffer();
		Reader reader = new FileReader(filePath));
		try {
			char[] buf = new char[1024];
			int numRead=0;
			while((numRead=reader.read(buf)) != -1){
				String readData = String.valueOf(buf, 0, numRead);
				fileData.append(readData);
				buf = new char[1024];
			}
		finally {
			reader.close();
		}
		// do something with content
	}

<!SLIDE>

# Example #

	@@@ java
	import org.cloudfoundry.tools.io.File;

	public void someMethod(File file) {
		String content = file.getContent().asString();
		// do something with content
	}

	// Only throws RuntimeExceptions, IOException is wrapped

.notes Highlight no IOException

<!SLIDE>

# Content #

	@@@ java
	file.getContent()

		String asString();
		byte[] asBytes();

		InputStream asInputStream();
		OutputStream asOutputStream();
		
		Reader asReader();
		Writer asWriter();

		void write(String content)
		void write(InputStream content)
		void write(Reader content);

		void copyTo(OutputStream outputStream)
		void copyTo(Writer writer)

<!SLIDE>

# Resources and filtering

	@@@ java

	// recursively delete backup files
	folder.find().files().include(
			FilterOn.names().ending(".bak")
		).delete();



	// copy immediate folders (excluding *.~*)
	folder.list().folders().exclude(
			FilterOn.antPattern("*.~*")
		).copyTo(destinationFolder);

<!SLIDE>

# Zip Files #

	@@@ java

	// Create a zip stream
	InputStream zipStream = ZipArchive.compress(folder);

	// Unpack a zip
	ZipArchive.unpack(zipStream, destinationFolder);

	// Read a zip file as if it is a folder
	Folder zip = new ZipArchive(file);
	zip.getFile("/inside/zip/file.txt").getContent().asString();


.notes Compress can handle resources

<!SLIDE>

# Virtual Folders #

	@@@ java
	// Virtual folder exist in memory
	Folder folder = new VirtualFolder();
	folder.getFile("/a/b.txt").getContent().write("in memory")

	// Only overwritten data is stored
	otherFolder.copyContentsTo(folder);
	folder.getFile("a.txt").getContent().write("replaced");
	folder.getFile("b.txt").getContent().asString();

<!SLIDE subsection>

# Files Demo #

<!SLIDE>

# How to @Configure mongo #

	@@@ java
	@Configuration
	@Profile("cloud")
	@ComponentScan
	public class CloudConfiguration {

		@Autowired
		private MongoDbFactory mongo;

		@Bean
		public CloudMongoDbFactoryBean mongo() {
			return new CloudMongoDbFactoryBean();
		}
	}

<!SLIDE >

# Gottcha #

* OSX Finder with WebDav does not work
* Transfer-Encoding: chunked fails <sup>*</sup>
* Finding problems like this can be hard
	* Use server logs
	* Use packet sniffing (Wireshark)

<div class="footnote"><sup>*</sup> http://stackoverflow.com/questions/8528600/how-to-make-a-chunked-request-via-nginx</div>

<!SLIDE subsection>

# Timeout #

<!SLIDE>

# 504 Gateway Timeouts #

* You are not the only thing in the cloud
* Sockets / File Handles are a commodity
* You will be cut off after 30-60 seconds without traffic

<!SLIDE>

# Drip Feeding #

	@@@ java

	// If you keep data flowing you will not timeout

	@RequestMapping("/drip")
	public void drop(HttpServletResponse response) throws IOException {
		ServletOutputStream outputStream = response.getOutputStream();
		while(gotWorkToDo()) {
			String fragment = doSomeWork(); // < 30 secs
			outputStream.write(fragment);
			response.flushBuffer();
		}
	}

<!SLIDE>

# Long Poll #

<pre>
[ Client ]              [ Server ]
    |-----------------------&gt;|
    |                        |
    |&lt;--------- 200 ---------|
</pre>

<!SLIDE>

# Long Poll #

<pre>
[ Client ]              [ Server ]
    |-----------------------&gt;|
    |                        |
    |                        |
    |                        |
    |&lt;--------- 504 ---------|
</pre>


<!SLIDE>

# Long Poll #

<pre>
[ Client ]              [ Nginx ] [ Tomcat ]
    |-----------------------&gt;|--------&gt;|
    |                        |         |
    |                        |         |
    |                        |         |
    |&lt;--------- 504 ---------|         |
                                       | continues to run
                                       X
                                 nowhere to go
</pre>

<!SLIDE>

# Long Poll #

<pre>
[ Client ]              [ Nginx ] [ Tomcat ]
    |-----------------------&gt;|--------&gt;|
    |                        |         |
    |                        |         |
    X abort                  |         |
    |---------- long poll --&gt;|--------&gt;|
    |                        |         |
    |                        |         | 
    |&lt;---------- 200 --------|&lt;--------|
</pre>

<!SLIDE>

# Long Poll Client Shim #

	@@@ html
	<script type="text/javascript" 
		th:src="@{/cloudfoundry/dojo-xhr-timeout-shim.js}"/>

<p/>

	@@@ javascript
	dojo.xhrPost({
		url : requestUrl,
		load : function(result) {
			// handle the result
		},
		error : function(result,ioargs) {
			// handle the error
		}
	});

<!SLIDE>

# Long Poll Server Shim #

	@@@ xml
	<filter>
		<filter-name>timeoutProtectionFilter</filter-name>
		<filter-class>org.springframework.web.filter.
				DelegatingFilterProxy</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>timeoutProtectionFilter</filter-name>
		<servlet-name>dispatcherServlet</servlet-name>
	</filter-mapping>

<!SLIDE>

# Long Poll Server Shim #

	@@@ java
	@Bean
	public TimeoutProtectionFilter timeoutProtectionFilter() {
		TimeoutProtectionFilter filter = 
			new TimeoutProtectionFilter();
		filter.setProtector(timeoutProtectionStrategy());
		return filter;
	}

	@Bean
	public TimeoutProtectionStrategy timeoutProtectionStrategy() {
		return new ReplayingTimeoutProtectionStrategy();
	}

<!SLIDE subsection>

# Timeout Demo #

<!SLIDE> 

# Security #

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


