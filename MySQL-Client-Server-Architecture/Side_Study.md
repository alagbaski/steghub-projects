# Side Study Documentation

## Overview
This document outlines the side study topics to be explored, focusing on network diagnostic utilities (`ping` and `traceroute`) and basic SQL commands. The goal is to understand the usage and interpretation of `ping` and `traceroute`, and to refresh knowledge on essential SQL commands.

## Topics

### 1. Network Diagnostic Utilities

### Ping
**Purpose:** 
`ping` is a network utility used to test the reachability of a host on an IP network and to measure the round-trip time for messages sent from the originating host to a destination computer.

**Usage:**
- To check connectivity: `ping <hostname or IP address>`
- Common options:
  - `-c <count>`: Number of echo requests to send.
  - `-i <interval>`: Time interval between sending each packet.
  - `-t <ttl>`: Set the Time to Live.

**Interpreting Results:**
- **Packets transmitted:** Number of packets sent.
- **Packets received:** Number of packets received back.
- **Packet loss:** Percentage of packets lost.
- **Round-trip time:** Min/avg/max time for packets to reach the destination and return.

### Traceroute
**Purpose:**
`traceroute` is a network diagnostic tool used to track the pathway taken by a packet on an IP network from source to destination. It helps in identifying routing issues and understanding the network topology.

**Usage:**
- To trace the route: `traceroute <hostname or IP address>`
- Common options:
  - `-m <max_ttl>`: Maximum number of hops.
  - `-p <port>`: Base UDP port number used in probes.
  - `-n`: Print hop addresses numerically.

**Interpreting Results:**
- **Hops:** Steps or stages the packet takes from source to destination.
- **Time:** Time taken for each hop.
- **Hostnames/IPs:** Identifies each router in the path.

### 2. Basic SQL Commands

#### SHOW
**Purpose:** 
Displays information about databases, tables, columns, or status.

**Usage:**
- Show databases: `SHOW DATABASES;`
- Show tables: `SHOW TABLES;`
- Show columns in a table: `SHOW COLUMNS FROM <table_name>;`

#### CREATE
**Purpose:**
Creates databases, tables, indexes, or other database objects.

**Usage:**
- Create a database: `CREATE DATABASE <database_name>;`
- Create a table:
  ```sql
  CREATE TABLE <table_name> (
      column1 datatype,
      column2 datatype,
      ...
  );
  ```
#### DROP
**Purpose:**
Deletes databases, tables, or other database objects.

Usage:
```sql
DROP DATABASE <database_name>;
DROP TABLE <table_name>;
```

#### SELECT
**Purpose:**
Retrieves data from one or more tables.

Usage:

```sql
SELECT * FROM <table_name>;
SELECT column1, column2 FROM <table_name>;
SELECT column1 FROM <table_name> WHERE condition;
```
#### INSERT
**Purpose:**
Inserts new data into a table.

Usage:

- Insert into all columns:  
  
  ```sql
  INSERT INTO <table_name> VALUES (value1, value2, ...);
  ```

- Insert into specific columns:

  ```sql
  INSERT INTO <table_name> (column1, column2) VALUES (value1, value2);
  ```


### Conclusion
By exploring the functionalities of `ping` and `traceroute`, and refreshing basic `SQL commands`, one can effectively diagnose network issues and manage SQL databases. This knowledge is fundamental for network administrators and database managers.