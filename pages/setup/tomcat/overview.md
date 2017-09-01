---
title: Install Apache Tomcat on the CAS servers
last_updated: September 1, 2017
sidebar: main_sidebar
permalink: setup_tomcat_overview.html
summary: Apache Tomcat will be used as the Java Servlet container for CAS. To ensure a best practices security configuration, the latest versions of Java, Tomcat, OpenSSL, and the Tomcat Native Library will be installed on the master build server and then distributed to the CAS servers.
---

{% include note.html content="CAS 5 is based on Spring Boot, which means that instead of building and installing an external servlet container, CAS can be deployed using an embedded servlet container and invoked with a `java -jar cas.war` style command. However, because we will be deploying multiple Java applications on the same set of servers, we will use the more traditional external container approach. This will avoid network port conflicts, allow us to make more efficient use of system resources, and enable us to manage our CAS installation in the same way we manage our other Java servlet-based applications." %}

CAS 5 requires Java 8 and a Java Servlet container that supports v3.1 or later of the Java Servlet specification. Apache Tomcat is the most commonly used container for CAS, so that's what we'll use for this deployment. Tomcat 8.0 was the first version to support v3.1 of the servlet specification, but this version has been superseded by Tomcat 8.5. Unfortunately, Red Hat does not, as of this writing, offer Tomcat 8.5 on RHEL 7, so it will have to be installed manually.

The *CAS Security Guide* warns that all communication with the CAS server must occur over a secure channel (i.e., Transport Layer Security) to prevent the disclosure of users' security credentials and/or CAS ticket-granting tickets. From a practical standpoint this means that all CAS URLs must use HTTPS, but it also means that all connections from the CAS server to the calling application must use HTTPS as well. Consequently, it's critical that Tomcat's TLS settings be configured in line with recognized best practices for cipher suite choice, key lengths, forward secrecy, and other parameters.

In the past, it has been difficult to configure Tomcat according to TLS best practices because of limitations in both Tomcat and the Java Secure Socket Extension (JSSE). Java 8 and Tomcat 8.5 have made significant improvements in this area, but installation of the optional Tomcat Native Library, which uses OpenSSL instead of Java cryptography libraries, is still the best way to ensure that a Tomcat installation scores an 'A' on the Qualys® SSL Labs SSL Server Test. The version of the Tomcat Native Library included with Tomcat 8.5 requires OpenSSL 1.0.2 or higher. Unfortunately, Red Hat does not, as of this writing, offer OpenSSL 1.0.2 or higher on RHEL 7, so it will also have to be installed manually.

## References

* [CAS 5: Security Guide][casdoc-sec-guide]
* [CAS 5: Servlet Container Configuration][casdoc-srvlt-cont]
* [Tomcat Wiki: TLS Cipher suite choice][tomcat-ciphers]

{% include reflinks.md %}
