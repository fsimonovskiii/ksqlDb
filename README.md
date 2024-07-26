<!-- ***

<br /> -->

# ksqlDB

The database purpose-built for stream processing applications.

# CREATE STREAM riderLocations

```sql
CREATE STREAM riderLocations (
    profileId VARCHAR,
    latitude DOUBLE,
    longitude DOUBLE)
  WITH (kafka_topic='locations', value_format='json', partitions=1);
```


## Materialized Views
ksqlDB allows you to define materialized views over your streams and tables.Materialized views are defined by what is known as a "persistent query". These queries are known as persistent because they maintain their incrementally updated results using a table.

```sql
CREATE TABLE hourly_metrics AS
  SELECT url, COUNT(*)
  FROM page_views
  WINDOW TUMBLING (SIZE 1 HOUR)
  GROUP BY url EMIT CHANGES;
```

Results may be **"pulled"** from materialized views on demand via `SELECT` queries. The following query will return a single row:

```sql
SELECT * FROM hourly_metrics
  WHERE url = 'http://myurl.com' AND WINDOWSTART = '2019-11-20T19:00';
```

Results may also be continuously **"pushed"** to clients via streaming `SELECT` queries. The following streaming query will push to the client all incremental changes made to the materialized view:

```sql
SELECT * FROM hourly_metrics EMIT CHANGES;
```

Streaming queries will run perpetually until they are explicitly terminated.

## Streaming ETL

Apache Kafka is a popular choice for powering data pipelines. ksqlDB makes it simple to transform data within the pipeline, readying messages to cleanly land in another system.

```sql
CREATE STREAM vip_actions AS
  SELECT userid, page, action
  FROM clickstream c
  LEFT JOIN users u ON c.userid = u.user_id
  WHERE u.level = 'Platinum' EMIT CHANGES;
```

## Anomaly Detection

ksqlDB is a good fit for identifying patterns or anomalies on real-time data. By processing the stream as data arrives you can identify and properly surface out of the ordinary events with millisecond latency.

```sql
CREATE TABLE possible_fraud AS
  SELECT card_number, count(*)
  FROM authorization_attempts
  WINDOW TUMBLING (SIZE 5 SECONDS)
  GROUP BY card_number
  HAVING count(*) > 3 EMIT CHANGES;
```

## Monitoring

Kafka's ability to provide scalable ordered records with stream processing make it a common solution for log data monitoring and alerting. ksqlDB lends a familiar syntax for tracking, understanding, and managing alerts.

```sql
CREATE TABLE error_counts AS
  SELECT error_code, count(*)
  FROM monitoring_stream
  WINDOW TUMBLING (SIZE 1 MINUTE)
  WHERE  type = 'ERROR'
  GROUP BY error_code EMIT CHANGES;
```

## Integration with External Data Sources and Sinks

ksqlDB includes native integration with [Kafka Connect](https://docs.ksqldb.io/en/latest/concepts/connectors) data sources and sinks, effectively providing a unified SQL interface over a [broad variety of external systems](https://www.confluent.io/hub).

The following query is a simple persistent streaming query that will produce all of its output into a topic named `clicks_transformed`:

```sql
CREATE STREAM clicks_transformed AS
  SELECT userid, page, action
  FROM clickstream c
  LEFT JOIN users u ON c.userid = u.user_id EMIT CHANGES;
```

Rather than simply send all continuous query output into a Kafka topic, it is often very useful to route the output into another datastore. ksqlDB's Kafka Connect integration makes this pattern very easy.

The following statement will create a Kafka Connect sink connector that continuously sends all output from the above streaming ETL query directly into Elasticsearch:

```sql
 CREATE SINK CONNECTOR es_sink WITH (
  'connector.class' = 'io.confluent.connect.elasticsearch.ElasticsearchSinkConnector',
  'key.converter'   = 'org.apache.kafka.connect.storage.StringConverter',
  'topics'          = 'clicks_transformed',
  'key.ignore'      = 'true',
  'schema.ignore'   = 'true',
  'type.name'       = '',
  'connection.url'  = 'http://elasticsearch:9200');
```

# Useful Snippets

### How to convert a 'KStream' to a 'KTable' in Kafka Streams
https://developer.confluent.io/confluent-tutorials/streams-to-table/kstreams/

### How to rekey a stream with ksqlDB
https://developer.confluent.io/confluent-tutorials/rekeying/ksql/
