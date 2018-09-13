System monitoring and notification
======

This section lists system monitoring tools within PFS IPMU system, and points 
to be checked for operation and error detection. 

* `Statistics`_
* `Alerting`_

Statistics
------

`Prometheus <https://pfs.ipmu.jp/internal/monitors/prometheus/>`_ is used for 
data gatherer of monitoring, syslog is gathered for analysis using 
`Elasticsearch <https://pfs.ipmu.jp/internal/monitors/elasticsearch/>`_ 
with data supplied through rsyslog server which is also used for archive 
of syslog lines, 
and `grafana <https://pfs.ipmu.jp/internal/monitors/grafana/>`_ is used for 
visualization. 

In addition to standard and/or service provided dashboards for 
grafana, some customized dashboards are provided. 

* `Service status <https://pfs.ipmu.jp/internal/monitors/grafana/d/000000014/service-status>`_

  * Downtime fraction are shown.
  * Targets with low non-zero fraction could be temporal high load on targets 
    but not real service down. Timeout of data gathering is set in prometheus, 
    and targets are marked as down when HTTP response has not finished within 
    timeout period. 

* `Host monitoring <https://pfs.ipmu.jp/internal/monitors/grafana/d/000000008/host-stats-prometheus-node-exporter>`_

  * Filesystem panels include ones via NFS. 
  * Last two panels ('Temp' and 'Power') are for physical host, values taken 
    via ipmi. Not available for VM guests. 
  * 'Network' panel shows all network interfaces except local ('lo'), ones 
    for VM guests ('vnet*') are also shown on VM host. 

* `Network interface status and flow <https://pfs.ipmu.jp/internal/monitors/grafana/d/000000009/network>`_

  * Top panel shows status of all ports in monitoring. Check transition of 
    ports, like online to disconnected, on this panel. 
  * All others shows network flow over time. Data sources are host for upper 
    ones and switch for lower ones. 
  * Use `weathermap <https://pfs.ipmu.jp/internal/monitors/weathermap/>`_ 
    for mapped flow. 

* `Syslog <https://pfs.ipmu.jp/internal/monitors/grafana/d/000000012/syslog>`_

  * Statistics of lines per host and severity are shown as top two panels. 
  * Raw lines are shown in lower three panels, but use elasticsearch for 
    detailed analysis. 

* `RAID status <https://pfs.ipmu.jp/internal/monitors/grafana/d/PHT16RKiz/hpraid>`_

  * Only HP RAID cards are listed for now. 

Alerting
------

`Prometheus alerts <https://pfs.ipmu.jp/internal/monitors/prometheus/alerts>`_ 
are pushed to `alertmanager <https://pfs.ipmu.jp/internal/monitors/alertmanager/>`_, 
and delivered to alert targets. 
Alertmanager site lists active alerts, and administrator can add/manage 
silence of alerts using alertmanager. 

Syslog monitoring
------

rsyslog server does not save any syslog line (except for locally generated) 
to local storage, but just proxy all syslog lines to elasticsearch for now. 
To query syslog lines for trouble shooting, use grafana panel for syslog or 
send query to elasticsearch API. 

Note, to enable file based output of syslog lines for future, it is better to 
mount remote storage and write to there, to have some script to compress old 
log files (current output is by host/category/severity/date, but not using 
logrotate), and to have separate stream for local and remote not to have 
local loop. 

Misc
----

* DHCP leased IP/host pairs are listed in `/var/lib/misc/dnsmasq.leases`
* Database statuses are gathered into prometheus, also 

  * slow log for mysql is out to local file `/var/log/mysql/mysql-slow.log`

