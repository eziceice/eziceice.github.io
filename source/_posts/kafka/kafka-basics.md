---
title: Kafka
date: 2021-10-26 15:34:57
categories:
- event-processing
- distribute-system
---
## Basics

- Event: an event stream records the history of what has happened in the world as a sequence of events.
- Table: a table represents the state of the world.
- A stream provides immutable data.
- A table provides mutable data.
- Events are serialised when they are written to a topic and deserialised when they are read.
- Common serialisation formats used by Kafka clients are Apache Arvo, Protobuf and JSON.
- Kafka brokers don't know the type of event they stored, it's just raw bytes to them.
- Kafka topics are partitioned, meaning a topic is spread over a number of "buckets" located on different brokers.
- Event producers determine event partitioning, the default strategy will spread the events evenly across the available topic partitions.
- Each stream task in a consumer group could consume maximum one partition.
- Tables are partitioned too, each table only knows the things in its own stream task.
- Global tables are not partitioned. 

## Store

- There are two types of stores in Kafka:
    - Global store, present in all instances and has the same data.
    - Local store, unique per partition.
- Kafka Streams will perform repartitioning if the group by using a different key than the partitioning key.
- Repartitioning is basically just changing the key of the record to the chosen grouping key, and sending it to intermediate Kafka topic, so the events that have the same grouping key will end up on the same partition, hence the same instance.
- Stores are not stored entirely in RAM, they are stored on a disk leveraging RocksDB.
- To prevent data loss, Kafka Streams sends all update to the store value to the changelog topic. In case of failure, another instance will take over the task, and restore the database from this changelog topic. 
