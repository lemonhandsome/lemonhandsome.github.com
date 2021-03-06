---
layout: post
title: "elk stack logstash"
description: ""
category: [bigdata]
tags: [elk, elasticsearch, logstash, kibana]
---
{% include JB/setup %}


### info

1. [logstash](https://www.elastic.co/products/logstash)

1. [downloads](https://www.elastic.co/downloads/logstash)

1. documentation

    1. [2.3(current)](https://www.elastic.co/guide/en/logstash/current/index.html)

    1. [5.0](https://www.elastic.co/guide/en/logstash/5.0/index.html)

1. install steps

    1. downloads

            #### AAA install with apt-get
            # 1. import pgp key
            # download and install the public signing key
            $ wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

            # 2. add the repository definition
            $ echo "deb https://packages.elastic.co/logstash/2.3/debian stable main" | \
            sudo tee -a /etc/apt/sources.list

            # 2.3 install
            $ sudo apt-get update && sudo apt-get install logstash

            #### BBB install from tar.gz
            $ curl -O https://download.elastic.co/logstash/logstash/logstash-5.0.0-alpha3.tar.gz
            $ tar zxvf logstash-5.0.0-alpha3.tar.gz

    1. prepare a `logstash.conf` config file

    1. run

            $ bin/logstash agent -f logstash.conf

1. config

    1. config file json format

            $ cat demo.conf
            # this is a comment
            input {
                ...
            }

            filter {
                ...
            }

            output {
                ...
            }

### docker-logstash

1. [git repo](https://github.com/gree2/hello-docker/tree/master/docker-elk/docker-logstash)

### processing pipeline

1. [input plugins](https://www.elastic.co/guide/en/logstash/2.3/input-plugins.html)

    1. file

    1. syslog

    1. redis

    1. [beats](https://www.elastic.co/downloads/beats/filebeat)

1. [filter plugins](https://www.elastic.co/guide/en/logstash/2.3/filter-plugins.html)

    1. grok

            parse and structure arbitrary text

    1. mutate

            transform: rename remove replace modify

    1. drop

    1. clone

    1. geoip

1. [output plugins](https://www.elastic.co/guide/en/logstash/2.3/output-plugins.html)

    1. elasticsearch

    1. file

    1. [graphite](http://graphite.wikidot.com/)

    1. statsd

1. [codec plugins](https://www.elastic.co/guide/en/logstash/2.3/codec-plugins.html)

    1. [json](https://www.elastic.co/guide/en/logstash/2.3/plugins-codecs-json.html)

    1. multiline
    
### demo

1. basic logstash pipeline

    1. test installation

            $ cd logstash-2.3.0
            $ bin/logstash -e 'input { stdin { } } output{ stdout{ } }'

    1. `-e` flag

             specify a configuration from command line

1. rubydebug

            $ bin/logstash -e 'input { stdin { } } output{ stdout { codec => rubydebug } }'
            hello

1. elasticsearch output

    1. command

            # if connect to another container
            # change `elasticsearch { host = localhost }`
            #     to `elasticsearch { hosts => ["192.168.99.100:9200"] }`
            $ bin/logstash -e 'input { stdin { } } output{ elasticsearch {  } }'

    1. check indexes

            $ curl http://localhost:9200/_search?pretty
            {
                "name" : "Ellie Phimster",
                "cluster_name" : "elasticsearch",
                "version" : {
                    "number" : "2.3.3",
                    "build_hash" : "218bdf10790eef486ff2c41a3df5cfa32dadcfde",
                    "build_timestamp" : "2016-05-17T15:40:04Z",
                    "build_snapshot" : false,
                    "lucene_version" : "5.5.0"
                },
                "tagline" : "You Know, for Search"
            }