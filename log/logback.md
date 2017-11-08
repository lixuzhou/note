[TOC]

[logback.qos.ch](https://logback.qos.ch/documentation.html)

# Chapter 1: Introduction

Logback-classic module requires the presence of **slf4j-api.jar** and **logback-core.jar** in addition to **logback-classic.jar** on the classpath.

---

In most cases, as far as logging is concerned, your classes will **only need to import SLF4J classes**. Thus, the vast majority, if not all, of your classes will use the SLF4J API and will be oblivious to the existence of logback.

---

By virtue of logback's ==**default configuration policy**==, when no default configuration file is found, logback will add a **ConsoleAppender** to the root logger.

---

Logback can report information about its internal state using a built-in status system. Important events occurring during logback's lifetime can be accessed through a component called **StatusManager**. For the time being, let us instruct logback to print its internal state by invoking the static print() method of the StatusPrinter class.  
```
package chapters.introduction;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.core.util.StatusPrinter;

public class HelloWorld2 {

  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld2");
    logger.debug("Hello world.");

    // print internal state
    LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
    StatusPrinter.print(lc);
  }
}
```

Note that in case of errors, logback will automatically print its internal state on the console.

---

# Chapter 2: Architecture

Logback-classic natively implements the SLF4J API so that you can readily switch back and forth between logback and other logging systems such as log4j or java.util.logging (JUL) introduced in JDK 1.4.

---

Logback is built upon three main classes: Logger, Appender and Layout. These three types of components work together to enable developers   
**Logger** - to log messages according to message type and level  
**Layout** - to control at runtime how these messages are formatted  
**Appender** - where they are reported.

---

Every single logger is attached to a **LoggerContext** which is responsible for manufacturing loggers as well as arranging them in a tree like hierarchy.

==**Named Hierarchy**==: For example, the logger named "com.foo" is a **parent** of the logger named "com.foo.Bar". Similarly, "java" is a parent of "java.util" and an **ancestor** of "java.util.Vector".

Loggers may be assigned levels: **TRACE**, **DEBUG**, **INFO**, **WARN** , **ERROR**, **ALL** and **OFF**  
==**If a given logger is not assigned a level, then it inherits one from its closest ancestor with an assigned level.**==  
To ensure that all loggers can eventually inherit a level, the root logger always has an assigned level. By default, this level is DEBUG.  
A logging request is said to be enabled if its level is higher than or equal to the effective level of its logger. Otherwise, the request is said to be disabled. (A log request of level p issued to a logger having an effective level q, is enabled if p >= q.)  

---

Configuration of the logback environment is typically done at application initialization. The preferred way is by reading a **configuration file**.

Logback makes it easy to **name loggers** by software component. This can be accomplished by instantiating a logger in each class, with the logger name equal to the fully qualified name of the class.  
However, as a developer, you are free to name loggers as you wish.

The **root logger** resides at the top of the logger hierarchy. Like every logger, it can be retrieved by its name, as follows:   
```Logger rootLogger = LoggerFactory.getLogger(org.slf4j.Logger.ROOT_LOGGER_NAME);```

---

Currently, **appenders** exist for the console, files, remote socket servers, to MySQL, PostgreSQL, Oracle and other databases, JMS, and remote UNIX Syslog daemons.

More than one appender can be attached to a logger.

The addAppender method adds an appender to a given logger. 

> ==**Appender Additivity**==  
**The output of a log statement of logger L will go to all the appenders in L and its ancestors.**  
However, if an ancestor of logger L, say P, has the **additivity flag** set to false, then L's output will be directed to all the appenders in L and its ancestors up to and including P but not the appenders in any of the ancestors of P.   
Loggers have their additivity flag set to true by default.

Logger Name | Attached Appenders | Additivity Flag | Output Targets | Comment
---|---|---|---|---|---
root | A1 | not applicable | A1 | Since the root logger stands at the top of the logger hierarchy, the additivity flag does not apply to it.
x | A-x1, A-x2 | TRUE | A1, A-x1, A-x2 | Appenders of "x" and of root.
x.y | none | TRUE | A1, A-x1, A-x2 | Appenders of "x" and of root.
x.y.z | A-xyz1 | TRUE | A1, A-x1, A-x2, A-xyz1 | Appenders of "x.y.z", "x" and of root.
security | A-sec | FALSE | A-sec | No appender accumulation since the additivity flag is set to?false. Only appender A-sec will be used.
security.access | none | TRUE | A-sec | Only appenders of "security" because the additivity flag in "security" is set to?false.

---

User can customize the output format accomplished by associationg a **layout** with an appender.  

The **PatternLayout**, part of the standard logback distribution, lets the user specify the output format according to conversion patterns similar to the C language printf function.

---

**Parameterized logging** -- mainly intended to **improve performance**

The following two lines will yield the exact same output. However, in case of a disabled logging statement, the second variant will outperform the first variant by a factor of at least 30.
```
logger.debug("The new entry is "+entry+".");
logger.debug("The new entry is {}.", entry);
```

---
The steps that the logback framework takes when the user invokes a logger's printing method:
![image](https://logback.qos.ch/manual/images/chapters/architecture/underTheHoodSequence2.gif)

# Chapter 3: Logback configuration

Logback can be configured either **programmatically** or with a configuration script expressed in **XML** or **Groovy** format.

---

the initialization steps that logback follows to try to configure itself:  
1. Logback tries to find a file called **logback-test.xml** in the classpath.
2. If no such file is found, logback tries to find a file called **logback.groovy** in the classpath.
3. If no such file is found, it checks for the file **logback.xml** in the classpath..
4. If no such file is found, service-provider loading facility (introduced in JDK 1.6) is used to resolve the implementation of com.qos.logback.classic.spi.Configurator interface by looking up the file META-INF\services\ch.qos.logback.classic.spi.Configurator in the class path. Its contents should specify the fully qualified class name of the desired Configurator implementation.
5. If none of the above succeeds, logback configures itself automatically using the **BasicConfigurator** which will cause logging output to be directed to the console. (ConsoleAppender/RootLogger=DEBUG/PatternLayoutEncoder=%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n)

If you are using Maven and if you place the logback-test.xml under the src/test/resources folder, Maven will ensure that it won't be included in the artifact produced. Thus, you can use a different configuration file, namely **logback-test.xml during testing**, and another file, namely, **logback.xml, in production**.

---

Example: Basic configuration file (logback-examples/src/main/resources/chapters/configuration/sample0.xml)
```
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

---

## STATUS OUTPUT

If warning or errors occur during the parsing of the configuration file, logback will automatically print its internal status messages on the console.

Instead of invoking StatusPrinter programmatically from your code, you can instruct the configuration file to dump status data, even in the absence of errors. To achieve this, you need to **set the debug attribute of the configuration element**. Please note that this debug attribute relates only to the status data.
```
<configuration debug="true"> 
  ...
</configuration>
```

**FORCING STATUS OUTPUT** In the absence of status messages, **tracking down a rogue logback.xml configuration file** can be difficult, especially in production where the application source cannot be easily modified. To help identify the location of a rogue configuration file, you can set a StatusListener via the "logback.statusListenerClass" system property (defined below) to force output of status messages. The "logback.statusListenerClass" system property can also be used to silence output automatically generated in case of errors.
```
<configuration>
  <statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener" />  

  ... the rest of the configuration file  
</configuration>
```

```
   LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory(); 
   StatusManager statusManager = lc.getStatusManager();
   OnConsoleStatusListener onConsoleListener = new OnConsoleStatusListener();
   statusManager.add(onConsoleListener);
```

```
java -Dlogback.statusListenerClass=ch.qos.logback.core.status.OnConsoleStatusListener ...
```

Enabling output of status data, either via the debug attribute or equivalently by installing OnConsoleStatusListener, will go a long way in helping you diagnose logback issues. As such, **enabling logback status data is very highly recommended and should be considered as a recourse of first resort**.

---

## ViewStatusMessagesServlet

Logback-classic ships with a servlet called ViewStatusMessagesServlet. This servlet prints the contents of the StatusManager associated with the current LoggerContext as an HTML table.
```
  <servlet>
    <servlet-name>ViewStatusMessages</servlet-name>
    <servlet-class>ch.qos.logback.classic.ViewStatusMessagesServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>ViewStatusMessages</servlet-name>
    <url-pattern>/lbClassicStatus</url-pattern>
  </servlet-mapping>
```
The ViewStatusMessages servlet will be viewable at the URL http://host/yourWebapp/lbClassicStatus

---

## specify configuration file location

You may **specify the location of the default configuration file** with a system property named "logback.configurationFile". The value of this property can be a URL, a resource on the class path or a path to a file external to the application.
```
java -Dlogback.configurationFile=/path/to/config.xml chapters.configuration.MyApp1
```
Note that the file extension must be ".xml" or ".groovy". Other extensions are ignored.

---

## reload configuration file on changed

If instructed to do so, logback-classic will scan for changes in its configuration file and automatically reconfigure itself when the configuration file changes.
```
<configuration scan="true"> 
  ... 
</configuration> 
```
By default, the configuration file will be scanned for changes once every minute. You can specify a different scanning period by setting the scanPeriod attribute of the <configuration> element. Values can be specified in units of milliseconds, seconds, minutes or hours.
```
<configuration scan="true" scanPeriod="30 seconds" > 
  ...
</configuration>
```

---

## show exception stack packageing data

```
<configuration packagingData="true">
  ...
</configuration>
```
```
LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
  lc.setPackagingDataEnabled(true);
```

---

## Configuration file syntax

The very basic structure of the configuration file can be described as, ```<configuration>``` element, containing zero or more ```<appender>``` elements, followed by zero or more ```<logger>``` elements, followed by at most one ```<root>``` element.
![](https://logback.qos.ch/manual/images/chapters/configuration/basicSyntax.png)

---

### logger

You can configure the levels of as many loggers as you wish.

**attribute**  
1. name(mandatory)  
2. level(optional) = TRACE, DEBUG, INFO, WARN, ERROR, ALL or OFF / INHERITED(NULL)  
3. additivity(optional) = true or false  

**element**  
1. <appender-ref>: 0~N

### root logger

**attribute**  
1. ~~name: named as "ROOT"~~  
2. level(optional) = TRACE, DEBUG(default), INFO, WARN, ERROR, ALL or OFF ~~/ INHERITED(NULL)~~  
3. ~~additivity(optional) = true or false~~  

**element**  
1. <appender-ref>: 0~N

### appender

**attribute**  
1. name(mandatory)  
2. class(mandatory)

**element**  
1. <layout>: 0/1
2. <encoder>: 0~N
3. <filter>: 0~N
4. any number of elements corresponding to JavaBean properties of the appender class

![](https://logback.qos.ch/manual/images/chapters/configuration/appenderSyntax.png)

---

## Setting the context name
Setting the context name is a simple and straightforward method in order to distinguish between multiple applications logging to the same target.
```
<configuration>
  <contextName>myAppName</contextName>
  ...
</configuration>
```

---

## Variable substitution

### Defining variables

variables can be defined **within the configuration file** itself, in an **external file**, in an **external resource** or even **computed** and defined **on the fly**.

automatically defined variables: **HOSTNAME** and **CONTEXT_NAME**  

For historical reasons, the XML element for defining variables is ```<property>``` although in logback 1.0.7 and later the element ```<variable>``` can be used interchangeably.

```
<configuration>

  <property name="USER_HOME" value="/home/sebastien" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>${USER_HOME}/myApp.log</file>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

```
java -DUSER_HOME="/home/sebastien" MyApp2
```

```
<configuration>

  <property file="src/main/java/chapters/configuration/variables1.properties" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
     <file>${USER_HOME}/myApp.log</file>
     <encoder>
       <pattern>%msg%n</pattern>
     </encoder>
   </appender>

   <root level="debug">
     <appender-ref ref="FILE" />
   </root>
</configuration>
```

```
<configuration>

  <property resource="resource1.properties" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
     <file>${USER_HOME}/myApp.log</file>
     <encoder>
       <pattern>%msg%n</pattern>
     </encoder>
   </appender>

   <root level="debug">
     <appender-ref ref="FILE" />
   </root>
</configuration>
```

### Scopes

During substitution, properties are looked up in the local scope first, in the context scope second, in the system properties scope third, and in the OS environment fourth and last.

**local SCOPE**(default) A property with local scope exists from the point of its definition in a configuration file until the end of interpretation/execution of said configuration file. As a corollary, each time a configuration file is parsed and executed, variables in local scope are defined anew.

**context SCOPE** A property with context scope is inserted into the context and lasts as long as the context or until it is cleared. Once defined, a property in context scope is part of the context. As such, it is available in all logging events, including those sent to remote hosts via serialization.

**system SCOPE** A property with system scope is inserted into the JVM's system properties and lasts as long as the JVM or until it is cleared.

```
<configuration>

  <property scope="context" name="nodeId" value="firstNode" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>/opt/${nodeId}/myApp.log</file>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

### Default values for variables
```${aName:-golden}```

## Conditional processing of configuration files
```
   <!-- if-then form -->
   <if condition="some conditional expression">
    <then>
      ...
    </then>
  </if>
  
  <!-- if-then-else form -->
  <if condition="some conditional expression">
    <then>
      ...
    </then>
    <else>
      ...
    </else>    
  </if>
```

## File inclusion
```
<configuration>
  <include file="src/main/java/chapters/configuration/includedConfig.xml"/>
  <!-- <include resource="includedConfig.xml"/> -->
  <!-- <include url="http://some.host.com/includedConfig.xml"/> -->

  <root level="DEBUG">
    <appender-ref ref="includedConsole" />
  </root>

</configuration>
```

# Chapter 4: Appenders

![](https://logback.qos.ch/manual/images/chapters/appenders/appenderClassDiagram.jpg)

## ConsoleAppender

## FileAppender

## RollingFileAppender

## SocketAppender and SSLSocketAppender

## ServerSocketAppender and SSLServerSocketAppender

## SMTPAppender

## DBAppender

## SyslogAppender

## SiftingAppender

## Writing your own Appender

# Chapter 5: Encoders

# Chapter 6: Layouts

# Chapter 7: Filters

# Chapter 8: Mapped Diagnostic

# Chapter 9: Logging Separation

# Chapter 10: JMx Configurator

# Chapter 11: Joran

# Chapter 12: Groovy Configuration

# Chapter 13: Migration from log4j

# Chapter 14: Receivers

# Chapter 15: Using SSL