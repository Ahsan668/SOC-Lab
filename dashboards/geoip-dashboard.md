# GeoIP Authentication Dashboard

## Purpose

Visualizes the geographic origin of failed authentication attempts against both the
Windows Server (RDP) and Ubuntu endpoint (SSH). Built to demonstrate that brute-force
attacks originate from real-world IP addresses, not just internal lab traffic.

## Why private IPs don't appear on the map

All lab VMs use RFC1918 private IP addresses (192.168.56.x, 192.168.10.x). GeoIP
databases contain no entries for private address ranges — they are by definition
unroutable on the public internet and have no geographic location. A brute-force
attack from 192.168.56.103 (Kali) therefore never populates the map, regardless of
how the pipeline is configured.

## How the GeoIP pipeline was validated

Rather than modifying the actual attack source, a test document was manually inserted
into the correct Elasticsearch data stream with a real public IP (8.8.8.8, Google DNS,
registered to the United States):

```bash
curl -k -u elastic:------------------- \
  -X POST "https://localhost:9200/logs-system.security-default/_doc" \
  -H 'Content-Type: application/json' \
  -d '{
    "@timestamp": "2026-09-10T18:00:00.000Z",
    "agent": { "name": "WIN-C1E99BQR58K" },
    "source": { "ip": "8.8.8.8", "geo": { "country_iso_code": "US" } },
    "event": { "code": "4625", "action": "logon-failed" },
    "winlog": { "event_id": 4625 }
  }'
```

The United States appeared on the world map immediately after insertion, confirming
the GeoIP enrichment pipeline is working correctly end-to-end.

## Key learning

GeoIP maps in a local lab will only show data when real public IPs are present in the
logs. In a production SOC environment pointing at internet-facing services, the map
would populate automatically from real attacker IPs.
