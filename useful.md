sudo apt-get install python3-gdbm
curl https://bootstrap.pypa.io/get-pip.py | python3.9
pip install urllib3==1.26.6
/usr/local/hbase/bin/hbase thrift
curl https://bootstrap.pypa.io/get-pip.py | python3.9


# Cheatsheet

PORTY:
- 50070: HDFS
- 8088: Apache Hadoop
- 9443: NiFi
- 18080: Apache Spark
- 8081: Apache Flink

# Connecting to VM

1. Otwarcie tunelu ssh
```bash
ssh -L 2222:vl26.mini.pw.edu.pl:22 galkowskim@ssh.mini.pw.edu.pl
```

2. Example port forwarding:
```bash
ssh -L 8088:vl26.mini.pw.edu.pl:8088 galkowskim@ssh.mini.pw.edu.pl
```

3. Połączenie z VM'ka
```bash
ssh -i Desktop\big_data\private_key vagrant@localhost -p 2222
```

4. Run all services
```bash
sudo ./scripts/bootstrap.sh
```

# HDFS commands

- list files in HDFS
```bash
hdfs dfs -ls /
```

- check free disk space
```bash
hdfs dfs -df -h
```

- check the size of each directory
```bash
hdfs dfs -du -h /
```

- make directory
```bash
hdfs dfs -mkdir dir_path
```

- put file to HDFS
```bash
hdfs dfs -put file_path hdfs_path
```

- copy from local to HDFS
```bash
hdfs dfs -copyFromLocal file_path hdfs_path
```

- get file from HDFS
```bash
hdfs dfs -get hdfs_path file_path
```

- copy to local from HDFS
```bash
hdfs dfs -copyToLocal hdfs_path file_path
```

- cat file
```bash
hdfs dfs -cat hdfs_path
```

- mv file
```bash
hdfs dfs -mv hdfs_path hdfs_path
```

- cp file
```bash
hdfs dfs -cp hdfs_path hdfs_path
```

- rm file
```bash
hdfs dfs -rm hdfs_path
```

- rm -r directory
```bash
hdfs dfs -rm -r hdfs_path
```

### REST API - CLI:

1. Make directory
```bash
curl -i -X PUT "vl26.mini.pw.edu.pl:50070/webhdfs/v1/user/galowskim/test6?user.name=hdfs&op=MKDIRS"
```

2. Create file
```bash
curl -i -X PUT "vl26.mini.pw.edu.pl:50070/webhdfs/v1/user/galowskim/tescik.txt?user.name=testuser&op=CREATE"
```

3. Append to file
```bash
curl -i -X PUT -T tesciiik.txt "node1:50075/webhdfs/v1/user/galowskim/tesciiik.txt?op=CREATE&user.name=testuser&namenoderpcaddress=node1:8020&overwrite=false"
```


4. Create file with input from local file
```bash
curl -i -X PUT -T Desktop\zamek_kaniowski.txt "vl26.mini.pw.edu.pl:50075/webhdfs/v1/user/galowskim/tesciiik.txt?op=CREATE&user.name=testuser&namenoderpcaddress=node1:8020&overwrite=false"
```

5. Otwieranie pliku (NIE DZIAŁA)
```bash
curl -i -L "vl26.mini.pw.edu.pl:50070/webhdfs/v1/user/galowskim/tesciiik.txt?user.name=hdfs&op=OPEN"
```

6. Otwieranie działa
```bash
curl -i -L "http://vl26.mini.pw.edu.pl:50075/webhdfs/v1/user/galowskim/tesciiik.txt?op=OPEN&user.name=testuser&namenoderpcaddress=node1:8020&offset=5"
```


7. Usuniecie katalogu
```bash
curl -i -X DELETE "vl26.mini.pw.edu.pl:50070/webhdfs/v1/user/galowskim?user.name=hdfs&op=DELETE"
```

### REST API - Python:
```python
import pywebhdfs
```

--------------

# Hive commands

## Server connection

```bash
hive -h <host_name> -p <port>
```

To quit:
```
quit;
```

## Query execution

```bash
hive -e <query in quotes> or hive -f <file_name>
```

```bash
hive -e "select * from employees limit 10"
```

## Basic commands

- show databases
```sql
show databases;
```

- list tables
```sql
show tables in default;
```

- sample content
```sql
select * from employees limit 10;
```

- save variable
```sql
select 2+3 as calculation;
```

- parametrized script
```sql
select * from employees limit ${hivevar:ROW_LIMIT};
```

- execute it
```bash
beeline -u jdbc:hive2://localhost:10000/ -hivevar ROW_LIMIT=10 -f test.hql
```

- also variable can be set in script
```sql
set TEST_VAR='test';
SET hivevar:ROW_LIMIT=2;
SET;
SELECT * FROM employees LIMIT ${hivevar:ROW_LIMIT};
```

- create table
```sql
CREATE TABLE IF NOT EXISTS wifi (
    id INT,
    name STRING,
    x_wgs84 STRING,
    y_wgs84 STRING,
) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\n';
LINES TERMINATED BY '\n';

LOAD DATA LOCAL INPATH '/home/vagrant/Desktop/big_data/data/wifi.csv' OVERWRITE INTO TABLE wifi;
```

- data insertion
```sql
FROM wifi
INSERT OVERWRITE TABLE wifi1
SELECT * where name like 'awil-%'
INSERT OVERWRITE TABLE wifi2
SELECT * where id>21 and id<=32;
```

### Different storing techniques

- as parquet
```sql
CREATE TABLE wifi_par STORED AS PARQUET as SELECT * FROM wifi;
```

- as avro
```sql
CREATE TABLE wifi_avro STORED AS AVRO as SELECT * FROM wifi;
```

- external tables
```sql
CREATE EXTERNAL TABLE external_table_trams (brigade INT, firstLine
INT,time TIMESTAMP,status STRING, lon DOUBLE, lat DOUBLE, line
INT, lowfloor BOOLEAN, finaltime TIMESTAMP)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\073'
LINES TERMINATED BY '\n'
LOCATION '/user/<user_name>/external_table_trams';
LOAD DATA INPATH '/user/<user_name>/trams.csv' INTO TABLE
external_table_trams;
```

- modyfiying timestamp format
```sql
ALTER TABLE external_table_trams SET SERDEPROPERTIES
("timestamp.formats"="yyyy-MM-dd'T'HH:mm:ss,yyyy-MM-dd'T'HH:mm:ss.SSS");
```

IF SOMETHING DOES NOT WORK, TRY:
```sql
SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=non-strict;
SET hive.enforce.bucketing=true;
```

- dynamic partitioning
```sql
CREATE EXTERNAL TABLE external_table_trams_part (brigade INT, firstLine INT,time TIMESTAMP, status STRING, lon DOUBLE, lat DOUBLE, line INT, lowfloor BOOLEAN, finaltime TIMESTAMP)
PARTITIONED BY (day STRING)
CLUSTERED BY (line) SORTED BY (line ASC) INTO 5 BUCKETS
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\073'
LINES TERMINATED BY '\n'
LOCATION '/user/<user_name>/external_table_trams_part';
```

- insert data
```sql
INSERT INTO external_table_trams_part PARTITION(day)
SELECT brigade, firstLine, time, status, lon, lat, line, lowfloor, finaltime, CURRENT_DATE day FROM external_table_trams;
```

