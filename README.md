# telegraf Ansible Role

[telegraf](https://www.influxdata.com/) is a self-hosted time-series database. This role helps you to set up telegraf:

- with everything run in [Docker](https://www.docker.com/) containers
- powered by [the official telegraf container image](https://hub.docker.com/r/_/telegraf/)

## Prerequisits

* A installed and running [infuxdb](https://www.influxdata.com/).


## Installing

To configure and install telegraf on your own server(s), you should use a playbook like [Mother of all self-hosting](https://github.com/mother-of-all-self-hosting/mash-playbook) or write your own.

## Configuring this role for your playbook

This role depends on a influxdb configuring telegraf. You need to obtain the influx token and config link in the influxdb.
In your browser, visit the influxdb and go to Load Data -> Telegraf.
There you need to add a telegraf configuraion. You can now obtain these values from the setup instructions and oaste them here.

```yaml
telegraf_enabled: true
telegraf_influx_token: SUPERSECRETTOKEN
telegraf_config_link: https://influxdb.example.org/api/v2/telegrafs/01234569
```

## Usage

In your influxdb configure the telegraf plugins as you like.

## Support

- Github issues: [mother-of-all-self-hosting/ansible-role-telegraf/issues](https://github.com/mother-of-all-self-hosting/ansible-role-telegraf.git/issues)
