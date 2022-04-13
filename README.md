alertmanager-discord
===

Give this a webhook (with the DISCORD_WEBHOOK environment variable) and point it as a webhook on alertmanager, and it will post your alerts into a discord channel for you as they trigger:

![](/.github/demo-new.png)

## Warning

This program is not a replacement to alertmanager, it accepts webhooks from alertmanager, not prometheus.

The standard "dataflow" should be:

```
Prometheus -------------> alertmanager -------------------> alertmanager-discord

alerting:                 receivers:                         
  alertmanagers:          - name: 'discord_webhook'         environment:
  - static_configs:         webhook_configs:                   - DISCORD_WEBHOOK=https://discordapp.com/api/we...
    - targets:              - url: 'http://localhost:9094'  
       - 127.0.0.1:9093   





```

## Example alertmanager config:

```
global:
  # The smarthost and SMTP sender used for mail notifications.
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.org'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'password'

# The directory from which notification templates are read.
templates: 
- '/etc/alertmanager/template/*.tmpl'

# The root route on which each incoming alert enters.
route:
  group_by: ['alertname']
  group_wait: 20s
  group_interval: 5m
  repeat_interval: 3h 
  receiver: discord_webhook

receivers:
- name: 'discord_webhook'
  webhook_configs:
  - url: 'http://localhost:9094'
```

## Docker

If you run a fancy docker/k8s infra, you can find the docker hub repo here: https://hub.docker.com/r/benjojo/alertmanager-discord/


### how to check webhook

    export DISCORD_WEBHOOK=https://discord.com/api/webhooks/xx/xx
    
    curl -s $DISCORD_WEBHOOK -i -H "Accept: application/json" -H "Content-Type:application/json" -X POST \
    --data '
        {
            "content":"",
            "embeds":[
                {"title":"",
                 "description":"FIRE: alertname on 1.example.net",
                 "color":10038562
                },
                {"title":"",
                 "description":"RESOLVE: alertname on 2.example.net",
                 "color":3066993
                }
            ]
        }'
    
### how to test this app

    cd ./example && docker-compose up --build

and send an alert
    
    curl -XPOST localhost:9093/api/v1/alerts -d '[
    {
        "status": "firing",
        "labels": {
            "alertname": "KubePodCrashLooping",
            "service": "postgresql-1",
            "severity":"critical",
            "instance": "172.1.1.1:8080"
        },
        "annotations": {
            "summary": "summary",
            "description": "Pod my-pod-597fc9b5bb-55kdb (service) is restarting 0.34 times / 5 minutes."
        },
        "generatorURL": "xxx"
    },
    {
        "status": "firing",
        "labels": {
            "alertname": "KubePodCrashLooping",
            "service": "postgresql-1",
            "severity":"critical",
            "instance": "172.1.1.2:8080"
        },
        "annotations": {
            "summary": "summary",
            "description": "Pod my/pod-597fc9b5bb-9b5bb (service) is restarting 0.34 times / 5 minutes."
        },
        "generatorURL": "xxx"
    }
    ]'
