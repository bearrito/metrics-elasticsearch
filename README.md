metrics-elasticsearch
=====================

[Kibana](http://www.elasticsearch.org/overview/kibana/)-friendly Transport and HTTP [Elasticsearch](http://www.elasticsearch.org/) reporters for [Dropwizard Metrics](https://github.com/dropwizard/metrics)

## Versions

If using the HTTP API, any version of Elasticsearch that supports the [Bulk API](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/docs-bulk.html) will report fine using any version of metrics-elasticsearch.
 
However, if using the Transport API, please note that metrics-elasticsearch uses a specific version of Elasticsearch noted in the table below.

| metrics-elasticsearch  | dropwizard metrics | elasticsearch |
| ------------- | ------------- | ------------- |
| 1.0.0  | 3.1.0  | 1.2.4  |
| 1.0.1  | 3.1.1  | 1.2.4  |
| 1.1.0  | 3.1.1  | 1.3.9  |

## Usage

### Setting up Maven & Gradle

* [Maven](https://github.com/tomcashman/metrics-elasticsearch/wiki/Setting-up-Maven)
* [Gradle](https://github.com/tomcashman/metrics-elasticsearch/wiki/Setting-up-Gradle)

### Reporters

The [Dropwizard Metrics user guide](https://dropwizard.github.io/metrics/3.1.0/getting-started/) documents how to set up the metrics registry and create different types of metrics.

This library provides two different methods to report to Elasticsearch; using the [HTTP API](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-http.html) and using the [Transport API](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-transport.html). 

#### HTTP Reporter

The following will create a reporter that uses an already initialised Metric Registry, sets the indexes generated as "test-YYYY.MM.DD" and reports over HTTP to localhost:9201.

```java
ElasticsearchReporter reporter = ElasticsearchHttpReporter
                .forRegistryAndIndexPrefix(registry, "test")
                .build("localhost:9201");
reporter.start(1, TimeUnit.SECONDS);
```

The reporter above is created with default values. You can change how the metrics are reported using further options like the example below.

```java
ElasticsearchReporter reporter = ElasticsearchHttpReporter
                .forRegistryAndIndexPrefix(registry, "test")
                .withTimestampFieldName("@timeywimey")
                .withClock(Clock.defaultClock()).prefixedWith("metric-name-prefix-")
                .convertRatesTo(TimeUnit.SECONDS)
                .convertDurationsTo(TimeUnit.MILLISECONDS)
                .withBulkRequestLimit(10).filter(MetricFilter.ALL)
                .build("localhost:9201");
reporter.start(1, TimeUnit.SECONDS);
```

#### Transport Reporter

The following will create a reporter that uses an already initialised Metric Registry, sets the indexes generated as "test-YYYY.MM.DD" and reports over the Transport API using an [Elasticsearch client instance](http://www.elasticsearch.org/guide/en/elasticsearch/client/java-api/current/client.html).

```java
ElasticsearchReporter reporter = ElasticsearchTransportReporter
                .forRegistryAndIndexPrefix(registry, "test")
                .build(client);
reporter.start(1, TimeUnit.SECONDS);
```

The reporter above is created with default values. You can change how the metrics are reported using further options like the example below.

```java
ElasticsearchReporter reporter = ElasticsearchTransportReporter
                .forRegistryAndIndexPrefix(registry, "test")
                .withTimestampFieldName("@timeywimey")
                .withClock(Clock.defaultClock()).prefixedWith("metric-name-prefix-")
                .convertRatesTo(TimeUnit.SECONDS)
                .convertDurationsTo(TimeUnit.MILLISECONDS)
                .withBulkRequestLimit(10).filter(MetricFilter.ALL)
                .build(client);
reporter.start(1, TimeUnit.SECONDS);
```

## Data Format

The data will be inserted under the specified index suffixed with the year, month and day (YYYY.MM.DD). 
The type used is the metric type (e.g. guage, histrogram, meter, etc.). 
Each document inserted into Elasticsearch will have its ID generated by Elasticsearch. 
The document itself stores the timestamp, metric name and metric values.