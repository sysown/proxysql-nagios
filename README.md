Introduction
============

ProxySQL is a high performance, high availability, protocol aware proxy for MySQL and forks (like Percona Server and MariaDB).
All the while getting the unlimited freedom that comes with a GPL license.

Its development is driven by the lack of open source proxies that provide high performance.  

Official website: http://www.proxysql.com/  
Benchmarks and old blog posts can be found at http://www.proxysql.blogspot.com/  
Forum: https://groups.google.com/forum/#proxysql/  
Wiki: https://github.com/sysown/proxysql/wiki/  
Linkedin group: https://www.linkedin.com/groups/13581070/

This repository contains `proxysql-nagios`, a script for monitoring ProxySQL using Nagios (or a Nagios drop in replacement such
as Icinga or Shinken). In addition, you have the option of outputting regular `--text` format as well as to log to a `--logfile`
in order to use this for a different monitoring tool or log monitoring.

The script has the ability to monitor the following:
- Global status variable values, either higher or lower than the specified thresholds
- Connection pool usage (%) per server
- Hostgroup availability (i.e. number of available servers per hostgroup)
- Query rule configuration (i.e. compare RUNTIME to DISK persisted)
- Service uptime status

The `proxysql-nagios` script has a lot of options detailed viewable using the `--help` switch:

```bash
$ ./proxysql-nagios --help
usage: proxysql-nagios [-h] [-u USER] [-p PASSWD] [-H HOST] [-P PORT]
                       [-d DEFAULTS_FILE] -t {conns,hg,rules,status,var}
                       [-n VAR_NAME] [-l] [-r] [-w WARN_THRESH]
                       [-c CRIT_THRESH] [-i INCLUDE_HG] [-g IGNORE_HG]
                       [--log-file LOGFILE] [--text]

ProxySQL Nagios Check version 1.0.0

Options:
  -h, --help            show this help message and exit
  -u USER, --user USER  ProxySQL admin username (default=admin)
  -p PASSWD, --password PASSWD
                        ProxySQL admin password (default=admin)
  -H HOST, --host HOST  ProxySQL hostname / IP (default=127.0.0.1)
  -P PORT, --port PORT  ProxySQL admin port (default=6032)
  -d DEFAULTS_FILE, --defaults-file DEFAULTS_FILE
                        ProxySQL defaults file (include the path to the file
                        location and specify [proxysql-nagios] group)
  -t {conns,hg,rules,status,var}, --type {conns,hg,rules,status,var}
                        ProxySQL check type (one of conns,hg,rules,status,var)
  -n VAR_NAME, --name VAR_NAME
                        ProxySQL variable name to check
  -l, --lower           Alert if ProxySQL value are LOWER than defined WARN /
                        CRIT thresholds (only applies to "var" check type
  -r, --runtime         Force ProxySQL Nagios check to query the
                        runtime_mysql_XXX tables rather than the mysql_XXX
                        tables, although this is more correct it can lead to
                        excessive logging in ProxySQL and needs to be
                        explicitely enabled (applies to "conns", "hg" and
                        "rules" check types)
  -w WARN_THRESH, --warning WARN_THRESH
                        Warning threshold
  -c CRIT_THRESH, --critical CRIT_THRESH
                        Critical threshold
  -i INCLUDE_HG, --include-hostgroup INCLUDE_HG
                        ProxySQL hostgroup(s) to include (only applies to "--
                        type hg" checks, accepts comma-separated list)
  -g IGNORE_HG, --ignore-hostgroup IGNORE_HG
                        ProxySQL hostgroup(s) to ignore (only applies to "--
                        type hg" checks, accepts comma-separated list)
  --log-file LOGFILE    File to log to (default = stdout)
  --text                Disable Nagios output mode and output pure text
```

