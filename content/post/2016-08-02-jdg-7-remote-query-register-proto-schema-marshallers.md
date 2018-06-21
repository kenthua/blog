---
author: Kent Hua
comments: true
date: "2016-08-02T17:00:00Z"
tags: [jdg, jboss, data grid, 7.0, 7, remote query, protobuf, register, marshaller, schema]
title: JBoss Data Grid 7.0 - Remote Query via Hot Rod Java, Register protobuf marshaller and schema
---

JDG 7.0 is out and provides some cool new features.  Such as a web console, which is only available in the new domain mode.  Prior to JDG 7.0, in Remote Client-Server mode, you ran individual clustered standalone instances.  In 7.0, it leverages the underlying EAP domain mode management.  The console allows you to manage nodes and caches, as well as see some jvm stats and cache stats.  

So what's this article about?  If you used JDG v6.x with the java Hot Rod client and Remote Query, you had to register your protobuf marshaller and schema with the internal `___protobuf_metadata` cache.  This worked whether you are running it via localhost or a remote address.  Well in v7.0 it's more secure and you get this error when trying to access it remotely:

`org.infinispan.client.hotrod.exceptions.HotRodClientException:Request for messageId=3 returned server error (status=0x84): org.infinispan.server.hotrod.RequestParsingException: Remote requests are allowed to protected caches only over loopback or if authorization is enabled. Do no send remote requests to cache '___protobuf_metadata'`

So what are we to do?  We enable security of course, which adds a whole new slew of excitement and troubleshooting.  This could be that JDG security is fairly new to me.  Most of this can be done with the new domain UI, but I still had some issues when enabling security at the cache manager level and the cache itself.  However, I found the answer below, which is defining a special role.  Follow the BEGIN and END XML comments in each section.

{{< highlight xml >}}
...
<subsystem xmlns="urn:infinispan:server:core:8.3">
    <cache-container name="clustered" default-cache="default" statistics="true">
        <transport lock-timeout="60000"/>
        <!-- BEGIN -->
        <security>
            <authorization>
                <identity-role-mapper/>
                <role name="role1" permissions="READ WRITE EXEC LISTEN ADMIN"/>
                <role name="role2" permissions="READ WRITE BULK_READ"/>
            </authorization>
        </security>
        <!-- END -->
        <global-state/>
...
        <distributed-cache-configuration name="test-default" owners="2" segments="20" mode="ASYNC" remote-timeout="30000" start="EAGER">
            <locking striping="false" acquire-timeout="30000" concurrency-level="1000"/>
            <transaction mode="NONE"/>
            <!-- BEGIN -->
            <security>
                <authorization enabled="true" roles="role1 role2"/>
            </security>
            <!-- END -->
        </distributed-cache-configuration>
        <distributed-cache name="data" configuration="test-default"/>
    </cache-container>

...

<subsystem xmlns="urn:infinispan:server:endpoint:8.0">
    <hotrod-connector cache-container="clustered" socket-binding="hotrod">
        <topology-state-transfer lazy-retrieval="false" lock-timeout="1000" replication-timeout="5000"/>
        <!-- BEGIN -->
        <authentication security-realm="ApplicationRealm">
            <sasl mechanisms="DIGEST-MD5" qop="auth" server-name="data-server">
                <policy>
                    <no-anonymous value="true"/>
                </policy>
                <property name="com.sun.security.sasl.digest.utf8">
                    true
                </property>
            </sasl>
        </authentication>
        <!-- END -->
    </hotrod-connector>
...    
{{< / highlight >}}

When defining your application-roles.properties or your LDAP directory, this is the most important part `___schema_manager`.  I found this digging through the infinispan [test code](https://github.com/infinispan/infinispan/blob/master/server/integration/testsuite/src/test/resources/application-roles.properties#L2) and [documentation](https://github.com/infinispan/infinispan/blob/master/documentation/src/main/asciidoc/user_guide/server_protocols_hotrod.adoc).

{{< highlight shell >}}
user1=role1,___schema_manager
{{< / highlight >}}

Why is `___schema_manager` important?  Without this you will get the error below.

{{< highlight java >}}

Exception in thread "main" org.infinispan.client.hotrod.exceptions.HotRodClientException:Request for messageId=7 returned server error (status=0x85): java.lang.SecurityException: ISPN000287: Unauthorized access: subject 'Subject with principal(s): [SimpleUserPrincipal [name=user1], InetAddressPrincipal [address=192.168.50.196/192.168.50.196], user1@ApplicationRealm, role1@ApplicationRealm, role1]' lacks 'WRITE' permission
	at org.infinispan.client.hotrod.impl.protocol.Codec20.checkForErrorsInResponseStatus(Codec20.java:343)
	at org.infinispan.client.hotrod.impl.protocol.Codec20.readPartialHeader(Codec20.java:132)
	at org.infinispan.client.hotrod.impl.protocol.Codec20.readHeader(Codec20.java:118)
	at org.infinispan.client.hotrod.impl.operations.HotRodOperation.readHeaderAndValidate(HotRodOperation.java:56)
	at org.infinispan.client.hotrod.impl.operations.AbstractKeyValueOperation.sendPutOperation(AbstractKeyValueOperation.java:56)
	at org.infinispan.client.hotrod.impl.operations.PutOperation.executeOperation(PutOperation.java:32)
	at org.infinispan.client.hotrod.impl.operations.RetryOnFailureOperation.execute(RetryOnFailureOperation.java:54)
	at org.infinispan.client.hotrod.impl.RemoteCacheImpl.put(RemoteCacheImpl.java:268)
	at org.infinispan.client.hotrod.impl.RemoteCacheSupport.put(RemoteCacheSupport.java:79)
    ...
{{< / highlight >}}

Below is the java code that we need to add to add security into the mix.  It was leveraged from the [jdg quickstarts](https://github.com/jboss-developer/jboss-jdg-quickstarts/blob/jdg-7.0.x/hotrod-secured/src/main/java/org/jboss/as/quickstarts/datagrid/hotrod/FootballManager.java#L198-L202) example.  You will also need a custom [LoginHandler](https://github.com/jboss-developer/jboss-jdg-quickstarts/blob/jdg-7.0.x/hotrod-secured/src/main/java/org/jboss/as/quickstarts/datagrid/hotrod/LoginHandler.java).

{{< highlight java >}}
...
ConfigurationBuilder builder = new ConfigurationBuilder();
builder.addServers(jdgProperty(JDG_HOSTS)).marshaller(
        new ProtoStreamMarshaller()); 

builder.security().authentication()
.serverName("data-server") //define server name, should be specified in XML configuration
.saslMechanism("DIGEST-MD5") // define SASL mechanism, in this example we use DIGEST with MD5 hash
.callbackHandler(new LoginHandler("user1", "password".toCharArray(), "ApplicationRealm")) // define login handler, implementation defined
.enable();
...
{{< / highlight >}}

--- 
  
Let's say we want to use the MBean method to register our protobuf schema files, which seems to only be available for standalone/standalone clustered configurations.  I can't seem to find the MBean for a domain configuration.  

`cacheContainerName` is `local` for standalone and `clustered` for standalone clustered.

This will use your `ManagementRealm` as it's connecting via JMX, so be sure to add the `___schema_manager` role to your user, in the `mgmt-groups.properties`

{{< highlight java >}}
package com.redhat.example.util;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.Reader;
import java.io.StringWriter;
import java.util.HashMap;
import java.util.Map;

import javax.management.MBeanServerConnection;
import javax.management.ObjectName;
import javax.management.remote.JMXConnector;
import javax.management.remote.JMXConnectorFactory;
import javax.management.remote.JMXServiceURL;

public class RegisterProto {
	private static final String PROTOBUF_DEFINITION_RESOURCE = "/data/entity.proto";
    
	public void registerViaJMX() throws Exception {

		String serverHost = "192.168.50.196";         // The address of your JDG server
		int serverJmxPort = 9990;        // The JMX port of your server
		String cacheContainerName = "clustered"; // The name of your cache container
		String schemaFileName = PROTOBUF_DEFINITION_RESOURCE;     // The name of the schema file
		System.out.println(">> Read ProtoFile");
		String schemaFileContents = readResource(PROTOBUF_DEFINITION_RESOURCE); // The Protobuf schema file contents

		Map env = new HashMap();
		String[] creds = new String[2];
		creds[0] = "admin";
		creds[1] = "passw0rd!";
		env.put(JMXConnector.CREDENTIALS, creds);
		
		System.out.println(">> Get JMX Connection");
		JMXServiceURL jmxURL = new JMXServiceURL("service:jmx:remote+http://" + serverHost + ":" + serverJmxPort);
		JMXConnector jmxConnector = JMXConnectorFactory.connect(jmxURL, env);		
		
		System.out.println(">> Get MBean Server Connection");
		MBeanServerConnection jmxConnection = jmxConnector.getMBeanServerConnection();

		ObjectName protobufMetadataManagerObjName = 
				new ObjectName("jboss.datagrid-infinispan:type=RemoteQuery,name=" +				
						ObjectName.quote(cacheContainerName) + 
						",component=ProtobufMetadataManager");

		System.out.println(">> Invoke");
		jmxConnection.invoke(protobufMetadataManagerObjName, 
		                     "registerProtofile", 
		                     new Object[]{schemaFileName, schemaFileContents}, 
		                     new String[]{String.class.getName(), String.class.getName()});
		System.out.println(">> Close");		
		jmxConnector.close();
	}
	
	public static void main(String[] args) throws Exception {
		RegisterProto rp = new RegisterProto();
		rp.registerViaJMX();
	}

	private String readResource(String resourcePath) throws IOException {
		InputStream is = getClass().getResourceAsStream(resourcePath);
		try {
			final Reader reader = new InputStreamReader(is, "UTF-8");
			StringWriter writer = new StringWriter();
			char[] buf = new char[1024];
			int len;
			while ((len = reader.read(buf)) != -1) {
				writer.write(buf, 0, len);
			}
			return writer.toString();
		} finally {
			is.close();
		}
	}
}
{{< / highlight >}}

You will need this in your `pom.xml`
{{< highlight java >}}
...
<dependency>
    <groupId>org.jboss.remotingjmx</groupId>
    <artifactId>remoting-jmx</artifactId>
    <version>2.0.1.Final-redhat-1</version>
</dependency>
...
{{< / highlight >}}

You will still need to register your marshallers into the cache.

{{< highlight java >}}
...
SerializationContext ctx = ProtoStreamMarshaller
        .getSerializationContext(cacheManager);
ctx.registerProtoFiles(FileDescriptorSource
        .fromResources(PROTOBUF_DEFINITION_RESOURCE));
ctx.registerMarshaller(new EntityMarshaller());
...
{{< / highlight >}}
