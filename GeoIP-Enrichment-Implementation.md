# GeoIP Enrichment Implementation

# Overview

# PHASE #1: Creating a custom endpoint to host your own database

Manage your own GeoIP Database Updates using a custom Endpoint.

## Description

This section will provide a brief overview of steps to complete this phase of the implementation. Refer to the reference section below to navigate to the full documentation.

### Disable the auto-update features

- Disable the logstash database auto-update feature: set the `xpack.geoip.downloader.enabled` value to `false` in `logstash.yml`.

- Disable the elasticsearch database auto-update feature via the cluster update API (https://www.elastic.co/guide/en/elasticsearch/reference/8.8/cluster-update-settings.html): set the `xpack.geoip.downloader.enabled` value to `false` in `logstash.yml`.

### Create a custom endpoint to host and update the database

- Create a custom Docker endpoint with the `elasticseach` and `geoipupdate` services configured and running.

[Sign up for or login to a MaxMind account](https://www.maxmind.com/en/account/login)

[Install GeoIP Update](https://dev.maxmind.com/geoip/updating-databases#using-geoip-update)

[Obtain GeoIP.conf with Account Information](https://dev.maxmind.com/geoip/updating-databases#2-obtain-geoipconf-with-account-information)

[Automatically run geoipupdate using a crontab file](https://dev.maxmind.com/geoip/updating-databases#3-run-geoip-update)

[Install and configure Elasticsearch](https://www.elastic.co/downloads/elasticsearch)

- From your Elasticsearch directory, run:

```text
./bin/elasticsearch-geoip -s my/database/dir
```
- Serve the static database files from Docker directory using an nginx server:

```text
docker run -v my/source/dir:/usr/share/nginx/html:ro nginx
```

### Specify the service's endpoint URL

- Logstash settings: using the `xpack.geoip.download.endpoint=http://localhost:8080/overview.json` setting in `logstash.yml`.

- Elasticsearch settings: In the `ingest.geoip.downloader.endpoint` setting of each node’s `elasticsearch.yml` file

### References

[Elastic (Logstash)- Manage your own database updates using a custom endpoint](https://www.elastic.co/guide/en/logstash/current/plugins-filters-geoip.html#plugins-filters-geoip-manage_update)

[Elastic (Elasticsearch) - Manage your own database updates using a custom endpoint](https://www.elastic.co/guide/en/elasticsearch/reference/8.8/geoip-processor.html#manage-geoip-database-updates)

[Maxmind - Updating GeoIP and GeoLite Databases](https://dev.maxmind.com/geoip/updating-databases)

# PHASE #2: Dtermining what logs to enrich

Determine what log source to GeoIP enrich and click the associated "enabled by" table column value:

|---------------------------------------|
|LOG SOURCE		| ENABLED BY (click)	|
|---------------------------------------|
|Logstash		| Using GeoIP Filter	|
|Filebeat		| Using GeoIP Processor	|
|Indexes		| Processor + Pipeline	|
|Index Templates| Processor + Pipeline	|
|Beats			| Processor + Pipeline	|
|---------------------------------------|


# PHASE #3: Configuring GeoIP Enrichment for your log sources

Specific steps to configure the Elastic GeoIP Enrichment for your desired log source.


## Using GeoIP Filter

Description:

The GeoIP filter adds information about the geographical location of IP addresses, based on data from the MaxMind GeoLite2 databases.


### Logstash: Geo Enrichment with GeoIP Filter Plugin

The geoip filter has a very simple function, it queries an IP address in an internal database, identify its geolocation and returns some fields like the country name, country code, city, geographic coordinates and a few others.

In the case of the `geoip` filter used by logstash, the internal database with the geolocation information is the `GeoLite2`, provided by maxmind, and using this filter with an IP address it is possible to obtain, in addition to the geographical information, data related to the Autonomous System (AS), associated with the routing for the IP.

To use the `geoip` filter you need your event to have a field where the value is a public IP address and you also need to create a specific mapping for your index to store fields with geolocation data.

Reference Documenation:

[Elastic - Logstash GeoIP Enrichment w/ GeoIP Filter](https://www.elastic.co/guide/en/logstash/current/plugins-filters-geoip.html)

[Configuration and usage example](https://web.leandrojmp.com/posts/en/2020/10/logstash-geoip)


## Using GeoIP Processor

Description:

The geoip processor adds information about the geographical location of an IPv4 or IPv6 address.

By default, the processor uses the GeoLite2 City, GeoLite2 Country, and GeoLite2 ASN GeoIP2 databases from MaxMind.

Reference Documentation:

[Elastic - GeoIP Processor](https://www.elastic.co/guide/en/elasticsearch/reference/current/geoip-processor.html)

### Filebeat: Geo Enrichment with GeoIP Procesor

Description:

You can use Filebeat along with the GeoIP Processor in Elasticsearch to export geographic location information based on IP addresses. Then you can use this information to visualize the location of IP addresses on a map in Kibana.

The `geoip` processor adds information about the geographical location of IP addresses, based on data from the Maxmind GeoLite2 City Database. 

Because the processor uses a geoIP database that’s installed on Elasticsearch, you don’t need to install a geoIP database on the machines running Filebeat.

Note:

If your use case involves using Logstash, you can use the GeoIP filter available in Logstash instead of using the geoip processor. However, using the `geoip` processor is the simplest approach when you don’t require the additional processing power of Logstash.


Reference Documenation:

[Filebeat: Geo Enrichment with GeoIP Procesor](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-geoip.html)

### Packetbeat: Geo Enrichment with GeoIP Procesor

Description:

You can use Packetbeat along with the GeoIP Processor in Elasticsearch to export geographic location information based on IP addresses. Then you can use this information to visualize the location of IP addresses on a map in Kibana.

The `geoip` processor adds information about the geographical location of IP addresses, based on data from the Maxmind GeoLite2 City Database. 

Because the processor uses a geoIP database that’s installed on Elasticsearch, you don’t need to install a geoIP database on the machines running Packetbeat.

Note:

If your use case involves using Logstash, you can use the GeoIP filter available in Logstash instead of using the geoip processor. However, using the `geoip` processor is the simplest approach when you don’t require the additional processing power of Logstash.

Reference Documentation:

[Packetbeat: Geo Enrichment with GeoIP Procesor](https://www.elastic.co/guide/en/beats/packetbeat/current/packetbeat-geoip.html)

## Using GeoIP Processor + Pipeline

Description:

Reference Documentation:


### Indexes: Geo Enrichment with GeoIP Pipeline

Description:

You can set a default or final pipeline to a given ingested index or index template...

Reference Documentation:

[Elastic - Ingest Pipelines](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html)

### Index - Default Pipeline
Use the index.default_pipeline index setting to set a default pipeline. 

Elasticsearch applies this pipeline to indexing requests if no pipeline parameter is specified.

```text
"index.default_pipeline": "geoip"
```

Reference Documentation:

[Elastic - Default Pipelines](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html#set-default-pipeline)


### Index - Final Pipeline	
Use the index.final_pipeline index setting to set a final pipeline. 

Elasticsearch applies this pipeline after the request or default pipeline, even if neither is specified.

```text
"index.final_pipeline": "geoip"
```	

Reference Documentation:
[Elastic - Final Pipeline](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html#set-final-pipeline)

### Index Templates: Geo Enrichment with GeoIP Pipeline

Create an index template that includes your pipeline in the `index.default_pipeline` or `index.final_pipeline` index setting. 

Ensure the template is data stream enabled. 

The template’s index pattern should match `logs-<dataset-name>-*`.

Reference Documentation: 

[Index Templates: Geo Enrichment with GeoIP Pipeline](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html)


### Beats: Geo Enrichment with GeoIP Pipeline

To add an ingest pipeline to an Elastic Beat, specify the pipeline parameter under `output.elasticsearch` in `<BEAT_NAME>.yml`. 

For example, for Filebeat, you’d specify pipeline in `filebeat.yml`.

```text
pipeline: geoip
```

Reference Documnetation:
[Elastic - Pipelines for Beats](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html#pipelines-for-beats)




