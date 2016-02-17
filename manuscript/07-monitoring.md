-# Part 4 Monitoring and Instrumentation

# How to instrument applications?


## How does it work?

. Define MetricsRegistry which is a JVM wide singleton for metric data
. Metrics Reporter is associated with the Metrics Registry
. Application services are instrumented with DropWizard Metrics
. The reporter reads the data from registry at fixed defined interval and writes this data to the MongoDB datastore is the relevant collection

![Metrics MongoDB](images/MetricsMongoDB.png)


## Dependencies

The module has following compile time dependencies

* io.dropwizard.metrics:metrics-core:jar:3.1.2
* org.mongodb:mongo-java-driver:jar:3.1.1
* org.slf4j:slf4j-api:jar:1.7.13

## Usage

Add the following dependency to your project's Maven POM file.


        <dependency>
            <groupId>io.github.aparnachaudhary</groupId>
            <artifactId>mongodb-metrics</artifactId>
            <version>${version.mongodb-reporter}</version>
        </dependency>


        // create metric registry
        MetricRegistry metricRegistry = new MetricRegistry();
        // register JVM metrics
        metricRegistry.register("jvm.attribute", new JvmAttributeGaugeSet());

        MongoDBReporter reporter = MongoDBReporter.forRegistry(metricRegistry)
                .serverAddresses(new ServerAddress[]{new ServerAddress("192.168.99.100", 32768)})
                .withDatabaseName("javasedemo")
                .prefixedWith("javase")
                .build();
        // Report metrics every 5 seconds
        reporter.start(5, TimeUnit.SECONDS);

        // register metric
        metricRegistry.counter("demo.counter").inc();

        // sleep for 10 seconds so that metric is reported to MongoDB store
        try {
            TimeUnit.SECONDS.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

Executing the above program creates two collections (counter and gauge) in javasedemo database.

*Counter collection:*


        /* 0 */
        {
            "_id" : ObjectId("565b3b7ec8f3530f3a327001"),
            "name" : "javase.demo.counter",
            "count" : 1,
            "timestamp" : ISODate("2015-11-29T17:53:02Z")
        }
        
        /* 1 */
        {
            "_id" : ObjectId("565b3b83c8f3530f3a327005"),
            "name" : "javase.demo.counter",
            "count" : 1,
            "timestamp" : ISODate("2015-11-29T17:53:07Z")
        }

*Gauge collection:*

Since we also registered the _JvmAttributeGaugeSet_; JVM metrics are registered in the _gauge_ collection.

        /* 0 */
        {
            "_id" : ObjectId("565b3b7ec8f3530f3a326ffe"),
            "name" : "javase.jvm.attribute.name",
            "value" : "69434@server.local",
            "timestamp" : ISODate("2015-11-29T17:53:02Z")
        }
        
        /* 1 */
        {
            "_id" : ObjectId("565b3b7ec8f3530f3a326fff"),
            "name" : "javase.jvm.attribute.uptime",
            "value" : 5711,
            "timestamp" : ISODate("2015-11-29T17:53:02Z")
        }
        
        /* 2 */
        {
            "_id" : ObjectId("565b3b7ec8f3530f3a327000"),
            "name" : "javase.jvm.attribute.vendor",
            "value" : "Oracle Corporation Java HotSpot(TM) 64-Bit Server VM 25.5-b02 (1.8)",
            "timestamp" : ISODate("2015-11-29T17:53:02Z")
        }
        
        /* 3 */
        {
            "_id" : ObjectId("565b3b83c8f3530f3a327002"),
            "name" : "javase.jvm.attribute.name",
            "value" : "69434@server.local",
            "timestamp" : ISODate("2015-11-29T17:53:07Z")
        }
        
        /* 4 */
        {
            "_id" : ObjectId("565b3b83c8f3530f3a327003"),
            "name" : "javase.jvm.attribute.uptime",
            "value" : 10561,
            "timestamp" : ISODate("2015-11-29T17:53:07Z")
        }
        
        /* 5 */
        {
            "_id" : ObjectId("565b3b83c8f3530f3a327004"),
            "name" : "javase.jvm.attribute.vendor",
            "value" : "Oracle Corporation Java HotSpot(TM) 64-Bit Server VM 25.5-b02 (1.8)",
            "timestamp" : ISODate("2015-11-29T17:53:07Z")
        }



# How to monitor log files?


## Logstash Set Up

    curl -O https://download.elasticsearch.org/logstash/logstash/logstash-1.4.0.beta1.tar.gz


    bin/logstash -e 'input { stdin { } } output { stdout { codec => rubydebug } }'

## Install Contrib package

    wget http://download.elasticsearch.org/logstash/logstash/logstash-contrib-1.4.0.beta1.tar.gz
    tar zxf ~/logstash-contrib-1.4.0.beta1.tar.gz

## Start logstash

    bin/logstash --debug -f logstash-mongodb.conf

    input {
      stdin { 
        # A type is a label applied to an event. It is used later with filters
        # to restrict what filters are run against each event.
        type => "human"
      } 
    }

    output {
      # Print each event to stdout.
      stdout {
        # Enabling 'rubydebug' codec on the stdout output will make logstash
        # pretty-print the entire event as something similar to a JSON representation.
        codec => rubydebug
      }
  
      # You can have multiple outputs. All events generally to all outputs.
      # Use mongodb to store logstash event.
      mongodb {
        uri => "mongodb://localhost"
        database => "logstash"
        collection => "logs" 
      }
    }

## MongoDB Output

    /* 0 */
    {
        "_id" : ObjectId("531234abc2e6e71e15acf93c"),
        "message" : "Aparna",
        "@version" : "1",
        "@timestamp" : "\"2014-03-01T19:27:39.236Z\"",
        "type" : "human",
        "host" : "server.local"
    }

## Wildfly Server Log File

    input {
      # WildFly server log file
      file {
      	path => "/Users/Aparna/Development/Tools/JBoss/wildfly-8.0.0.CR1/standalone/log/server.log"
      	start_position => "beginning"
        type => "server"
        format => "plain"
      }
    }

    filter {
      # combine stacktraces into single line
      multiline {
        type => "server"
        pattern => "(^.+Exception: .+)|(^\s+at .+)|(^\s+... \d+ more)|(^\s*Caused by:.+)"
        what => "previous"
      }
    }

# HTTP JSON Monitoring

# JMX Monitoring


