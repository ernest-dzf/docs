# Mariadb GTID

## `gtid_binlog_pos`

> This variable is the GTID of the last event group written to the binary log, for each replication domain.

## `gtid_slave_pos`

> This system variable contains the GTID of the last transaction applied to the database by the server's slave threads for each replication domain

## `gtid_current_pos`

> This system variable contains the GTID of the last transaction applied to the database for each replication domain.

## `gtid_binlog_state`

> The variable gtid_binlog_state holds the internal state of the binlog.