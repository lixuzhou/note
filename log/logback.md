[TOC]

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

Loggers may be assigned levels: **TRACE**, **DEBUG**, **INFO**, **WARN** , **ERROR** and **OFF**  
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