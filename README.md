# Cloudflare exporter for Prometheus

A very simple Prometheus exporter that exposes metrics from Cloudflare's colocations API as described in the [API documentation](https://api.cloudflare.com/#zone-analytics-analytics-by-co-locations). Sadly, this is for Cloudflare Enterprise customers only.
It'll expose metrics per PoP and shows requests, bandwidth and threats.

Please note that because of how the Cloudflare API works this exporter will only return statistics from `now() - 5 minutes`.

In addition, the exporter will also fetch DNS analytics. Exporting metrics per records queried at edge resolvers per PoP.

The following roles are required:

* Analytics Administrator
* Firewall Administrator

### Try it

Running the container:

```
docker run \
 -d \
 -p 9199:9199 \                                                    
 -e ZONE=example.com \
 -e AUTH_KEY=deadbeefcafe \
 -e AUTH_EMAIL=admin@example.com \
vaheminasyan/cloudflare-exorter:0.5.1
```

### Metrics
The exporter exposes the following metrics, all returned per PoP, or per country:


| Name                                              | Description                                             |  Type |
|:--------------------------------------------------|:--------------------------------------------------------|:-----:|
| `bandwidth_byte_per_country`                      | cached/uncached bandwidth used per country              | gauge |
| `bandwidth_bytes_all_countries`                   | cached/uncached bandwidth used from all countries       | gauge |
| `cloudflare_exporter_processing_time_miliseconds` | exporter processing time                                | gauge |
| `cloudflare_pop_bandwidth_bytes`                  | cached/uncached bandwidth sent from an edge-location    | gauge |
| `cloudflare_pop_http_responses_sent`              | breakdown of requests per HTTP code                     | gauge |
| `cloudflare_pop_received_requests`                | cached/uncached requests received on an edge-location   | gauge |
| `cloudflare_pop_threats_seen`                     | number of threats identified per location               | gauge |
| `cloudflare_pop_threats_seen_by_country`          | countries causing threats                               | gauge |
| `http_responses_sent_all_countries`               | sent http responses sent to all countries               | gauge |
| `received_requests_all_countries`                 | received requests from all countries                    | gauge |
| `received_requests_by_country`                    | received requests breakdown by country                  | gauge |
| `cloudflare_content_type_count`                   | Requests count breakdown by content-type                | gauge |
| `bandwidth_byte_per_content_type`                 | Bandwidth breakdown by content-type                     | gauge |


Random scrape result:

```python
# HELP cloudflare_pop_http_responses_sent Breakdown per HTTP response code.
# TYPE cloudflare_pop_http_responses_sent gauge
cloudflare_pop_http_responses_sent{colo_id="BRU",http_status="200",zone="example.com"} 25.0
cloudflare_pop_http_responses_sent{colo_id="BRU",http_status="302",zone="example.com"} 1.0
cloudflare_pop_http_responses_sent{colo_id="BRU",http_status="204",zone="example.com"} 2.0
# HELP cloudflare_pop_threat_countries Threat breakdown per threat country.
# TYPE cloudflare_pop_threat_countries gauge
# HELP cloudflare_pop_threat_types Threat breakdown per threat type.
# TYPE cloudflare_pop_threat_types gauge
# HELP cloudflare_pop_threats_seen Threats identified.
# TYPE cloudflare_pop_threats_seen gauge
cloudflare_pop_threats_seen{colo_id="BRU",zone="example.com"} 0.0
# HELP cloudflare_pop_bandwidth_bytes Bandwidth sent from this PoP location.
# TYPE cloudflare_pop_bandwidth_bytes gauge
cloudflare_pop_bandwidth_bytes{colo_id="BRU",type="cached",zone="example.com"} 404362.0
cloudflare_pop_bandwidth_bytes{colo_id="BRU",type="uncached",zone="example.com"} 68411.0
# HELP cloudflare_pop_received_requests Requests received at this PoP location.
# TYPE cloudflare_pop_received_requests gauge
cloudflare_pop_received_requests{colo_id="BRU",type="cached",zone="example.com"} 10.0
cloudflare_pop_received_requests{colo_id="BRU",type="uncached",zone="example.com"} 18.0
# HELP cloudflare_dns_record_queries DNS queries per record at PoP location.
# TYPE cloudflare_dns_record_queries gauge
cloudflare_dns_record_queries{colo_id="SOF",query_response="NXDOMAIN",record_name="qlgijqgzsd.example.com",record_type="A",zone="example.com"} 1.0
cloudflare_dns_record_queries{colo_id="LAX",query_response="NOERROR",record_name="www.example.com",record_type="A",zone="example.com"} 5.0
# HELP cloudflare_waf_rules WAF-rules in the system and a hit count.
# TYPE cloudflare_waf_rules counter
cloudflare_waf_rules{rule_id="unknown",rule_message="internal"} 9.0
# HELP cloudflare_waf_uri_hits WAF-rule hits at PoP location per uri.
# TYPE cloudflare_waf_uri_hits gauge
cloudflare_waf_uri_hits{action="block",attacking_country="T1",colo_id="OSL",host="www.example.com",method="GET",protocol="HTTP/1.1",rule_id="unknown",uri="/"} 9.0
```



### Dashboard

A sample Grafana dashboard is [included](grafana-cloudflare-dashboard.json) to visualize the information.

Overview of requests, cached and uncached:
![Requests](./docs/assets/requests_total.png)

Overview of requests served per Cloudflare network point-of-presence:
![Requests per PoP](./docs/assets/requests_per_pop.png)

Overview of actual countries that cause threats:
![Attacking Countries](./docs/assets/threats.png)

### Todo

- [ ] Implement a way to store datapoints in Prometheus using timestamps received from Cloudflare. This should remove the delay as we currently have it.
