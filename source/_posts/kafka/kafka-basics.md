---
title: Kafka Basics
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
