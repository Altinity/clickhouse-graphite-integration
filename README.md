# clickhouse-graphite-integration

# Table of Contents

 * [Introduction](#introduction---what-are-we-talking-about)
 * [Install Graphite](#install-graphite)
 * [Install ClickHouse](#install-clickhouse)
 * [Setup ClickHouse - Graphite integration](#setup-clickhouse---graphite-integration)
 * [Monitoring](#monitoring)

# Introduction - what are we talking about
We'd like to monitor ClickHouse's status, preferably with some tool, which can provide nice on-the-fly graphics.
[Graphite](https://graphiteapp.org/) is one of such tools available, so we'd use it to receive and display metrics, coming from ClickHouse.

Graphite - what is it? Graphite is an enterprise-scale monitoring tool, and as we'd like to monitor Clickhouse's health, we need to report to Graphite some kind of health metrics.
Let's setup Graphite, Clickhouse, integration between them and take a look on monitoring.

# Install Graphite
Graphite installation is [described in details in its manual](http://graphite.readthedocs.io/en/latest/install.html)
Feel free to choose any type of installation.

# Install Clickhouse
ClickHouse installation is explained in several sources, such as:
 * for [deb-based systems](https://clickhouse.yandex/docs/en/getting_started/#installing-from-packages-debianubuntu)
 * for [rpm-based systems](https://github.com/Altinity/clickhouse-rpm-install)

# Setup ClickHouse - Graphite integration
Setup ClickHouse to report metrics into Graphite. Edit `/etc/clickhouse-server/config.xml` and append something like the following:
```xml
    <graphite>
        <host>192.168.74.150</host>
        <port>2003</port>
        <timeout>0.1</timeout>
        <interval>60</interval>
        <root_path>one_min_cr_plain</root_path>

        <metrics>true</metrics>
        <events>true</events>
        <asynchronous_metrics>true</asynchronous_metrics>
    </graphite>
    <graphite>
        <host>192.168.74.150</host>
        <port>2003</port>
        <timeout>0.1</timeout>
        <interval>1</interval>
        <root_path>one_sec_cr_plain</root_path>

        <metrics>true</metrics>
        <events>true</events>
        <asynchronous_metrics>false</asynchronous_metrics>
    </graphite>
```
Settings description:
 * `host` – host where Graphite is running.
 * `port` – Carbon plain text receiver port (2003 is default in Graphite). In general, Graphite has multiple ports open, with the following default values:
   * `80` nginx
   * `2003` carbon receiver - plaintext **this one should receive data from ClickHouse**
   * `2004` carbon receiver - pickle
   * `2023` carbon aggregator - plaintext
   * `2024` carbon aggregator - pickle
   * `8080` Graphite internal gunicorn port (without Nginx proxying).
   * `8125` statsd
   * `8126` statsd admin
 * `interval` – interval for sending data from ClickHouse, in seconds.
 * `timeout` – timeout for sending data, in seconds.
 * `root_path` – prefix used by Graphite.
 * `metrics` – should data from `system_tables-system.metrics` table be sent.
 * `events` – should data from `system_tables-system.events` table be sent.
 * `asynchronous_metrics` – should data from `system_tables-system.asynchronous_metrics` table be sent.

Multiple `<graphite>` clauses can be configured for sending different data at different intervals.

# Monitoring

Navigate to Graphite web monitoring tool in browser as `http://host/dashboard` and you'll see Graphite metrics. 
In case all is well and ClickHouse sends its data, you'll see entries with prefixes, specified in ClickHouse's configuration, which is

```xml
<root_path>one_sec_cr_plain</root_path>
```
in our case. You should see metrics available and you can choose graphs to watch.

![Graphite screenshot](images/graphite_web.png?raw=true "Graphite screenshot")

