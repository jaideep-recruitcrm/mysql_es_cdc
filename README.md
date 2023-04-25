## Elasticsearch Sink

### Topology

```
                   +-------------+
                   |             |
                   |    MySQL    |
                   |             |
                   +------+------+
                          |
                          |
                          |
          +---------------v------------------+
          |                                  |
          |           Kafka Connect          |
          |    (Debezium, ES connectors)     |
          |                                  |
          +---------------+------------------+
                          |
                          |
                          |
                          |
                  +-------v--------+
                  |                |
                  | Elasticsearch  |
                  |                |
                  +----------------+
```
We are using Docker Compose to deploy the following components:

* MySQL
* Kafka
  * ZooKeeper
  * Kafka Broker
  * Kafka Connect with [Debezium](https://debezium.io/) and  [Elasticsearch](https://github.com/confluentinc/kafka-connect-elasticsearch) Connectors
* Elasticsearch

### Usage

How to run:

```shell
# Start the application
export DEBEZIUM_VERSION=2.2
docker compose up -d --build

# Start Elasticsearch connector
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @es-sink.json

# Start MySQL connector
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @source.json

```

Check contents of the MySQL database:

```shell
docker compose up -d'
```

Verify that Elasticsearch has the same content:

```shell
curl 'http://localhost:9200/customers/_search?pretty'
```
#### New record

Insert a new record into MySQL:

```shell
docker exec -it mysqldbz /bin/bash
bash-4.4# mysql -u root -pdebezium
mysql> USE inventory;
mysql> SELECT * FROM customers;
mysql> INSERT INTO customers VALUES(default, 'Jaideep', 'Chahal', 'Jaideep@recruitcrm.io');
mysql> exit;
bash-4.4# exit
```

Check that Elasticsearch contains the new record:

```shell
curl 'http://localhost:9200/customers/_search?pretty'
```

#### Record update

Update a record in MySQL:

```shell
docker exec -it mysqldbz /bin/bash
bash-4.4# mysql -u root -pdebezium
mysql> USE inventory;
mysql> SELECT * FROM customers;
mysql> UPDATE customers SET first_name='Jaideep', last_name='Updated' WHERE last_name='Chahal';
mysql> exit;
bash-4.4# exit
```

Verify that record in Elasticsearch is updated:

```shell
curl 'http://localhost:9200/customers/_search?pretty'
```


#### Record delete

Delete a record in MySQL:

```shell
docker exec -it mysqldbz /bin/bash
bash-4.4# mysql -u root -pdebezium
mysql> USE inventory;
mysql> SELECT * FROM customers;
mysql> DELETE FROM customers WHERE email='Jaideep@recruitcrm.io';
mysql> exit;
bash-4.4# exit
```

Verify that record in Elasticsearch is deleted:

```shell
curl 'http://localhost:9200/customers/_search?pretty'
```

End the application:

```shell
# Shut down the cluster
docker compose down
```