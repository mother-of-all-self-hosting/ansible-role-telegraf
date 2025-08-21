<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Telegraf

This is an [Ansible](https://www.ansible.com/) role which installs [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Telegraf is an open source server agent to help you collect metrics from your stacks, sensors, and systems.

See the project's [documentation](https://docs.influxdata.com/telegraf/v1/) to learn what Telegraf does and why it might be useful to you.

## Adjusting the playbook configuration

To enable Telegraf with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# telegraf                                                             #
#                                                                      #
########################################################################

telegraf_enabled: true

########################################################################
#                                                                      #
# /telegraf                                                            #
#                                                                      #
########################################################################
```

### Set variables for connectiong to an InfluxDB instance (optional)

The Telegraf instance can be configured to collect and write metrics to [InfluxDB](https://www.influxdata.com/) or other outputs.

>[!NOTE]
> If you are looking for an Ansible role for InfluxDB, you can check out [this role (ansible-role-influxdb)](https://github.com/mother-of-all-self-hosting/ansible-role-influxdb) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

You can connect the Telegraf instance with the an InfluxDB instance by adding the following configuration to your `vars.yml` file:

```yaml
telegraf_influx_token: YOUR_INFLUXDB_TOKEN_HERE

telegraf_config_link: https://influxdb.example.com/api/v2/telegrafs/0123456789
```

Those values can be retrieved from your InfluxDB instance. To retrieve them, open the instance's URL (`https://influxdb.example.com`) and go to **Load Data** -> **Telegraf**.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `telegraf_environment_variables_additional_variables` variable

For a complete list of Telegraf's config options that you could put in `telegraf_environment_variables_additional_variables`, see its [environment variables](https://docs.influxdata.com/telegraf/v1/configuration/#set-environment-variables).

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, the Telegraf instance becomes available.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu telegraf` (or how you/your playbook named the service, e.g. `mash-telegraf`).
