-# Part 4 Monitoring

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


