# Guide

Copy the `.env.example` into `.env`:
```
cp .env.example .env
```

Populate the .env file with your settings.
You must generate a password for Elastic and Kibana.

Start Elastic stack:

```bash
cd elastic
docker-compose -f docker-compose-elastic.yml up -d
```

Once Kibana is ready:

```bash
# Prepare the FLEET application on kibana
curl -k -u "elastic:<es_password>" -XPOST http://localhost:5601/api/fleet/setup --header 'kbn-xsrf: true'

# Create a Fleet Server Policy
curl -k -u "elastic:<es_password>" "http://localhost:5601/api/fleet/agent_policies?sys_monitoring=true" \
--header 'kbn-xsrf: true' \
--header 'Content-Type: application/json' \
--data-raw '{"id":"fleet-server-policy","name":"Fleet Server policy","description":"","namespace":"default","monitoring_enabled":["logs","metrics"],"has_fleet_server":true}'

# Update Fleet Server URL
curl -k -u "elastic:<es_password>" -XPUT "http://localhost:5601/api/fleet/settings" \
--header 'kbn-xsrf: true' \
--header 'Content-Type: application/json' \
--data-raw '{"fleet_server_hosts":["http://fleet:8220"]}'

# Update Fleet Output
curl -k -u "elastic:<es_password>" -XPUT "http://localhost:5601/api/fleet/outputs/fleet-default-output" \
--header 'kbn-xsrf: true' \
--header 'Content-Type: application/json' \
--data-raw '{"name":"default","type":"elasticsearch","is_default":true,"is_default_monitoring":true,"hosts":["https://elasticsearch:9200"],"config_yaml": "ssl.certificate_authorities: [\"/usr/share/elastic-agent/config/certs/ca/ca.crt\"]"}'

# Generate Service token
curl -k -u "elastic:<es_password>D" -s -X POST http://localhost:5601/api/fleet/service-tokens --header 'kbn-xsrf: true' | jq -r .value
```

Save the token returned in the last command into the `.env` file as `FLEET_SERVER_SERVICE_TOKEN` variable.

Navigate to Kibana and modify the policy:
http://localhost:5601/app/fleet/policies/fleet-server-policy
Add ModSecurity and Apache integrations. Use the default settings.

Then run the fleet sever & agent:

```bash
docker-compose -f docker-compose-fleet.yml up -d
```

Then navigate to http://localhost, login with admin:admin, initialize the DVWA.
Login again using admin:password and perform some attacks, for example, navigate to `http://localhost/vulnerabilities/sqli/?id=%27+OR+1+%3D+1+--+&Submit=Submit#`.

This should generate Apache2 and ModSecurity logs that will be processed by the Elastic Agent, and displayed in Kibana.