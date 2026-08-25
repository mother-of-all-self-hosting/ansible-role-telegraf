<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2026 Slavi Pantaleev
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

Telegraf is an open-source server agent to help you collect metrics from your stacks, sensors, and systems.

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

telegraf_configuration: |
  [agent]
    interval = "10s"
    flush_interval = "10s"

  [[inputs.cpu]]
    percpu = false
    totalcpu = true

  [[inputs.mem]]

  [[outputs.file]]
    files = ["stdout"]
    data_format = "influx"

########################################################################
#                                                                      #
# /telegraf                                                            #
#                                                                      #
########################################################################
```

## Configuring what Telegraf collects and where it writes it

Telegraf refuses to start unless it is given a configuration which defines at least one output plugin, so this role requires you to configure it in one of the two ways described below. They may also be combined, in which case Telegraf merges the configurations. If you configure neither, the role stops with an error rather than installing a service which would restart-loop.

### Providing the configuration file yourself

Set `telegraf_configuration` to the full contents of a [Telegraf configuration file](https://docs.influxdata.com/telegraf/v1/configuration/) (TOML), as in the example above. The role renders it to `/telegraf/telegraf.conf` on the host and mounts it read-only into the container at `/etc/telegraf/telegraf.conf`, replacing the sample configuration which ships in the container image.

The value is passed through to Telegraf verbatim — the role does not merge anything into it, and there is deliberately no separate "extension" variable to append to it. Telegraf's configuration is TOML made out of `[[inputs.x]]`-style array-of-table sections, in which a key appended after the end of the file silently lands in whichever section happens to come last, so appending to it is not a safe thing for a role to offer.

Values from the container's environment can be referenced from the configuration as `${VARIABLE_NAME}`. `telegraf_influx_token` is passed in as `INFLUX_TOKEN`, and anything set through `telegraf_environment_variables_additional_variables` is available too.

>[!NOTE]
> Since Telegraf v1.38.0, an unknown plugin name or an unknown option on a known plugin is a hard error rather than a warning, and Telegraf exits at startup instead of carrying on. If the service does not come up after a configuration change, `journalctl -fu telegraf` reports exactly which option it rejected.

### Set variables for connecting to an InfluxDB instance (optional)

The Telegraf instance can be configured to collect and write metrics to [InfluxDB](https://www.influxdata.com/) or other outputs.

>[!NOTE]
> If you are looking for an Ansible role for InfluxDB, you can check out [this role (ansible-role-influxdb)](https://github.com/mother-of-all-self-hosting/ansible-role-influxdb) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

You can connect the Telegraf instance with the an InfluxDB instance by adding the following configuration to your `vars.yml` file:

```yaml
telegraf_influx_token: YOUR_INFLUXDB_TOKEN_HERE

telegraf_config_link: https://influxdb.example.com/api/v2/telegrafs/0123456789
```

Those values can be retrieved from your InfluxDB instance. To retrieve them, open the instance's URL (`https://influxdb.example.com`) and go to **Load Data** -> **Telegraf**.

### Collecting metrics about the host

The role runs Telegraf in a container with no access to the host beyond what a container normally gets: no `/proc` or `/sys` from the host, no Docker socket, and no `--pid=host`. The `[[inputs.cpu]]`, `[[inputs.mem]]` and similar plugins therefore report what the container can see, which for some of them (`[[inputs.disk]]` in particular) is not the host at all.

This is a deliberate default — an agent that can read the whole host and talk to the Docker socket is a large amount of privilege to hand to something whose job is to make network calls — so widening it is left to you. Telegraf reads the `HOST_PROC`, `HOST_SYS`, `HOST_ETC`, `HOST_VAR`, `HOST_RUN` and `HOST_MOUNT_PREFIX` environment variables to find a mounted host filesystem, so mounting one and pointing Telegraf at it looks like this:

```yaml
telegraf_container_extra_arguments_custom:
  - "--mount type=bind,src=/,dst=/hostfs,ro,bind-propagation=rslave"

telegraf_environment_variables_additional_variables: |
  HOST_MOUNT_PREFIX=/hostfs
  HOST_PROC=/hostfs/proc
  HOST_SYS=/hostfs/sys
  HOST_ETC=/hostfs/etc
```

Give the container only what the inputs you enable actually need. Plugins such as `[[inputs.ping]]` additionally need capabilities the role drops (`--cap-add=NET_RAW`), and `[[inputs.docker]]` needs the Docker socket mounted, which is equivalent to giving Telegraf root on the host.

## Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file

Note that Telegraf itself is not configured through environment variables — its settings live in the configuration file. What `telegraf_environment_variables_additional_variables` is for is supplying values that the configuration file then [references as `${VARIABLE_NAME}`](https://docs.influxdata.com/telegraf/v1/configuration/#set-environment-variables), which is how secrets stay out of it.

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
