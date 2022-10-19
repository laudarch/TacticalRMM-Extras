# Prepair your Environment
Project Name: TRMM for Zabbix\
Author: Bernardo Lankheet\
Telegram: [@bernardolankheet](https://t.me/bernardolankheet)\
Description EN: Collect Status Metrics on TacticalRMM . *It does not apply in environments that use table partitioning or other routines for maintaining the bank.*\

### Steps:
1. Enable core status on TacticalRmm. On: **TRMM: Tips and Tricks**
2. Create Zabbix monitoring. On: **Zabbix: Instructions**

## (TRMM Tips and Tricks)<https://docs.tacticalrmm.com/tipsntricks/#monitor-your-trmm-instance-via-the-built-in-monitoring-endpoint>

### Monitor your TRMM Instance via the Built-in Monitoring Endpoint.
Generate a random string to be used as a token and append it to the bottom of `/rmm/api/tacticalrmm/tacticalrmm/local_settings.py` like this:

```python
MON_TOKEN = "SuperSekretToken123456"
```

Then restart Django to activate the endpoint with `sudo systemctl restart rmm.service`

Send a POST request with the following json payload to `https://api.yourdomain.com/core/status/`
```json
{"auth": "SuperSekretToken123456"}
```

Example using curl:
```
curl -X POST https://api.yourdomain.com/core/status/ -d '{"auth": "SuperSekretToken123456"}' -H 'Content-Type: application/json'
```

Response will look something like this:
```json
{
  "version": "0.14.0",
  "agent_count": 984,
  "client_count": 23,
  "site_count": 44,
  "disk_usage_percent": 12,
  "mem_usage_percent": 36,
  "days_until_cert_expires": 47,
  "cert_expired": false,
  "services_running": {
    "django": true,
    "mesh": true,
    "daphne": true,
    "celery": true,
    "celerybeat": true,
    "redis": true,
    "postgres": true,
    "mongo": true,
    "nats": true,
    "nats-api": true,
    "nginx": true
  }
}
```

## Zabbix: Instructions
Tested on Zabbix 5.0. 

Template Macros:
![Template Macros](./img/macros.jpg)

1. [download](./Template-Tactical-RMM-Monitoring.xml) and import template to zabbix server (Configuration-->Templates-->Import)

2. link template to validator host

3. configure host Macros
   - {$RMM_API_URL}           = Tactical API address. **api.tacticalrmm.com**
   - {$HTTP}                  = http or https, default value `https`
   - {$TIME_API_DOWN}         = Used on Trigger **TRMM: API or WebService Down**, value default: `10m`
   - {$TIME_SERVICES_DOWN}    = Used on Trigger **TRMM: Service XXXXXX is Down**, value default: `15m`
   - {$LLD_DAYS_EXPIRE}       = Used on Trigger **TRMM: Certificate SSL is expiring ({ITEM.LASVALUE}<{$LLD_DAYS_EXPIRE})**, value default: `10` days
   - {$LOW_DISK_UTIL}         = Used on Trigger **TRMM: Disk space is XXXXX**, value default: `93`
   - {$HIGH_MEMORY_UTIL}      = Used on Trigger **TRMM: High memory utilization**, value default: `97`

### some screenshots
***monitoring values***

![monitoring values](./img/latestdata.jpg)

***triggers***

![Triggers](./img/triggers.jpg)