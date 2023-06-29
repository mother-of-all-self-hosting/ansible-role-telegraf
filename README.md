# telegraf Ansible Role

[telegraf](https://www.influxdata.com/) is a self-hosted time-series database. This role helps you to set up telegraf:

- with everything run in [Docker](https://www.docker.com/) containers
- powered by [the official telegraf container image](https://hub.docker.com/r/_/telegraf/)


## Installing

To configure and install telegraf on your own server(s), you should use a playbook like [Mother of all self-hosting](https://github.com/mother-of-all-self-hosting/mash-playbook) or write your own.

# Configuring this role for your playbook

```
telegraf_enabled: true
telegraf_hostname: 'example.org'
```

## Advanced configuration

To bootstrap an initial user, bucket and organization you can use

```yaml
# Configure the inital user, organization and bucket
# This setting will only be used once upon initial installation of telegraf. Changing this values after the first
# start of telegraf will have no effect.
# Not setting this will allow you to manually set these by visiting the domain you set in telegraf_hostname after installation.
telegraf_init: true
telegraf_init_username: "USERNAME"
telegraf_init_password: "SUPERSECRETPASSWORD"
telegraf_init_org: "EXAMPLE_ORG"
telegraf_init_bucket: "SOMEBUCKET"
```

## Support

- Github issues: [mother-of-all-self-hosting/ansible-role-telegraf/issues](https://github.com/mother-of-all-self-hosting/ansible-role-telegraf.git/issues)
