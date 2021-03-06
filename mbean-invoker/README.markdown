# JMX MBean Invoker #
This vcloud component can be installed into your Tomcat 6.0 server as a
Listener. It will consume events on the queue you define and invoke the
given JMX MBean, JSON-serializing the result.

#### Dependencies ####
* RabbitMQ AMQP Java client libraries (ver 1.7).
* Jackson JSON Parser/Generator (ver 1.5).

This allows remote management software to unobtrusively, and with minimal
system overhead, monitor the JVMs of processes running in the cloud without
resorting to expensive and inefficient continuous-polling scenarios.

Here's an example snippet from my server.xml file:

<pre><code>&lt;Listener className="com.jbrisbin.vcloud.mbean.CloudInvokerListener"
          instanceName="instance.replyTo"
          eventsExchange="vcloud.events"
          mbeanEventsExchange="vcloud.events.mbean"
          mbeanEventsQueue="mbean.events.instance.replyTo"
          mbeanEventsRoutingKey="tcserver.instance.replyTo"/&gt;
</code></pre>

There's a test file written in my RabbitMQ Groovy DSL to test this:

<pre><code>mq.exchange(name: "vcloud.events.mbean") {
  queue(routingKey: "tcserver.instance.replyTo") {
    println "Sending messages to remote mbean..."
    publish body: {msg, out -&gt;
      msg.properties.replyTo = "mbean.response"
      out.write('{ "mbean": "java.lang:type=Runtime", "attribute": "VmName" }'.bytes)
      out.flush()
    }

    publish body: {msg, out ->
      msg.properties.replyTo = "mbean.response"
      out.write('{ "mbean": "java.lang:type=Memory", "attribute": "HeapMemoryUsage" }'.bytes)
      out.flush()
    }
  }
}
</code></pre>

Sending a message to this listener using the RabbitMQ Groovy DSL results
in the following:

<pre><code>Sending messages to remote mbean...
Response: {"VmName":"Java HotSpot(TM) 64-Bit Server VM"}
Response: {"committed":83230720,"init":0,"max":85393408,"used":8968160}
</code></pre>