# KSQL Step-by-Step Live Demo

This project shows simple commands and step-by-step guide to try about KSQL, the open source streaming SQL engine for Apache Kafka.

## Requirements
The code is developed and tested on Mac and Linux operating systems. As Kafka does not support and work well on Windows, this is not tested at all.

- [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) - newer version should also work
- [Confluent Platform 5.0 or newer](https://www.confluent.io/download/) (Open Source or Enterprise)


## Start server components

    confluent status

Start KSQL Server and its dependencies ZooKeeper and Kafka Broker. Schema Registry is also started, but only required if you use Avro.

    confluent start ksql-server

## Generate continous test data streams

    /Users/kai.waehner/confluent-5.2.0/bin/ksql-datagen quickstart=users format=json topic=users maxInterval=1000  propertiesFile=/Users/kai.waehner/confluent-5.2.0/etc/ksql/datagen.properties

    /Users/kai.waehner/confluent-5.2.0/bin/ksql-datagen quickstart=pageviews format=delimited topic=pageviews maxInterval=100 propertiesFile=/Users/kai.waehner/confluent-5.2.0/etc/ksql/datagen.properties

## Start KSQL CLI

    ksql

## First STREAM and SELECT Query
    SHOW TOPICS;
    PRINT 'pageviews' FROM BEGINNING;

    CREATE STREAM pageviews_original (viewtime bigint, userid varchar, pageid varchar) WITH (kafka_topic='pageviews', value_format='DELIMITED', key='userid');

    SHOW STREAMS;
    DESCRIBE pageviews_original;
    DESCRIBE EXTENDED pageviews_original;

    SELECT pageid, userid FROM pageviews_original LIMIT 10;

    SELECT * FROM pageviews_original; 

## KSQL Functions

    LIST FUNCTIONS;

    DESCRIBE FUNCTION MAX;

    SELECT pageid, userid, MAX(viewtime) FROM pageviews_original GROUP BY pageid, userid LIMIT 10;

You can also [create your own User Defined Functions (UDF)](https://www.confluent.io/blog/build-udf-udaf-ksql-5-0) easily.

## First TABLE

    CREATE TABLE users_original (registertime bigint, gender varchar, regionid varchar, userid varchar) WITH (kafka_topic='users', value_format='JSON', key = 'userid');

    SHOW TABLES;

    DESCRIBE users_original;

KSQL adds the implicit columns ROWTIME and ROWKEY to every stream and table, which represent the corresponding Kafka message timestamp and message key, respectively.

TABLE => You only see each user once (but then you also see continuous updates, i.e. new incoming events) -> Restart query, and you again see each one only once

Get most recent events and data of the table:

    SELECT * FROM users_original;

Consume table from beginning (including values which are overwritten):

    SET 'auto.offset.reset'='earliest';
    SELECT * FROM pageviews_original;
    SELECT * FROM users_original;  
    SET 'auto.offset.reset'='latest';

## OPTIONAL: Show non-KSQL Kafka client

    kafka-console-consumer --bootstrap-server localhost:9092 --topic users --from-beginning

    kafka-console-consumer --bootstrap-server localhost:9092 --topic pageviews --from-beginning

    kafka-topics --bootstrap-server localhost:9092 --list

    kafka-topics --bootstrap-server localhost:9092 --describe

## STREAM TABLE JOIN

    CREATE TABLE FEMALEUSERS AS SELECT * from users_original WHERE gender = 'FEMALE';

    PRINT 'FEMALEUSERS' FROM BEGINNING;

    SELECT * FROM users_original WHERE gender = 'MALE' LIMIT 3;

    CREATE STREAM pageviews_female AS SELECT users_original.userid AS userid, pageid, regionid, gender FROM pageviews_original LEFT JOIN users_original ON pageviews_original.userid = users_original.userid WHERE gender = 'FEMALE';

    SELECT * FROM pageviews_female LIMIT 3;

## AVRO for automatic inference of message structure

    /Users/kai.waehner/confluent-5.2.0/bin/ksql-datagen quickstart=ratings format=avro topic=ratings maxInterval=500

    PRINT 'ratings' FROM BEGINNING;

    CREATE STREAM ratings WITH (KAFKA_TOPIC='ratings',VALUE_FORMAT='AVRO');

    DESCRIBE ratings;
    DESCRIBE EXTENDED ratings;

    SELECT * FROM ratings;
    SELECT * FROM ratings WHERE stars <= 3;

## Stop services
KSQL CLI => Type 'Exit'

STOP ALL DATA GENERATORS via CTRL-C

    confluent destroy
    confluent status






