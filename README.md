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

This repository contains `proxysql-nagios`, a script for monitoring ProxySQL using Nagios (or a Nagios drop in replacement such as Icinga or Shinken). In addition, you have the option of outputting regular `--text` format as well as to log to a `--logfile` in order to use this for a different monitoring tool or log monitoring.

The script has the ability to monitor the following:
- Global status variable values, either higher or lower than the specified thresholds
- Connection pool usage (%) per server
- Hostgroup availability (i.e. number of available servers per hostgroup)
- Query rule configuration (i.e. compare RUNTIME to DISK persisted)
- Service uptime status

```
## You may first need to install system Python & MySQL dev packages as well as MySQL-python 
sudo apt install python-dev libmysqlclient-dev
sudo pip install MySQL-python

# or alternatively just the system package if you prefer not to use `pip`
sudo apt install python-mysqldb
```

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

Below you can also find a sample Nagios configuration file for a set of three ProxySQL hosts (10.10.10.1 / 10.10.10.2 / 10.10.10.3). You will either need to specify the ProxySQL username / password on the defined `COMMAND` or you will need to use a MySQL defaults file such as the one specified (NOTE: you will need to enable a remote admin user as described [here](https://github.com/sysown/proxysql/wiki/Global-variables#admin-admin_credentials) so that this check can query ProxySQL remotely else you should configure this check via NRPE using the local `admin` ProxySQL user):

##### MySQL defaults file for the following example:
```
[proxysql-nagios]
user=radmin
password=radmin
```

##### Nagios config file excerpt... (not complete):
```
### NAGIOS HOST DEFINITION:

define host{
        use                     proxysql-nodes
        host_name               web-server-1
        alias                   web-server-1
        address                 10.10.10.1
        _padmin_port            6032
        _ssh_port               22
        }

define host{
        use                     proxysql-nodes
        host_name               web-server-2
        alias                   web-server-2
        address                 10.10.10.2
        _padmin_port            6032
        _ssh_port               22
        }

define host{
        use                     proxysql-nodes
        host_name               web-server-3
        alias                   web-server-3
        address                 10.10.10.3
        _padmin_port            6032
        _ssh_port               22
        }

### NAGIOS HOSTGROUP DEFINITION:

define hostgroup{
        hostgroup_name          proxysql-nodes
        alias                   ProxySQL Nodes
        members                 web-server-1,web-server-2,web-server-3
        }

define host{
        name                            proxysql-nodes
        notifications_enabled           1                       ; Host notifications are enabled
        event_handler_enabled           1                       ; Host event handler is enabled
        flap_detection_enabled          1                       ; Flap detection is enabled
        failure_prediction_enabled      1                       ; Failure prediction is enabled
        process_perf_data               1                       ; Process performance data
        retain_status_information       1                       ; Retain status information across program restarts
        retain_nonstatus_information    1                       ; Retain non-status information across program restarts
        check_period                    24x7                    ; By default, Linux hosts are checked round the clock
        check_interval                  1                       ; Actively check the host every minute
        retry_interval                  1                       ; Schedule host check retries at 1 minute intervals
        max_check_attempts              5                       ; Check each Linux host 5 times (max)
        check_command                   check_host_alive        ; Default command to check Linux hosts
        contact_groups                  admins                  ; Notifications get sent to the admins by default
        notification_period             24x7                    ; Send host notifications at any time
        notification_interval           60                      ; Resend notifications every hour
        notification_options            d,u,r                   ; Only send notifications for specific host states
        register                        0                       ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL HOST, JUST A TEMPLATE!
        }

### NAGIOS SERVICE DEFINITION:

define service{
        use                             pull-alerts
        hostgroup_name                  proxysql-nodes
        service_description             ProxySQL Uptime Check
        check_command                   check_proxysql_uptime
        }

# Active_Transactions is just an example, there are other variables you may which to monitor.
# Remember to define the `--lower` switch in the COMMAND if you want to monitor when a variable dips below a
# specified threshold.
define service{
        use                             pull-alerts
        hostgroup_name                  proxysql-nodes
        service_description             ProxySQL Active_Transactions Check
        check_command                   check_proxysql_status_var!Active_Transactions!32!64
        }

define service{
        use                             pull-alerts
        hostgroup_name                  proxysql-nodes
        service_description             ProxySQL Connection Pool Usage
        check_command                   check_proxysql_conn_pool!95!98
        }

define service{
        use                             pull-alerts
        hostgroup_name                  proxysql-nodes
        service_description             ProxySQL Reader Hostgroup Availability
        check_command                   check_proxysql_hg!1!0!2
        }

define service{
        use                             pull-alerts
        hostgroup_name                  proxysql-nodes
        service_description             ProxySQL Writer Hostgroup Availability
        check_command                   check_proxysql_hg!0!0!1
        }

define service{
        use                             pull-alerts
        hostgroup_name                  proxysql-nodes
        service_description             ProxySQL Query Rule Configuration
        check_command                   check_proxysql_query_rules
        }

### NAGIOS COMMAND DEFINITION:

define command{
        command_name    check_proxysql_uptime
        command_line    $USER1$/proxysql-nagios -H $HOSTADDRESS$ -P $_HOSTPADMIN_PORT$ -d /etc/nagios/proxysql.cnf -t status
        }

define command{
        command_name    check_proxysql_status_var
        command_line    $USER1$/proxysql-nagios -H $HOSTADDRESS$ -P $_HOSTPADMIN_PORT$ -d /etc/nagios/proxysql.cnf -t var -n $ARG1$ -w $ARG2$ -c $ARG3$
        }

define command{
        command_name    check_proxysql_conn_pool
        command_line    $USER1$/proxysql-nagios -H $HOSTADDRESS$ -P $_HOSTPADMIN_PORT$ -d /etc/nagios/proxysql.cnf -t conns -w $ARG1$ -c $ARG2$
        }

define command{
        command_name    check_proxysql_hg
        command_line    $USER1$/proxysql-nagios -H $HOSTADDRESS$ -P $_HOSTPADMIN_PORT$ -d /etc/nagios/proxysql.cnf -t hg -w $ARG1$ -c $ARG2$ --include $ARG3$
        }

define command{
        command_name    check_proxysql_query_rules
        command_line    $USER1$/proxysql-nagios -H $HOSTADDRESS$ -P $_HOSTPADMIN_PORT$ -d /etc/nagios/proxysql.cnf -t rules --runtime
        }
```
