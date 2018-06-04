

##Title: "Streaming Data With Kafka"

##Author: "Harby Ariza"


##
##

Installing Kafka on Centos 7 OS:

First go the url below check for any prerequites. Then download Apache Kafka and install it under **/opt/kafka**. 

https://www.vultr.com/docs/how-to-install-apache-kafka-on-centos-7

Just one more thing , at the time I downloaded kafka I used the command below because the version in the url above is not available anymore:

##
```
 wget https://www-us.apache.org/dist/kafka/0.11.0.2/kafka_2.11-0.11.0.2.tgz 
 
 ```
  
Unzip the file and copy onto the **/opt** directory. Then rename the directory as shown below :  
##
```
tar -xvf kafka_2.11-0.11.0.2.tgz 
mv kafka_2.11-0.11.0.2 /opt
mv kafka_2.11-0.11.0.2 kafka
```


After you done installing kafka you need to visit to the confluent site and download the open source version and install it under **/opt/confluent** as you will need this in order to execute the KSQL demo/testing. 

https://www.confluent.io/download/

Also if you want to have a quick look about what you can do and achieve with KSQL I suggest you to check the slides below:

https://www.confluent.io/kafka-summit-london18/unlocking-the-world-of-stream-processing-with-ksql-the-streaming-sql-engine-for-apache-kafka


Also if you need Datasets for testing I suggest you to visit the kaggle.com as there is a huge variety of datasets that can be used for testing purposes.

https://www.kaggle.com/

After the installations are done then bring up the zookeeper:
##
```
cd /opt/kafka/bin
root@localhost:/opt/kafka/bin 
./zookeeper-server-start.sh -daemon /op/kafka/config/zookeeper.properties
```

Followed by kafka broker:
##
```
./kafka-server-start.sh -daemon /opt/kafka/config/server.properties

```

Then bring up the ksql server:
##
```
cd /opt/confluent/confluent-4.1.0/bin
root@localhost:/opt/confluent/confluent-4.1.0/bin
./ksql-server-start -daemon /opt/confluent/confluent-4.1.0/etc/ksql/ksql-server.properties
```

Now that we have the Zookeeper , Kafka and the Ksql server runing then we start by creating a new topics. I will need the bank_details and 
the bank_details_python topics:
##
```
cd /opt/kafka/bin
root@localhost:/opt/kafka/bin 
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic bank_details
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic bank_details_python

```

If you want to list the new topics:

```
./kafka-topics.sh --list --zookeeper localhost:2181

```
Now we invoke the kafka producer console executing the following command:
```
tail -f /home/hariza/data/bank_new.csv | ./kafka-console-producer.sh --broker-list localhost:9092 --topic bank_details

```
Then on a new terminal invoke the kafka consumer console  and you should be able to see data coming through:

```
cd /opt/kafka/bin
root@localhost:/opt/kafka/bin 
./kafka-console-consumer.sh --zookeeper localhost:2181 --topic bank_details
root@localhost:/opt/kafka/bin $ ./kafka-console-consumer.sh --zookeeper localhost:2181 --topic bank_details 
Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].

```



  Now we want to start appending some data into the bank_new.csv file so in order to do so we execute the DataGenBankDetails.py program.   


The code below is just a little program that reads a file and appends it onto another one in this case the file we are executing the **tail** command **( tail -f /home/hariza/data/bank_new.csv ) **.  Also please bear in mind that for this demostration the program is opening and closing the file so you can see the updates coming more often into the kafka consumer. The file bank_details.csv was downloaded from the kaggle.com website.:

##
##
```
#!/usr/bin/env python

 ###  read data from a file and append it to another one #####

import fileinput
import time
import os
import io

fhw = open('/home/hariza/data/bank_new.csv','a')
 
for line in fileinput.input('/home/hariza/data/bank_details.csv'):
        print (line)
        fhw.writelines(line)
        time.sleep(1)
        fhw.close() 
        fhw = open('bank_new.csv','a')
 
```
##
##
```
cd /home/hariza/data
./DataGenBankDetails.py
```

So once this program is running you can see the data displayed on the consumer console. 

##
```
root@localhost:/opt/kafka/bin $ ./kafka-console-consumer.sh --zookeeper localhost:2181 --topic bank_details 
Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].
1,30,unemployed,married,primary,no,1787,no,no,cellular,19,oct,79,1,-1,0,unknown,no
2,33,services,married,secondary,no,4789,yes,yes,cellular,11,may,220,1,339,4,failure,no
3,35,management,single,tertiary,no,1350,yes,no,cellular,16,apr,185,1,330,1,failure,no
4,30,management,married,tertiary,no,1476,yes,yes,unknown,3,jun,199,4,-1,0,unknown,no

```

  Now we are going to start sending data direclty to the **bank_details_python** topic by executing the **DataGenKafkaProducer.py** program. So the program basically reads the bank_details.txt file and push line by line to the KafkaProducer. Also please bear in mind that you need to check if you have the kafka-python package install otherwise you can just execute the **'yum install pip'** and **'pip install kafka-python'** commands and then the program should run just fine.

##
##
```
#!/usr/bin/env python

   ## DataGenKafkaProducer.py 
   
from kafka import KafkaProducer
import fileinput
import time


KAFKA_BROKERS='localhost:9092'
p = KafkaProducer(bootstrap_servers=KAFKA_BROKERS)
for line in fileinput.input('bank_details.txt'):
        print (line)
	p.send('bank_details_python',value=line)
        time.sleep(2)
        

```


Now we want to view the stream of data coming through so this where we invoke the Ksql console:

##
```
LOG_DIR=/tmp/ksql_logs ksql


hariza@localhost:/opt/confluent $ LOG_DIR=/tmp/ksql_logs ksql
                  
                  ===========================================
                  =        _  __ _____  ____  _             =
                  =       | |/ // ____|/ __ \| |            =
                  =       | ' /| (___ | |  | | |            =
                  =       |  <  \___ \| |  | | |            =
                  =       | . \ ____) | |__| | |____        =
                  =       |_|\_\_____/ \___\_\______|       =
                  =                                         =
                  =  Streaming SQL Engine for Apache Kafka® =
                  ===========================================

Copyright 2017 Confluent Inc.

CLI v4.1.0, Server v4.1.0 located at http://localhost:8088

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!

ksql> 
```

##
##

```
ksql> 
ksql> 
ksql> show topics;

 Kafka Topic                            | Registered | Partitions | Partition Replicas | Consumers | Consumer Groups 
---------------------------------------------------------------------------------------------------------------------
 _confluent-ksql-default__command_topic | true       | 1          | 1                  | 0         | 0               
 bank_details                           | false      | 1          | 1                  | 0         | 0               
 bank_details_python                    | true       | 1          | 1                  | 0         | 0               
---------------------------------------------------------------------------------------------------------------------
ksql> 

```
Now with the following code I am creating a new stream called **bank_stream_from_python**. I have specified the data types for each column after browsing the bank_details.txt file.

```
ksql> 
ksql> create stream bank_stream_from_python (s_no int ,age int ,job string,marital string,education string,default string,balance bigint,housing string ,loan string ,contact string,day string,month string,duration int,campaign int,pdays int,previous int ,poutcome string,y string) with (kafka_topic='bank_details_python',value_format='delimited',key='s_no');

 Message        
----------------
 Stream created 
----------------
ksql> 
```

Then execute a query to the new stream. For this demo I have repeated many lines of the file so you can see as the data arrives into the stream:

##
```
ksql> select * from bank_stream_from_python where s_no =5;
1527920169417 | null | 5 | 59 | blue-collar | married | secondary | no | 0 | yes | no | unknown | 5 | may | 226 | 1 | -1 | 0 | unknown | no
1527920189438 | null | 5 | 59 | blue-collar | married | secondary | no | 0 | yes | no | unknown | 5 | may | 226 | 1 | -1 | 0 | unknown | no
1527920209457 | null | 5 | 59 | blue-collar | married | secondary | no | 0 | yes | no | unknown | 5 | may | 226 | 1 | -1 | 0 | unknown | no
```

Let's create a table so we can perform aggregations using customers by age:

##
```
sql> 
ksql> 
ksql> create table customer_by_age as select age,count(*) from bank_stream_from_python group by age;

 Message                   
---------------------------
 Table created and running 
---------------------------
ksql> 
```

So now you can see the data aggregation happening on the fly as more data comes in: 

##
```
ksql> select * from customer_by_age;
1527921500849 | 43 | 43 | 3
1527921502851 | 30 | 30 | 5
1527921504852 | 33 | 33 | 3
1527921506853 | 35 | 35 | 6
1527921508856 | 30 | 30 | 6
1527921510859 | 59 | 59 | 4
1527921512861 | 35 | 35 | 7
1527921514864 | 36 | 36 | 4
1527921516867 | 39 | 39 | 4
```


Let's describe the stream **bank_stream_from_python** :
##
```
ksql> describe bank_stream_from_python;

 Field     | Type                      
---------------------------------------
 ROWTIME   | BIGINT           (system) 
 ROWKEY    | VARCHAR(STRING)  (system) 
 S_NO      | INTEGER                   
 AGE       | INTEGER                   
 JOB       | VARCHAR(STRING)           
 MARITAL   | VARCHAR(STRING)           
 EDUCATION | VARCHAR(STRING)           
 DEFAULT   | VARCHAR(STRING)           
 BALANCE   | BIGINT                    
 HOUSING   | VARCHAR(STRING)           
 LOAN      | VARCHAR(STRING)           
 CONTACT   | VARCHAR(STRING)           
 DAY       | VARCHAR(STRING)           
 MONTH     | VARCHAR(STRING)           
 DURATION  | INTEGER                   
 CAMPAIGN  | INTEGER                   
 PDAYS     | INTEGER                   
 PREVIOUS  | INTEGER                   
 POUTCOME  | VARCHAR(STRING)           
 Y         | VARCHAR(STRING)           
---------------------------------------
For runtime statistics and query details run: DESCRIBE EXTENDED <Stream,Table>;
ksql> 
```


And do the same for the new table **customer_by_age**
##
```
ksql> describe extended customer_by_age;

Type                 : TABLE
Key field            : BANK_STREAM_FROM_PYTHON.AGE
Timestamp field      : Not set - using <ROWTIME>
Key format           : STRING
Value format         : DELIMITED
Kafka output topic   : CUSTOMER_BY_AGE (partitions: 4, replication: 1)

 Field      | Type                      
----------------------------------------
 ROWTIME    | BIGINT           (system) 
 ROWKEY     | VARCHAR(STRING)  (system) 
 AGE        | INTEGER          (key)    
 KSQL_COL_1 | BIGINT                    
----------------------------------------

Queries that write into this TABLE
-----------------------------------
id:CTAS_CUSTOMER_BY_AGE - create table customer_by_age as select age,count(*) from bank_stream_from_python group by age;

For query topology and execution plan please run: EXPLAIN <QueryId>

Local runtime statistics
------------------------
messages-per-sec:         0   total-messages:       320     last-message: 2/06/18 4:48:09 PM
 failed-messages:         0 failed-messages-per-sec:         0      last-failed:       n/a
(Statistics of the local KSQL server interaction with the Kafka topic CUSTOMER_BY_AGE)
```



Also you can create a persistent stream based on a existing stream using the **where** condition:

##
```
ksql> 
ksql> 
ksql> 
ksql> create stream bank_stream_single as select * from bank_stream_from_python where marital = 'single'; 

 Message                    
----------------------------
 Stream created and running 
----------------------------
ksql> select * from bank_stream_single limit 5;
1528093123000 | null | 3 | 35 | management | single | tertiary | no | 1350 | yes | no | cellular | 16 | apr | 185 | 1 | 330 | 1 | failure | no
1528093129006 | null | 6 | 35 | management | single | tertiary | no | 747 | no | no | cellular | 23 | feb | 141 | 2 | 176 | 3 | failure | no
1528093143025 | null | 3 | 35 | management | single | tertiary | no | 1350 | yes | no | cellular | 16 | apr | 185 | 1 | 330 | 1 | failure | no
1528093149031 | null | 6 | 35 | management | single | tertiary | no | 747 | no | no | cellular | 23 | feb | 141 | 2 | 176 | 3 | failure | no
```


