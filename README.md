# SQL Server CDC Using Kafka Demo

```shell
# Start the infrastructure 
docker-compose up -d

# Initialize database and insert test data
cat inventory.sql | docker-compose exec -T sqlserver bash -c '/opt/mssql-tools/bin/sqlcmd -U sa -P $SA_PASSWORD'

# Start SQL Server connector
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-sqlserver.json

# list availabe kafka topics
docker-compose exec kafka /kafka/bin/kafka-topics.sh --list \
    --bootstrap-server kafka:9092
    
# Consume messages from a Debezium topic
docker-compose exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic server1.dbo.customers

# Modify records in the database via SQL Server client (do not forget to add `GO` command to execute the statement)
docker-compose exec sqlserver bash -c '/opt/mssql-tools/bin/sqlcmd -U sa -P $SA_PASSWORD -d testDB'

# Shut down the cluster
docker-compose down
```

## Azure Event Hubs
_Note_: Not working yet!
```shell
# Start the infrastructure 
docker-compose up -d

# Initialize database and insert test data
cat inventory.sql | docker-compose exec -T sqlserver bash -c '/opt/mssql-tools/bin/sqlcmd -U sa -P $SA_PASSWORD'

# Start SQL Server connector
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-sqlserver-eh.json

# list availabe kafka topics
docker-compose exec kafka /kafka/bin/kafka-topics.sh --list \
    --bootstrap-server pli-kafka-eh.servicebus.windows.net:9093
    
# Consume messages from a Debezium topic
docker-compose exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server pli-kafka-eh.servicebus.windows.net:9093 \
    --from-beginning \
    --property print.key=true \
    --topic server1.dbo.customers

# Modify records in the database via SQL Server client (do not forget to add `GO` command to execute the statement)
docker-compose exec sqlserver bash -c '/opt/mssql-tools/bin/sqlcmd -U sa -P $SA_PASSWORD -d testDB'

# Shut down the cluster
docker-compose down

```

## Azure Event Hubs Integration with MirrorMaker
```shell
# Start the infrastructure 
docker-compose up -d

# Setup  MirrorMaker
docker-compose exec kafka bash

# create source-kafka.config and mirror-eventhub.config under config/ and run the following command and keep it running
bin/kafka-mirror-maker.sh --consumer.config config/source-kafka.config --num.streams 1 --producer.config config/mirror-eventhub.config --whitelist=".*"

# Initialize database and insert test data
cat inventory.sql | docker-compose exec -T sqlserver bash -c '/opt/mssql-tools/bin/sqlcmd -U sa -P $SA_PASSWORD'

# Start SQL Server connector
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-sqlserver.json

# list availabe kafka topics
docker-compose exec kafka /kafka/bin/kafka-topics.sh --list \
    --bootstrap-server kafka:9092
    
# Consume messages from a Debezium topic
docker-compose exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic server1.dbo.customers

# Modify records in the database via SQL Server client (do not forget to add `GO` command to execute the statement)
docker-compose exec sqlserver bash -c '/opt/mssql-tools/bin/sqlcmd -U sa -P $SA_PASSWORD -d testDB'

# Manually create the EventHubs (e.g., server1.dbo.customers)

# Using Azure Service Bus Explorer to monitor the changes in EventHub

# Shut down the cluster
docker-compose down
```