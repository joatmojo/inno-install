Changes in MySQL 8.0.26 (2021-07-20)

     * Audit Log Notes

     * Authentication Notes

     * Compilation Notes

     * Component Notes

     * Deprecation and Removal Notes

     * Error Handling

     * Event Scheduler Notes

     * Firewall Notes

     * Packaging Notes

     * Pluggable Authentication

     * Server Administration

     * Spatial Data Support

     * X Plugin Notes

     * Functionality Added or Changed

     * Bugs Fixed

Audit Log Notes

     * Previously, each event logged by MySQL Enterprise Audit
       included the SQL statement literal text. To provide an
       alternative (because it possible that statements contain
       sensitive information), the audit log filtering language
       now supports logging a statement's digest rather than its
       literal text. For example, instead of logging this
       statement:

SELECT * FROM orders WHERE some_sensitive_column=1234567

       The audit log plugin can log this digest:

SELECT * FROM `orders` WHERE `some_sensitive_column` = ?

       This is similar to what is already logged for prepared
       statements, for which parameter markers appear rather
       than actual data values.

       To perform digest logging, use audit filter definitions
       that replace the statement literal text by its
       corresponding digest, as discussed in Replacement of
       Event Field Values
(https://dev.mysql.com/doc/refman/8.0/en/audit-log-filter-definitions.html#audit-log-filtering-field-replacement).

       Because text replacement occurs at an early auditing
       stage (during filtering), the choice of whether to log
       statement literal text or digest values applies
       regardless of log format written later (that is, whether
       the audit log plugin produces XML or JSON output). (Bug
       #31482609)

     * For MySQL Enterprise Audit, the new
       audit_log_format_unix_timestamp system variable enables
       inclusion of a time field in each audit record. The field
       value is an integer that represents the UNIX timestamp
       value indicating the date and time when the audit event
       was generated. The time field is supported only for
       JSON-format log files.

     * For MySQL Enterprise Audit, the new audit_log_max_size
       system variable enables audit log file pruning based on
       combined log file size. To have an effect,
       audit_log_max_size requires that audit_log_rotate_on_size
       be greater than 0. If that is true, the pruning algorithm
       uses audit_log_max_size in conjunction with
       audit_log_prune_seconds, with nonzero values of
       audit_log_max_size taking precedence over nonzero values
       of audit_log_prune_seconds. For details, see Space
       Management of Audit Log Files
(https://dev.mysql.com/doc/refman/8.0/en/audit-log-logging-configuration.html#audit-log-space-management).

Authentication Notes

     * Previously, as part of the "hello" packet sent by the
       server to clients, the server sent the name of the
       server-side authentication plugin rather than the
       client-side plugin. The server now sends the client-side
       name, which is more appropriate for the client's needs
       and may help to avoid extra protocol round trips.

Compilation Notes

     * macOS: It is now possible to build MySQL for macOS 11 on
       ARM (that is, for Apple M1 systems). (Bug #32386050, Bug
       #102259)

     * Building on openSUSE 15 and SLES 15 now requires GCC 9,
       found in packages gcc-9 and gcc9-c++.

       Building on SLES 12 now requires GCC 10, found in
       packages gcc-10 and gcc10-c++.

       It is also recommended to use the named GCC version when
       building third-party applications that are based on the
       libmysqlclient C API library. (Bug #32886268, Bug
       #32886439)

     * Building on Ubuntu 18.04 (bionic) now requires GCC 8,
       found in packages gcc-8 and g++-8. It is also recommended
       to use GCC 8 when building third-party applications that
       are based on the libmysqlclient C API library. (Bug
       #32877062)

     * It is now possible to build MySQL on Solaris using GCC
       10, which becomes the default and recommended compiler.
       It is also recommended to use GCC 10 when building
       third-party applications that are based on the
       libmysqlclient C API library. (Bug #32552988)

Component Notes

     * A new component service enables server components to set
       system variable values. For information about this
       service, see the MySQL Server Doxygen documentation,
       available at https://dev.mysql.com/doc/index-other.html
       (search for s_mysql_mysql_system_variable_update_string
       and mysql_system_variable_update_string_imp).

Deprecation and Removal Notes

     * The TLSv1 and TLSv1.1 connection protocols now are
       deprecated and support for them is subject to removal in
       a future MySQL version. (For background, refer to the
       IETF memo Deprecating TLSv1.0 and TLSv1.1
(https://tools.ietf.org/id/draft-ietf-tls-oldversions-deprecate-02.html).)
       It is recommended that connections be
       made using the more-secure TLSv1.2 and TLSv1.3 protocols.
       TLSv1.3 requires that both the MySQL server and the
       client application be compiled with OpenSSL 1.1.1 or
       higher.

       On the server side, this deprecation has the following
       effects:

          + If the tls_version or admin_tls_version system
            variable is assigned a value containing a deprecated
            TLS protocol, the server produces a warning for each
            deprecated protocol:

               o If the assignment occurs during server startup,
                 the warning appears in the error log.

               o If the assignment occurs at runtime, the
                 warning is added to the result of executing the
                 ALTER INSTANCE RELOAD TLS statement.

          + If a client successfully connects using a deprecated
            TLS protocol, the server writes a warning to the
            error log.

       On the client side, the deprecation has no visible
       effect. Clients do not issue a warning if configured to
       permit a deprecated TLS protocol. This includes:

          + Client programs that support a --tls-version option
            for specifying TLS protocols for connections to the
            MySQL server.

          + Statements that enable replicas to specify TLS
            protocols for connections to the source server.
            (CHANGE REPLICATION SOURCE TO has a
            SOURCE_TLS_VERSION option and CHANGE MASTER TO has a
            MASTER_TLS_VERSION option.)

          + The group_replication_recovery_tls_version system
            variable that enables joining members to specify TLS
            protocols for distributed recovery connections.

       (Bug #32565996)

     * The temptable_use_mmap variable is now deprecated and
       subject to removal in a future MySQL version.

     * TLS support in MySQL has been moving toward a channel
       model using named sets of TLS parameters that apply to
       different securable ports or protocols. For example, to
       query the state of a particular TLS channel, use the
       Performance Schema tls_channel_status table:

mysql> SELECT VALUE FROM performance_schema.tls_channel_status
       WHERE CHANNEL = 'mysql_main' AND PROPERTY = 'Enabled';
+-------+
| VALUE |
+-------+
| Yes   |
+-------+

       This makes monolithic parameters that apply to TLS
       support as a whole less applicable, so the following
       options and system variables are now deprecated and
       subject to removal in a future MySQL version:

          + The --ssl and --admin-ssl server options.

          + The have_ssl and have_openssl system variables.

       The --ssl and --admin-ssl options are enabled by default,
       so it is normally unnecessary to specify them. As an
       alternative to specifying those options in negated form,
       if it is desired to disable encrypted connections for the
       main or administrative interface, set the corresponding
       TLS version system variable to the empty value to
       indicate that no TLS versions are supported. For example,
       these lines in the server my.cnf file disable encrypted
       connections for both interfaces:

[mysqld]
tls_version=''
admin_tls_version=''

Error Handling

     * Information written to the server error log for client
       timeouts now includes (if available) the timeout value,
       and client user and host. (Bug #31581289, Bug #100112)

Event Scheduler Notes

     * If the Event Scheduler is enabled, enabling the
       super_read_only system variable prevents it from updating
       event "last executed" timestamps in the events data
       dictionary table. This causes the Event Scheduler to stop
       the next time it tries to execute a scheduled event,
       after writing a message to the server error log.

       Previously, if enabling super_read_only caused the Event
       Scheduler to stop, then after subsequently disabling
       super_read_only, it was necessary to manually restart the
       Event Scheduler by enabling it again. As a convenience,
       the server now automatically restarts the Event Scheduler
       as needed when super_read_only is disabled. (Bug
       #31633859)

Firewall Notes

     * In MySQL 8.0.23, MySQL Enterprise Firewall implemented
       group profiles that each can apply to multiple accounts,
       in addition to the previously implemented account
       profiles that each apply to a single account. See Using
       MySQL Enterprise Firewall
       (https://dev.mysql.com/doc/refman/8.0/en/firewall-usage.html).

       A group profile with a single member account is logically
       equivalent to an account profile for that account, so it
       is possible to administer the firewall using group
       profiles exclusively, rather than a mix of account and
       group profiles. For new firewall installations, that is
       accomplished by uniformly creating new profiles as group
       profiles and avoiding account profiles. For upgrades from
       firewall installations that already contain account
       profiles, MySQL Enterprise Firewall now includes a stored
       procedure named sp_migrate_firewall_user_to_group() for
       converting account profiles to group profiles.

       Due to the greater flexibility offered by group profiles,
       all aspects of the firewall related to account profiles
       are now deprecated and subject to removal in a future
       MySQL version:

          + INFORMATION_SCHEMA tables: MYSQL_FIREWALL_USERS,
            MYSQL_FIREWALL_WHITELIST

          + mysql system schema tables: firewall_users,
            firewall_whitelist

          + mysql system schema stored procedures:
            sp_reload_firewall_rules(), sp_set_firewall_mode()

          + Loadable functions: read_firewall_users(),
            read_firewall_whitelist(), set_firewall_mode()

       Additionally, if the server detects account profiles at
       startup, it writes a warning for every successfully
       loaded account profile.

       For information about converting account profiles to
       group profiles (which you should do at your earliest
       convenience), see Migrating Account Profiles to Group
       Profiles
(https://dev.mysql.com/doc/refman/8.0/en/firewall-usage.html#firewall-account-profile-migration).

Packaging Notes

     * Binary packages that include curl rather than linking to
       the system curl library have been upgraded to use curl
       7.77.0. (Bug #33077562)

     * For Ubuntu packages, the AppArmor profile for mysqld was
       too restrictive regarding PID and socket file names and
       failed for servers not using the exact names in the
       profile. Now the profile applies to the directories in
       which the files live, enabling it to apply for different
       file names, and multiple servers. (Bug #32857611)

     * The dh-systemd package has been removed from Ubuntu
       21.04, so the dependency on it has been removed from
       MySQL packages built for that distribution. (Bug
       #32688072)

     * For Debian packages, an EnvironmentFile directive was
       added to enable the systemd service to read environmental
       variables from the /etc/default/mysql file if it is
       present. (Bug #32082863, Bug #101363)

     * Debian packages now use /run rather than /var/run for
       path names. (Bug #31955638)

     * The bundled lz4 library was upgraded to version 1.9.3.
       (Bug #29747853)

     * For Debian packages, the update-alternatives priority of
       the MySQL configuration file was increased to ensure it
       replaces an existing file from a previously installed
       distribution. (Bug #29606955)

Pluggable Authentication

     * Linux: MySQL Enterprise Edition now supports an
       authentication method that enables users to authenticate
       to MySQL Server using Kerberos, provided that appropriate
       Kerberos tickets are available or can be obtained. For
       details, see Kerberos Pluggable Authentication
(https://dev.mysql.com/doc/refman/8.0/en/kerberos-pluggable-authentication.html).

       This authentication method uses a pair of plugins,
       authentication_kerberos on the server side and
       authentication_kerberos_client on the client side. It is
       available only on MySQL server and client hosts running
       Linux, but can access Kerberos services running on
       non-Linux hosts.

Server Administration

     * Setting the session value of the innodb_strict_mode
       system variable is now a restricted operation and the
       session user must have privileges sufficient to set
       restricted session variables.

       For information about the privileges required to set
       restricted session variables, see System Variable
       Privileges
       (https://dev.mysql.com/doc/refman/8.0/en/system-variable-privileges.html).
       (Bug #32944980)

Spatial Data Support

     * The ST_Buffer() function now permits the geometry
       argument to have a geographic spatial reference system
       (SRS), if the geometry is a point value. Previously,
       ST_Buffer() supported only geometry arguments in a
       Cartesian SRS. The ST_Difference() and ST_Union()
       functions now permit the geometry arguments to have a
       geographic SRS. Previously, ST_Difference() and
       ST_Union() supported only geometry arguments in a
       Cartesian SRS. See Spatial Operator Functions
(https://dev.mysql.com/doc/refman/8.0/en/spatial-operator-functions.html).

X Plugin Notes

     * When the X DevAPI Session.run_sql() method was used to
       execute a query that returned multiple results, due to a
       caching issue, the result.columns property was not
       updated to reflect the columns present in the active
       result, although the result.column_names property was.
       (Bug #32887586)

     * During an upgrade process, X Plugin logged a message
       stating that it was ready for connections once the TCP
       port and UNIX socket had been allocated. However,
       connections could not actually be accepted until after
       the upgrade process was complete. The message is now
       issued only after the upgrade has finished. (Bug
       #32814997)

Functionality Added or Changed

     * Incompatible Change: From MySQL 8.0.26, new aliases or
       replacement names are provided for most remaining
       identifiers that contain the terms "master", which is
       changed to "source"; "slave", which is changed to
       "replica"; and "mts" (for "multithreaded slave"), which
       is changed to "mta" (for "multithreaded applier"). Help
       text is also changed where applicable to use the new
       names.

       The following name replacements are visible in the
       Performance Schema tables, the process list, and the
       replica status information. These changes are
       incompatible with earlier releases. Monitoring tools that
       work with these instrumentation names might be impacted:

          + Instrumented locks (mutexes), visible in the
            mutex_instances and events_waits_* Performance
            Schema tables with the prefix wait/synch/mutex/

          + Read/write locks, visible in the rwlock_instances
            and events_waits_* Performance Schema tables with
            the prefix wait/synch/rwlock/

          + Instrumented condition variables, visible in the
            cond_instances and events_waits_* Performance Schema
            tables with the prefix wait/synch/cond/

          + Instrumented memory allocations, visible in the
            memory_summary_* Performance Schema tables with the
            prefix memory/sql/

          + Thread names, visible in the threads Performance
            Schema table with the prefix thread/sql/

          + Thread stages, visible in the events_stages_*
            Performance Schema tables with the prefix
            stage/sql/, and without the prefix in the threads
            and processlist Performance Schema tables, the
            output from the SHOW PROCESSLIST statement, the
            Information Schema processlist table, and the slow
            query log

          + Thread commands, visible in the
            events_statements_history* and
            events_statements_summary_*_by_event_name
            Performance Schema tables with the prefix
            statement/com/, and without the prefix in the
            threads and processlist Performance Schema tables,
            the output from the SHOW PROCESSLIST statement, the
            Information Schema processlist table, and the output
            from the SHOW REPLICA STATUS statement

       If the incompatible changes do have an impact for you,
       you can set the new system variable
       terminology_use_previous to BEFORE_8_0_26 to make MySQL
       Server use the old versions of the names for the objects
       specified in the previous list. This enables monitoring
       tools that rely on the old names to continue working
       until they can be updated to use the new names. The
       system variable can be set with session scope to support
       individual functions, or global scope to be a default for
       all new sessions. When global scope is used, the slow
       query log contains the old versions of the names.

       For semisynchronous replication, you can choose whether
       to use the new or the old versions of the system
       variables and status variables. New versions of the
       plugins that implement semisynchronous replication, one
       for the source server and one for the replica, are
       supplied that replace the terms master and slave with
       source and replica, and you can install these versions
       instead of the old ones:

          + The rpl_semi_sync_master plugin (semisync_master.so
            library) for the source has a new version
            rpl_semi_sync_source (semisync_source.so library)

          + The rpl_semi_sync_slave plugin (semisync_slave.so
            library) for the replica has a new version
            rpl_semi_sync_replica (semisync_replica.so library)

       You cannot have both the new and the old version of the
       relevant plugin installed on an instance. If you use the
       new version of the plugins, the new system variables and
       status variables are available but the old ones are not.
       If you use the old version of the plugins, the old system
       variables and status variables are available but the new
       ones are not.

       The following internal-use items are converted to use the
       new terms but are not externalized to users or monitoring
       tools, and MySQL Server handles any necessary resolution
       internally:

          + C++ filenames in source code

          + Header guards in C++ files

          + Debug symbols

          + User variables passed in the replication protocol
            handshake by the replica when it connects to a
            replication source server (the replica sets both the
            old and the new name)

       The following categories of identifiers have a new alias,
       and a deprecation warning is issued when the old name is
       used, although the old name continues to work. Both names
       are available in Performance Schema tables and status
       displays, and no deprecation warning is issued when
       reading these. The new aliases are not affected by the
       new system variable terminology_use_previous, and can
       still be used when it is set:

          + System variables that contain the terms "master",
            "slave", or "mts", with the exception of some that
            have already been deprecated or are scheduled for
            deprecation, and those defined by NDB. If these
            system variables are persisted using a SET PERSIST
            statement, both the old and the new name are
            persisted, regardless of which was specified in the
            statement. With a RESET PERSIST statement, both are
            reset.

          + Status variables that contain the terms "master",
            "slave", or "mts", with the exception of those
            defined by NDB.

          + Command-line options for mysqld that contain the
            terms "master", "slave", or "mts", with the
            exception of some that have already been deprecated
            or are scheduled for deprecation, and those defined
            by NDB.

          + Command-line options for mysqladmin that contain the
            terms "master", "slave", or "mts".

          + Command-line options for mysqlbinlog that contain
            the terms "master", "slave", or "mts".

          + Command-line options for mysqldump that contain the
            terms "master", "slave", or "mts".

          + An SQL function that contains the term "master".

       The complete list of identifiers with new aliases (or in
       the case of semisynchronous replication, replacements) is
       as follows:

          + System variables:

               o master_verify_checksum now has the alias
                 source_verify_checksum

               o sync_master_info now has the alias
                 sync_source_info

               o init_slave now has the alias init_replica

               o rpl_stop_slave_timeout now has the alias
                 rpl_stop_replica_timeout

               o log_slow_slave_statements now has the alias
                 log_slow_replica_statements

               o slave_max_allowed_packet now has the alias
                 replica_max_allowed_packet

               o slave_compressed_protocol now has the alias
                 replica_compressed_protocol

               o slave_exec_mode now has the alias
                 replica_exec_mode

               o slave_type_conversions now has the alias
                 replica_type_conversions

               o slave_sql_verify_checksum now has the alias
                 replica_sql_verify_checksum

               o slave_parallel_type now has the alias
                 replica_parallel_type

               o slave_preserve_commit_order now has the alias
                 replica_preserve_commit_order

               o log_slave_updates now has the alias
                 log_replica_updates

               o slave_allow_batching now has the alias
                 replica_allow_batching

               o slave_load_tmpdir now has the alias
                 replica_load_tmpdir

               o slave_net_timeout now has the alias
                 replica_net_timeout

               o sql_slave_skip_counter now has the alias
                 sql_replica_skip_counter

               o slave_skip_errors now has the alias
                 replica_skip_errors

               o slave_checkpoint_period now has the alias
                 replica_checkpoint_period

               o slave_checkpoint_group now has the alias
                 replica_checkpoint_group

               o slave_transaction_retries now has the alias
                 replica_transaction_retries

               o slave_parallel_workers now has the alias
                 replica_parallel_workers

               o slave_pending_jobs_size_max now has the alias
                 replica_pending_jobs_size_max

               o pseudo_slave_mode now has the alias
                 pseudo_replica_mode

               o skip_slave_start now has the alias
                 skip_replica_start

          + If the new rpl_semi_sync_source and
            rpl_semi_sync_replica plugins are used for
            semisynchronous replication:

               o rpl_semi_sync_slave_enabled is replaced by
                 rpl_semi_sync_replica_enabled

               o rpl_semi_sync_slave_trace_level is replaced by
                 rpl_semi_sync_replica_trace_level

               o rpl_semi_sync_master_wait_for_slave_count is
                 replaced by
                 rpl_semi_sync_source_wait_for_replica_count

               o rpl_semi_sync_master_enabled is replaced by
                 rpl_semi_sync_source_enabled

               o rpl_semi_sync_master_timeout is replaced by
                 rpl_semi_sync_source_timeout

               o rpl_semi_sync_master_trace_level is replaced by
                 rpl_semi_sync_source_trace_level

               o rpl_semi_sync_master_wait_point is replaced by
                 rpl_semi_sync_source_wait_point

          + The following system variables are not changed:

               o ndb_slave_conflict_role (NDB system variables
                 are not changed)

               o binlog_rotate_encryption_master_key_at_startup
                 ("master key" is an accepted term)

               o slave_rows_search_algorithms (This system
                 variable is already deprecated)

               o master_info_repository (This system variable is
                 already deprecated)

          + Status variables:

               o Slave_open_temp_tables now has the alias
                 Replica_open_temp_tables

               o Slave_rows_last_search_algorithm_used now has
                 the alias
                 Replica_rows_last_search_algorithm_used

          + If the new rpl_semi_sync_source and
            rpl_semi_sync_replica plugins are used for
            semisynchronous replication:

               o Rpl_semi_sync_slave_status is replaced by
                 Rpl_semi_sync_replica_status

               o Rpl_semi_sync_master_status is replaced by
                 Rpl_semi_sync_source_status

               o Rpl_semi_sync_master_clients is replaced by
                 Rpl_semi_sync_source_clients

               o Rpl_semi_sync_master_yes_tx is replaced by
                 Rpl_semi_sync_source_yes_tx

               o Rpl_semi_sync_master_no_tx is replaced by
                 Rpl_semi_sync_source_no_tx

               o Rpl_semi_sync_master_wait_sessions is replaced
                 by Rpl_semi_sync_source_wait_sessions

               o Rpl_semi_sync_master_no_times is replaced by
                 Rpl_semi_sync_source_no_times

               o Rpl_semi_sync_master_timefunc_failures is
                 replaced by
                 Rpl_semi_sync_source_timefunc_failures

               o Rpl_semi_sync_master_wait_pos_backtraverse is
                 replaced by
                 Rpl_semi_sync_source_wait_pos_backtraverse

               o Rpl_semi_sync_master_tx_wait_time is replaced
                 by Rpl_semi_sync_source_tx_wait_time

               o Rpl_semi_sync_master_tx_waits is replaced by
                 Rpl_semi_sync_source_tx_waits

               o Rpl_semi_sync_master_tx_avg_wait_time is
                 replaced by
                 Rpl_semi_sync_source_tx_avg_wait_time

               o Rpl_semi_sync_master_net_wait_time is replaced
                 by Rpl_semi_sync_source_net_wait_time

               o Rpl_semi_sync_master_net_waits is replaced by
                 Rpl_semi_sync_source_net_waits

               o Rpl_semi_sync_master_net_avg_wait_time is
                 replaced by
                 Rpl_semi_sync_source_net_avg_wait_time

          + NDB-related status variables are not changed.

          + For mysqld, the command-line versions of all the
            aliased and replaced system variables in the lists
            above have equivalent command-line aliases or
            replacements, plus the following command-line option
            that is not a system variable:

               o show-slave-auth-info has the alias
                 show-replica-auth-info

          + The following command-line options are not changed:

               o abort-slave-event-count (This command-line
                 option is scheduled for deprecation)

               o disconnect-slave-event-count (This command-line
                 option is scheduled for deprecation)

               o master-info-file (This command-line option is
                 already deprecated)

               o master-retry-count (This command-line option is
                 already deprecated)

          + For mysqladmin, the option start-slave now has the
            alias start-replica, and the option stop-slave now
            has the alias stop-replica.

          + For mysqlbinlog, the option read-from-remote-master
            now has the alias read-from-remote-source.

          + For mysqldump, the following command-line options
            have new aliases:

               o apply-slave-statements now has the alias
                 apply-replica-statements

               o delete-master-logs now has the alias
                 delete-source-logs

               o dump-slave now has the alias dump-replica

               o include-master-host-port now has the alias
                 include-source-host-port

               o master-data now has the alias source-data

          + The built-in SQL function MASTER_POS_WAIT has a new
            alias SOURCE_POS_WAIT.

     * InnoDB: The new innodb_segment_reserve_factor system
       variable permits configuring the percentage of tablespace
       file segment pages that are reserved as empty pages. For
       more information, see Configuring the Percentage of
       Reserved File Segment Pages
(https://dev.mysql.com/doc/refman/8.0/en/innodb-file-space.html#innodb-config-reserved-file-segment-pages).

       Thanks to Facebook for the contribution. (Bug #32312743,
       Bug #102044)

     * The URL for downloading Boost was updated. Thanks to
       Marcelo Altmann for the contribution. (Bug #32856104, Bug
       #103611)

     * These statements now report utf8mb3 rather than utf8 when
       writing character set names: EXPLAIN, SHOW CREATE
       PROCEDURE, SHOW CREATE EVENT.

       Stored program definitions retrieved from the data
       dictionary now report utf8mb3 rather than utf8 in
       character set references. This affects any output
       produced from those definitions, such as SHOW CREATE
       statements.

       This error message now reports utf8mb3 rather than utf8
       when writing character set names:
       ER_INVALID_CHARACTER_STRING. (Bug #32233614, Bug
       #32392077, Bug #32392209, Bug #32428538, Bug #32428598)

     * For Group Replication, a group in single-primary mode can
       now be configured to stay in super read-only mode, so
       that it only accepts replicated transactions and does not
       accept any direct writes from clients. This setup means
       that when a group's purpose is to provide a secondary
       backup to another group for disaster tolerance, you can
       ensure that the secondary group remains synchronized with
       the first. You can configure the group to remain in super
       read-only mode when a new primary is elected, by
       disabling the action that normally takes place to remove
       that mode on the primary.

       Administrators can configure a group in this way using
       the new Group Replication functions
       group_replication_enable_member_action and
       group_replication_disable_member_action, which can enable
       and disable actions for members of a group to take in
       specified situations. The functions can also be used on
       servers that are not part of a group, as long as the
       Group Replication plugin is installed. Member actions are
       configured on the primary and propagated to other group
       members and joining members using group messages. Another
       function group_replication_reset_member_actions is
       available to reset the member actions configuration to
       the default setting for all member actions.

     * The system variable transaction_write_set_extraction is
       now deprecated, and a warning message is issued if you
       attempt to set it or read its value. The system variable
       will be removed in a future MySQL version. This system
       variable was used on a replication source server that has
       multithreaded replicas, to specify the algorithm used to
       hash the writes extracted for a transaction's write set.
       The XXHASH64 algorithm, which is the default in MySQL 8.0
       and is required for Group Replication, is selected when
       the system variable is not used.

     * You can now select an alternative UUID to form part of
       the GTIDs that are used when Group Replication's
       internally generated transactions for view changes
       (View_change_log_event) are written to the binary log.
       The new Group Replication system variable
       group_replication_view_change_uuid specifies a UUID that
       is used instead of the group name (the value of the
       group_replication_group_name system variable). The
       alternative UUID makes it easier to distinguish view
       change events from transactions received by the group
       from clients. This can be useful if your setup allows for
       failover between groups, and you need to identify and
       discard transactions that were specific to the backup
       group. Note that all members of the group must have the
       same alternative UUID specified, so groups set up in this
       way cannot include members at releases below MySQL
       8.0.26.

     * On platforms that support fdatasync() system calls, the
       new innodb_use_fdatasync variable permits using
       fdatasync() instead of fsync() for operating system
       flushes. An fdatasync() system call does not flush
       changes to file metadata unless required for subsequent
       data retrieval, providing a potential performance
       benefit. The innodb_use_fdatasync variable can be set
       dynamically using a SET statement.

Bugs Fixed

     * Incompatible Change: Within trigger bodies, INSERT or
       UPDATE statements containing a SET clause that used OLD
       or NEW values as assignment targets could raise an
       assertion or lead to a server exit. Such assignments are
       no longer permitted. (Bug #32803211)

     * Performance: Internal functions used to copy values
       between columns have been improved such that computations
       not necessary when the values are of similar types are no
       longer performed. Queries using temporary tables should
       be noticeably faster with this enhancement. Our internal
       testing has shown such queries being executed up to 11%
       faster than previously; as always, your results may
       differ from these depending on environment,
       configuration, and other factors. (Bug #32742537)

     * InnoDB: To reduce the number of unnecessary warning
       messages in the error log, instances of the
       fil_space_acquire() function in the InnoDB sources were
       replaced by the fil_space_acquire_silent() function where
       possible. (Bug #32944543)

     * InnoDB: The TRX_FORCE_ROLLBACK_ASYNC flag in the InnoDB
       sources, which indicates whether a transaction was rolled
       back asynchronously or by the owning thread, was found to
       be redundant and has been removed. (Bug #32912595)

     * InnoDB: Use of the ut_delete symbol instead of the
       UT_DELETE macro in the InnoDB sources caused a failure in
       builds that disable Performance Schema memory tracing
       (-DDISABLE_PSI_MEMORY=ON). (Bug #32910699)

     * InnoDB: Dictionary system mutex_enter and mutex_exit
       calls in the InnoDB sources were renamed to
       dict_sys_mutex_enter() and dict_sys_mutex_exit(),
       respectively. (Bug #32907980)

     * InnoDB: Legacy UNIV_INLINE and UNIV_MATERIALIZE artifacts
       were removed from InnoDB sources. UNIV_HOTBACKUP was
       added to method declarations in some header files. (Bug
       #32894165)

     * InnoDB: The lock_sys sharded rw_lock index used random
       index values generated by the ut_rnd_interval() function,
       which was not optimal for low-concurrency workloads. (Bug
       #32880577)

     * InnoDB: A string value setting for the
       innodb_redo_log_encrypt variable was not handled
       properly. (Bug #32851525)

     * InnoDB: Read-write transaction set (trx_sys->rw_trx_set)
       shards, each with a dedicated mutex, were introduced to
       alleviate transaction system mutex (trx_sys->mutex)
       contention caused by transaction set insertions and
       removals. Related enhancements include moving transaction
       set modifiers to less critical locations, eliminating
       heap allocation inside of the TrxUndoRsegs constructor,
       converting transaction state (trx->state) and transaction
       start time (trx->start_time) fields to std::atomic
       fields, and new assertion code to validate threads that
       operate on transactions. (Bug #32832196)

     * InnoDB: Record buffer logic for the InnoDB memcached GET
       command was revised. (Bug #32828352)

     * InnoDB: The ut_list base member in the InnoDB sources now
       locates list nodes using the element portion of the list
       type rather than storing a member pointer in the base
       node of a list at runtime, which waisted resources. The
       patch also includes other ut_list related code
       improvements. (Bug #32820458)

     * InnoDB: A deadlock between a user thread and purge thread
       involving a undo log page and rollback segment page
       occurred after an undo tablespace truncate operation was
       initiated. The deadlock caused a long semaphore wait and
       an eventual failure. (Bug #32800020)

     * InnoDB: Type-safe enhancements for PSI_memory_key
       identifiers were introduced. PSI_memory_key identifiers
       are used by Performance Schema for instrumentation of
       memory operations. With this enhancement,
       ut::aligned_name library functions are able to report
       type errors at compile time. (Bug #32797838)

     * InnoDB: The buf_get_LRU_mutex() function was optimized to
       avoid acquiring the LRU mutex unnecessarily when flushing
       from the flush list. (Bug #32797451, Bug #103391)

     * InnoDB: In debug builds, an access to
       Fil_shard::m_deleted_spaces (deleted tablespaces vector)
       was not protected by the Fil_shard mutex, causing a
       failure. (Bug #32792816)

     * InnoDB: Enabling innodb_dedicated_server on a machine
       with 2GB of RAM resulted in a single redo log file being
       created, causing a startup failure. The
       innodb_dedicated_server automated configuration logic was
       revised to ensure that a minimum of two log files are
       created.

       Thanks to Adam Cable for the contribution. (Bug
       #32788772, Bug #103372)

     * InnoDB: A failure occurred when truncating a partitioned
       table after an operation that added too many columns to
       the table, exceeding the column limit. The number of
       columns added is now evaluated before an ADD COLUMN
       operation is permitted. (Bug #32788564, Bug #103363)

     * InnoDB: On platforms that support punch hole where the
       disk is near full, creating a tablespace with a large
       AUTOEXTEND_SIZE setting could lead to a no space on
       device failure and a subsequent InnoDB recovery failures.
       (Bug #32771235)

     * InnoDB: The list of table locks requested by a
       transaction (trx->lock.table_locks), which is a subset of
       the transaction lock list (trx->lock.trx_locks), was
       removed. Table locks requested by a transaction are now
       are now listed at the beginning of the transaction lock
       list instead. (Bug #32762881)

     * InnoDB: A failure occurred during recovery with the disk
       being near full, leaving the data in an inconsistent
       state. The failure occurred in the
       fil_tablespace_redo_extend() function, which is used to
       redo a tablespace extension operation. (Bug #32749974,
       Bug #32748733)

     * InnoDB: After relocating a file-per-table tablespace
       offline and making the new location known to InnoDB using
       the innodb_directories option, an ALTER TABLE operation
       that used the COPY algorithm failed with a storage engine
       error. The failure was due to a renaming check, which
       searched the data directory instead of the new directory
       location. (Bug #32721533)

     * InnoDB: ut_allocator() compliance issues with the C++
       standard template library (STL) were addressed. (Bug
       #32715698)

     * InnoDB: Instances of ut_allocator::allocate()
       instantiated by std::vector in the InnoDB sources failed
       to trace memory allocations and deallocations performed
       implicitly by std::vector. The same issue was found for
       other C++ standard template library (STL) and STL-like
       data-structures. (Bug #32715688)

     * InnoDB: The ut_allocator::construct() interface in the
       InnoDB sources, which is a custom interface implemented
       in a pre-C++11 style, caused unnecessary overhead. The
       interface was not necessary and has been removed. (Bug
       #32715381)

     * InnoDB: The ut_list length member variable in the InnoDB
       sources was replaced by an atomic field to permit lock
       free access without undefined behavior. The default
       ut_list constructor was replaced by a new constructor
       that performs all list initialization. (Bug #32715371)

     * InnoDB: The ut_allocator() out of memory reporting
       mechanism in the InnoDB sources was not reliable and has
       been removed. (Bug #32715359)

     * InnoDB: Implicit handling of Performance Schema metadata
       was implemented for ut_allocator::allocate_large() and
       ut_allocator::deallocate_large() functions in the InnoDB
       sources. This modification aligns Performance Schema
       metadata handling with that of similar allocation
       functions. (Bug #32714144)

     * InnoDB: Stalls were caused by concurrent SELECT COUNT(*)
       queries where the number of parallel read threads
       exceeded the number of machine cores. A patch for this
       issue was provided for Windows builds in MySQL 8.0.24.
       The MySQL 8.0.26 patch addresses the same issue on other
       affected platforms. (Bug #32678019)

       References: See also: Bug #32224707.

     * InnoDB: To avoid costly calls to the rec_get_offsets()
       function, which determines the offsets for each field in
       a record, caching of offsets is extended to indexes that
       meet certain requirements such as having a fixed length,
       no virtual columns, instant columns, and so on. (Bug
       #32673649)

     * InnoDB: When starting the server with a data directory
       that was restored by MySQL Enterprise Backup, the
       doublewrite buffer (controlled by the innodb_doublewrite
       variable) remained disabled until the next server
       restart. (Bug #32642866)

     * InnoDB: "Too many open files" errors were encountered
       when creating a large number of tables. (Bug #32634620)

       References: This issue is a regression of: Bug #32541241.

     * InnoDB: InnoDB recovery was unable to proceed due to a
       page tracking system recovery failure, which should have
       been non-blocking. (Bug #32630875)

     * InnoDB: An integer underflow issue was addressed in the
       InnoDB mecached plugin sources. (Bug #32620378, Bug
       #32620398)

     * InnoDB: When a transaction started waiting for lock, the
       InnoDB lock system provided information to the server
       about the transaction currently holding the lock but
       failed to inform the server after releasing the lock and
       grating it to another waiting transaction. As a result,
       the replication applier thread coordinator was unable to
       detect potential deadlocks in the intended transaction
       commit order that could occur if the third transaction in
       this scenario committed after the initial waiting
       transaction. (Bug #32618301)

     * InnoDB: When inserting a record into a unique secondary
       index, the index record locks taken to prevent concurrent
       transactions from inserting a conflicting record into the
       affected range included an unnecessary gap lock on the
       first record after the range. (Bug #32617942)

     * InnoDB: InnoDB code that creates dynamically allocated
       over-aligned types was replaced by ut::aligned_name
       library functions. (Bug #32601599)

     * InnoDB: Memory allocation functions belonging to the API
       that handles dynamic storage of over-aligned types
       (ut::aligned_name library functions) were extended to
       support memory tracing using Performance Schema. The
       HAVE_PSI_MEMORY_INTERFACE source configuration option
       enables the memory tracing module.

       An aligned_zalloc() library function, which provides
       support for zero-initialized memory allocations, was
       added to the API. (Bug #32600981)

       References: See also: Bug #32601599, Bug #32246061, Bug
       #32246200.

     * InnoDB: Sampling of InnoDB data for the purpose of
       generating histogram statistics is now supported with all
       transaction isolation levels supported by InnoDB.
       Previously, sampling was performed using only the READ
       UNCOMMITTED isolation level. (Bug #32555575)

     * InnoDB: An index with a key prefix length greater than
       767 bytes was permitted on a table defined with the
       REDUNDANT row format, exceeding the index key prefix
       length limit for that row format. The ALTER TABLE
       operation that added the index validated the index key
       prefix length for the row format defined by the
       innodb_default_row_format variable instead of the actual
       row format of the table. The fix ensures that index key
       prefix length is validated for the correct row format.
       (Bug #32507117, Bug #102597)

     * InnoDB: Adaptive hash index latches did not provide
       meaningful latch location information. (Bug #32477773)

     * InnoDB: A dependency related to redo and undo log
       encryption at server initialization time was removed.
       (Bug #32476724)

     * InnoDB: An online buffer pool resizing operation freed
       the previous buffer pool page hash, conflicting with a
       concurrent buffer pool lookup that required the previous
       page hash. (Bug #32460315)

     * InnoDB: When using the TempTable storage engine
       (internal_tmp_mem_storage_engine=TempTable), more than
       255 aggregate functions in a SELECT list caused errors
       due to overflow of an internal variable that stores
       indexed column field positions. (Bug #32458104, Bug
       #102468)

     * InnoDB: A workload stalled while executing a undo
       tablespace truncation operation on an instance with a
       large buffer pool. The function that truncates marked
       undo tablespaces now takes a shared latch instead of an
       exclusive latch, and the shared latch is taken for a
       shorter period of time. (Bug #32353863, Bug #102143)

     * InnoDB: A programming interface was added for handling
       dynamic storage of over-aligned types. (Bug #32246200,
       Bug #32246061)

     * InnoDB: Under certain circumstances, a failure during
       InnoDB recovery could have caused a loss of committed
       changes. Checkpoints permitted during recovery did not
       handle page flushing, flush list maintenance, or
       persisting changes to the data dictionary table buffer as
       necessary for a proper checkpoint operations. Checkpoints
       and advancing the checkpoint LSN are therefore no longer
       permitted until redo log recovery is complete and data
       dictionary dynamic metadata (srv_dict_metadata) is
       transferred to data dictionary table (dict_table_t)
       objects. Should the redo log run out of space during
       recovery or after recovery (but before data dictionary
       dynamic metadata is transferred to data dictionary table
       objects) as a result of this change, an
       innodb_force_recovery restart may be required, starting
       with at least the SRV_FORCE_NO_IBUF_MERGE setting or, in
       case that fails, the SRV_FORCE_NO_LOG_REDO setting. If an
       innodb_force_recovery restart fails in this scenario,
       recovery from backup may be necessary. (Bug #32200595)

     * InnoDB: Rollback segments are now initialized in parallel
       during startup. Previously, rollback segments were
       initialized serially. (Bug #32170127, Bug #101658)

     * InnoDB: A failure occurred during testing of
       innodb_log_writer_threads variable configuration. The
       failure was caused by a race condition. (Bug #32129814)

       References: This issue is a regression of: Bug #30088404.

     * InnoDB: A race condition occurred between a purge thread
       that was truncating an undo tablespace and a server
       thread that queried the INFORMATION_SCHEMA.FILES table.
       As a result, the truncated undo tablespace did not appear
       in the INFORMATION_SCHEMA.FILES table when queried, which
       in turn caused a MySQL Enterprise Backup failure due to a
       dependency on the INFORMATION_SCHEMA.FILES table for undo
       tablespace file locations. (Bug #32104924, Bug #32654667)

     * InnoDB: When DML operations are concentrated on a single
       table, purge work was performed by a single purge thread,
       which could result in slowed purge operations, increased
       purge lag, and increased tablespace file size if DML
       operations involve large object values. To address this
       issue, purge work is now automatically redistributed
       among available purge threads when the purge lag exceeds
       the innodb_max_purge_lag setting. (Bug #32089028)

     * InnoDB: The function that populates the
       INFORMATION_SCHEMA.INNODB_TABLESPACES table accessed the
       space for a file and performed an unprotected stat()
       operation and retrieval of file size information. (Bug
       #32025344)

     * InnoDB: Code related to the trx_t::is_recovered flag in
       the InnoDB sources was revised to address various
       complexity and correctness issues. One of the issues
       addressed caused an XA transaction to be described
       incorrectly as "recovered", which occurred when a client
       session disconnected from an XA transaction after XA
       PREPARE. (Bug #31870582)

     * InnoDB: TempTable debug assertion code for an
       Indexed_cells member function
       (cell_from_mysql_buf_index_read()) did not account for
       non-nullable columns with zero length. (Bug #31091089)

     * InnoDB: Using the InnoDB memcached plugin, attempting to
       retrieve multiple values in a single get command returned
       an incorrect value. (Bug #29675958, Bug #95032)

     * InnoDB: The trx_sys_t::serialisation_mutex was introduced
       to reduce contention on the on the trx_sys_t::mutex. The
       new mutex protects the trx_sys_t::serialisation_list when
       a transaction number is assigned, which was previously
       protected by the trx_sys_t::mutex.

       Thanks to Zhai Weixiang for the contribution. (Bug
       #27933068, Bug #90643)

     * Partitioning: When a table was partitioned by TIMESTAMP
       and a timestamp literal with a time zone offset was used
       in the WHERE clause of a SELECT statement, it was
       possible for a partition to be omitted from the result
       set.

       When a time zone offset is specified in a timestamp
       literal, it is expected to be converted to a timestamp
       without a time zone offset, and then compared against a
       timestamp column, but this was not done properly in all
       cases, with the result that a partition could be pruned
       while selecting the partitions to be scanned for the
       query.

       We fix this by making sure that a timestamp with a time
       zone offset is always converted as described before
       comparing with values from the column. (Bug #101530, Bug
       #32134145)

     * Replication: When the SOURCE_CONNECTION_AUTO_FAILOVER=1
       option was set on the CHANGE REPLICATION SOURCE TO
       statement for a replication channel, a STOP REPLICA
       IO_THREAD statement did not stop the monitor thread that
       monitors for connection failures on the channel, and
       could incorrectly stop the applier thread. (Bug
       #32892977)

     * Replication: When the system variable
       replication_optimize_for_static_plugin_config was set,
       the plugins for Group Replication and semi-synchronous
       replication could not be uninstalled cleanly on server
       shutdown. (Bug #32798287)

     * Replication: In Group Replication's single-primary mode,
       unnecessary copies of the transaction data were created
       during data serialization. This is now done in a single
       step to reduce the memory footprint. (Bug #32781945)

     * Replication: A deadlock could occur when START
       GROUP_REPLICATION and STOP GROUP_REPLICATION statements
       were issued at the same time that a view change was
       taking place for the group. (Bug #32738137, Bug
       #32836868)

     * Replication: An incorrect default value in code meant
       that the allowlist of IP addresses permitted for a
       replication group was implicitly reconfigured although no
       value had been supplied. (Bug #32714911)

     * Replication: A deadlock could occur if a STOP
       GROUP_REPLICATION statement was issued when a replication
       channel on a group member was attempting to commit a
       transaction. The server now rolls back the transaction
       immediately if it cannot acquire the relevant lock,
       rather than waiting for the lock and the commit to
       complete and causing the deadlock. (Bug #32633176)

     * Replication: On a multithreaded replica, the reference to
       the active event was sometimes managed incorrectly when
       retrying a transaction. (Bug #32590974)

     * Replication: Previously, a warning message was written to
       the error log if the original commit timestamp on a
       replicated transaction was more recent than the immediate
       commit timestamp on the replica applying it. The message
       could occur inappropriately if the fluctuation in the
       replication lag had a similar value to the clock
       difference between the machines involved, which could be
       made more likely by better quality connections between
       them. In the event of a Group Replication failover
       procedure, it was possible for a message to be returned
       for every transaction, flooding the log. To avoid this
       situation, the warning message has been withdrawn. (Bug
       #32554807)

     * Replication: After a DML operation was performed on the
       last partition of a table with more than 128 partitions,
       MySQL Server and MySQL clients (such as mysqlbinlog)
       parsed the event information from the binary log
       incorrectly, resulting in an inaccurate partition ID
       being stated. The information is now read using an event
       reader function that is endianness independent. (Bug
       #32358736, Bug #102191)

     * Replication: In a new MySQL Server installation, the
       mysql.gtid_executed system table was missing the property
       STATS_PERSISTENT=0 to disable persistent statistics,
       which is present for the other replication-related
       tables. (Bug #32250735)

     * Replication: When the same row in a table was updated
       multiple times by the same event, the replication
       applier's hash scan algorithm omitted to check for JSON
       partial updates, which are logged when
       binlog_row_value_options=PARTIAL_JSON is set. This could
       result in replication stopping with a "key not found"
       error. (Bug #32221703)

     * Replication: Replica servers now check and validate the
       transaction ID part of a GTID before applying and
       committing the transaction associated with it. (Bug
       #32103192)

     * Replication: With the Group Replication system variable
       group_replication_consistency = AFTER set, if a view
       change event was delayed until after a locally prepared
       transaction was completed, a different GTID could be
       applied to it, causing errors in replication. The data is
       now processed in the same sequence it is received to
       avoid the situation. (Bug #31872708)

     * Replication: Replication could stop on a multithreaded
       replica if a unique secondary key was omitted from the
       writeset hashes used to compute transaction dependencies,
       leading to errors when executing the transactions on the
       multithreaded replica. Write set hashes now always
       include unique secondary keys even if they are not
       included in the read set and write set. (Bug #31636339)

     * Replication: On a multithreaded replica (with
       slave_parallel_workers > 0 ), the algorithm used by GTID
       auto-positioning to check for missing transactions
       briefly sets a low value (4) for the event position in
       the course of its calculations. If the operation is
       stopped at that moment, the recovery process that
       resolves gaps in the sequence of transactions can fail.
       The process to resolve gaps is not actually necessary
       when GTID auto-positioning is used, so the process has
       now been disabled in that situation.

       As a result, on a multithreaded replica, when GTID_MODE =
       ON is set for the instance and SOURCE_AUTO_POSITION is
       set for the channel using the CHANGE REPLICATION SOURCE
       TO statement, the following behaviors now apply:

          + A START REPLICA UNTIL SQL_AFTER_MTS_GAPS statement
            just stops the applier thread when it finds the
            first event to execute, and does not attempt to
            check for gaps in the sequence of transactions.

          + A CHANGE REPLICATION SOURCE TO statement does not
            automatically fail if there are gaps in the sequence
            of transactions.

       These changed behaviors only apply on a multithreaded
       replica that uses GTIDs and GTID auto-positioning, and
       not on a replica that uses binary log position-based
       replication. (Bug #30571587, Bug #97694)

     * Microsoft Windows: Writing to Windows event logs could be
       unsuccessful. (Bug #32890918)

     * JSON: Reading JSON values from tables that used the CSV
       storage engine raised an error such as Cannot create a
       JSON value from a string with CHARACTER SET 'binary'.
       This happened because the CSV engine uses my_charset_bin
       as the character set for the record buffer but creation
       of JSON values includes an explicit check for
       my_charset_bin, and raises an error if this character set
       is given.

       We handle this issue by passing the actual character set
       of the column instead of the character set of the buffer
       holding the data, which is always binary. (Bug #102843,
       Bug #32597017)

     * A query for which the derived condition pushdown
       optimization could be applied was not so optimized when
       the query was part of INSERT ... SELECT. (Bug #32959186)

     * Import operations for access privilege information became
       very slow for large numbers of accounts and schemas. (Bug
       #32934351)

     * Queries which needed to sort the results of a full-text
       index scan were in some circumstances not handled
       correctly. (Bug #32924198)

     * Thanks to Xiaoyu Wang for a code correction to
       PFS_notification_registry::is_empty(). (Bug #32919118,
       Bug #103788)

     * Queries containing GROUP BY, ORDER BY, and LIMIT in a
       subquery and accessed using a cursor could cause a server
       exit. (Bug #32918240)

     * When invoked with the --help and --verbose options,
       mysqld created an auto.cnf file in the current directory.
       (Bug #32906164)

     * Queries that involved pushing a condition with view
       references down to a materialized derived table could
       cause a server exit. (Bug #32905044, Bug #32324234)

     * Messages for ER_CANT_INITIALIZE_UDF errors could be
       truncated. (Bug #32891703)

     * A regression was found in the simplification of streaming
       aggregation (GROUP BY of data already sorted) that was
       performed in MySQL 8.0.23.

       We fix this issue as follows: When there is an implicit
       grouping on a single table which is the subject of a
       fulltext search, we now force insertion of a temporary
       table to materialize MATCH() temporaries before they are
       sent to the AggregateIterator, since it tries to save and
       restore the rows it receives, but cannot properly include
       the fulltext search information, as it is hidden. (Bug
       #32889491)

       References: This issue is a regression of: Bug #31790217.

     * For conversion of -DBL_MAX to string and back to double,
       the new double value differed from the original and was
       rejected as out of bounds. (Bug #32888982, Bug #103709)

     * Now, whenever the JSON_LENGTH() function includes the
       optional path argument, the server rewrites it as
       JSON_LENGTH(JSON_EXTRACT(doc, path)). This means that
       JSON_LENGTH() now supports wildcards (such as $, ., and
       *) and array ranges in the path, as shown here:

mysql> SELECT JSON_LENGTH('[1,2,3,4,5,6,7]', '$[2 to 4]') AS x;
+------+
| x    |
+------+
|    3 |
+------+

       (Bug #32877703)

     * For flags typically used for RPM and Debian packages, the
       new WITH_PACKAGE_FLAGS CMake option controls whether to
       add those flags to standalone builds on those platforms.
       The default is ON for nondebug builds. This is used to
       "harden" the builds; for example, by adding
       -D_FORTIFY_SOURCE=2. (Bug #32876974)

     * The NULLIF() function did not perform all necessary
       checks for errors. (Bug #32865008)

     * For views that depended on other views, output from the
       SHOW CREATE VIEW statement used during production of dump
       files could cause an error at restore time. (Bug
       #32854203, Bug #103583)

     * The query-attributes code did not properly handle large
       64-bit numbers. (Bug #32847269)

     * Information retrieved from the Performance Schema
       metadata_locks table could be incorrect for foreign keys
       and CHECK constraints. (Bug #32832004, Bug #103532)

     * When generating unique names for view columns, the server
       now considers only those objects whose names are visible.
       (Bug #32821372)

     * When a condition is pushed down to a materialized derived
       table, a clone of the derived table expression replaces
       the column (from the outer query block) in the condition.
       When the cloned item included a FULLTEXT function, it was
       added to the outer query block instead of the derived
       table query block, which led to problems. To fix this, we
       now use the derived query block to clone such items. (Bug
       #32820437)

     * A common table expression which was used more than once
       in a statement, at least once within a subquery that was
       subsequently removed during resolution due to being
       always true, or always false. (Bug #32813547, Bug
       #32813554)

     * If a statement is prepared against a table that is
       persistent at preparation time but temporary at the time
       of first execution, an assertion could be raised. (Bug
       #32799797)

     * An internal function used by spatial functions could
       reference memory after freeing it. (Bug #32793104)

     * The impossible filter optimization removed MRR access
       paths that were required by the corresponding BKA access
       paths. (Bug #32787415)

     * The new WITH_AUTHENTICATION_CLIENT_PLUGINS CMake option
       is enabled automatically if any corresponding server
       authentication plugins are built. Its value thus depends
       on other CMake options and it should not be set
       explicitly. (Bug #32778378)

     * The MRR iterator normally filters out NULL keys by
       checking impossible_null_ref(), but when a join condition
       either contained an IS NULL predicate, or used the
       NULL-safe equals operator ≪=>, the optimizer had to check
       whether the join condition used the predicate terms as
       part of its join condition, and not set the internal flag
       HA_MRR_NO_NULL_ENDPOINTS in such cases. Now we check,
       using a bitmask, whether the each column in the key
       rejects NULL, in which case we can set
       HA_MRR_NO_NULL_ENDPOINTS without further checks. (Bug
       #32774281)

     * For system variables with an enumeration type, SET
       PERSIST_ONLY var_name = DEFAULT persisted the numeric
       value and not the symbolic name. (Bug #32761053)

     * The X DevAPI operations Collection.replaceOne and
       Collection.addOrReplaceOne now verify that the value of
       the _id field in a document matches the id parameter
       specified for the operation. If it does not, an error is
       returned. (Bug #32753547)

     * For applications that use the C API to execute prepared
       statements, query attributes could not be used for
       prepared statements with no parameters. (Bug #32753030,
       Bug #32790714, Bug #32955044)

     * The arguments to IN() were not always converted to the
       correct character set. (Bug #32746552)

     * The LOCATE() function unconditionally returned NULL when
       an argument could not be evaluated. Now, when used in an
       expression that is determined to be non-nullable, the
       function returns zero instead. (Bug #32746536)

     * The internal function my_well_formed_len_utf32() asserted
       when presented with a string of invalid length. Now in
       such cases, the function reports an invalid string
       instead. (Bug #32745294)

     * The functions TRIM(), RTRIM(), and LTRIM() did not always
       perform proper error checking. (Bug #32744772)

     * A previous fix in an internal resolver function ensured
       that it raises an error when a generated column cannot be
       resolved. This worked without any problem when the
       generated column is part of a CREATE TABLE statement, but
       in the case where the table with the generated column was
       generated on a MySQL 5.7 database and then upgraded to
       MySQL 8.0, an error was reported and the upgrade
       terminated.

       We fix this by using the correct pointer when reporting
       the table causing the error in the function
       fix_generated_columns_for_upgrade(). (Bug #32738972)

     * When looking inside rollup wrappers in the SELECT list,
       and trying to find the same item in the GROUP BY list,
       the server did not take into account that a cache might
       have been added around the expression. Now any such
       caches are unwrapped before looking for the item. (Bug
       #32729377, Bug #32918400)

     * A prepared statement that used MIN() or MAX() could
       return an incorrect result if it also included dynamic
       parameters. (Bug #32713860, Bug #103170)

     * Replication could fail if a DML statement was executed
       immediately after an XA transaction was rejected or
       forced to rollback due to a deadlock. (Bug #32707060)

     * Queries containing multiple instances of GROUPING() were
       not always handled correctly. (Bug #32695103)

     * When executing EXPLAIN ANALYZE, materialization iterators
       count every single init() call, even those that only
       retain existing data, causes materializations to appear
       to cost too little compared to the number of underlying
       calls. We fix this by allowing the materialization
       iterator to override the call and row counts with its own
       data. (Bug #32688540)

     * A race condition in the metadata locking code could
       result in a server exit. (Bug #32686116)

     * An index-only scan on a covering full-text index could
       return incorrect results for queries with multiple calls
       to MATCH() function depending on the order in which the
       MATCH() calls were evaluated. (Bug #32685616)

     * Including query attributes for a prepared statement could
       cause a statement execution failure. (Bug #32676878)

     * Replicated transactions in GTID-based replication include
       an original_commit_timestamp to show when the transaction
       was committed on the original source server, and an
       immediate_commit_timestamp to show when it was committed
       on the replica. Previously, the original_commit_timestamp
       was not set for Group Replication's view change events,
       which are agreed by the group but then generated and
       applied by each group member, so the timestamp appeared
       as zero in viewable output. For improved observability,
       group members now set local timestamp values for
       transactions associated with view change events. (Bug
       #32668567)

     * Executing DDL statements on a system table could cause a
       server exit. (Bug #32665990)

     * With the --online-migration option, mysql_migrate_keyring
       could exit during connection establishment to the
       migration server. (Bug #32651203)

     * mysql_migrate_keyring failed to enforce the condition
       that the source and destination keystores must differ.
       (Bug #32637784)

     * System cache size checking could be inaccurate on Ubuntu.
       (Bug #32619199, Bug #102926)

     * The server did not process correctly some queries that
       used MATCH ... AGAINST on a column with a fulltext index
       in a HAVING clause. (Bug #32617181)

       References: This issue is a regression of: Bug #30969045.

     * Replicated transactions in GTID-based replication include
       an original_commit_timestamp to show when the transaction
       was committed on the original source server, and an
       immediate_commit_timestamp to show when it was committed
       on the replica. Previously, Group Replication servers set
       the original_commit_timestamp when they committed locally
       generated transactions, but they did not set the
       immediate_commit_timestamp. For improved observability,
       group members now set this timestamp. If the transaction
       originated in the group, the original_commit_timestamp
       and immediate_commit_timestamp are both generated by
       Group Replication, and are equal. If the transaction
       originated outside the group, the
       original_commit_timestamp is preserved, and an
       immediate_commit_timestamp is set. (Bug #32613896)

     * The internal function Item_func_match::eq() raised an
       assert failure in debug builds when it was called with an
       argument that was an Item_func_match_predicate. The
       assertion was added with the expectation that an
       Item_func_match object would not be compared with an
       Item_func_match_predicate object, but it was later found
       that this can happen during the ONLY_FULL_GROUP_BY check
       when the predicate is in a HAVING clause.

       We fixe this by removing the assertion so that the
       function returns false, instead. (Bug #32611913)

     * SELECT using DISTINCT with a GROUP BY ... WITH ROLLUP on
       a primary key column returned a different result than
       when the column was not a primary key. (Bug #32565875)

     * When an item in the SELECT list came from a table that
       was found to be constant, it was possible to add caches
       around it before adding ROLLUP wrappers, causing it to be
       unfindable in the group list (which had no such
       wrappers). This is addressed by unwrapping the caches.
       (Bug #32559356)

     * The C client library could produce a Malformed
       communication packet error if a prepared statement
       parameter was longer than 8KB and the next parameter
       represented a NULL value. (Bug #32558782)

     * When comparing arguments of different types, and the
       arguments are deemed constants, the server may insert its
       own hidden item cache to save on repeated conversion (and
       possible warnings). This is not visible in the output of
       any statements such as EXPLAIN.

       An issue arose due to the fact that resolution of ROLLUP
       runs after the comparisons are set up, and may replace
       the arguments, which could lead to incorrect results from
       comparisons. Now when this happens, we signal that the
       comparison needs to reconsider its caches, and not
       possibly re-use stale values. (Bug #32548377)

     * Incorrect reference counting for a destructor in the
       implementation of a loadable function could raise an
       assertion for debug builds or report that the function
       does not exist for nondebug builds. (Bug #32528898, Bug
       #102649)

     * MySQL source distributions now bundle the Google Test
       source code, which no longer need be downloaded to run
       Google Test-based unit tests. Consequently, the
       WITH_GMOCK and ENABLE_DOWNLOADS CMake options have been
       removed and are ignored if specified.

       This change also corrects a couple of build issues
       whereby source distributions incorrectly included an
       empty source_downloads directory, and CMake did not fail
       when WITH_UNIT_TESTS was enabled but the Google Test
       source was not found. (For the latter issue, it can no
       longer occur because the source is always present.) (Bug
       #32512077, Bug #27326599, Bug #29935278)

     * When outer batched key access and block-nested loop (or
       hash join) occurred together in a query, both were
       reverted to nested loops in the plan interpretation to
       access paths. Problems arose in some cases in which
       non-equality join conditions had been pushed to a special
       kind of BKA index condition, and by converting to a
       regular index lookup (which does not check the BKA
       condition), it was possible to drop conditions that
       should have been checked.

       We fix this by removing the BKA index condition; since
       its use is very rare, potential gains in most practical
       queries are very low, and removing it decreases
       complexity significantly. (Bug #32424884)

     * SHOW CREATE USER could cause an unexpected server exit if
       the structure of the mysql.user system table was manually
       altered. (Bug #32417780, Bug #31654586)

     * For large values of the group_concat_max_len system
       variable, prepared statements that used the
       GROUP_CONCAT() function could be unnecessarily
       re-prepared across executions. (Bug #32413530, Bug
       #102344)

     * Changes were not properly rolled back on re-execution of
       prepared statements or stored procedures for which walk
       and replace was performed as part of optimization. (Bug
       #32384324, Bug #32573871)

     * Client programs using the asynchronous C API functions
       could leak memory if a connection timeout occurred.
       Thanks for Facebook for a contribution used in fixing
       this issue. (Bug #32372038, Bug #102189, Bug #32800091,
       Bug #103409)

     * The dynamic statistics cache exposed by the
       INFORMATION_SCHEMA could be updated in the middle of a
       transaction before it was known whether the transaction
       would commit, potentially caching information
       corresponding to no committed state. (Bug #32338335)

     * Repreparation of a prepared statement at the beginning of
       an implicit transaction could cause an
       ER_GTID_NEXT_TYPE_UNDEFINED_GROUP error. (Bug #32326510,
       Bug #102031)

     * USING HASH was removed from the definitions of
       Performance Schema thread pool tables because the
       Performance Schema does not support hash indexes. (Bug
       #32194617)

     * Inserting a datetime literal with an explicit time zone
       offset into a TIMESTAMP column could produce the wrong
       time if time_zone=SYSTEM and the system time zone has DST
       transitions. (Bug #32173664, Bug #101670)

     * While setting up ORDER BY for a window function, a window
       definition including a subquery was removed along with
       the subquery in the ORDER BY. (Bug #32103260)

     * The use of a SET PERSIST or SET PERSIST_ONLY statement is
       now disallowed for the group_replication_force_members
       system variable. This system variable must be cleared
       after use before you can issue a START GROUP_REPLICATION
       statement, so it should not be persisted to the MySQL
       server instance's option file. (Bug #31983203)

     * Upgrades from MySQL 5.7 to 8.0 failed if the MySQL
       installation had FEDERATED tables. (Bug #31973053)

     * Persisting component-related system variables could
       result in unnecessary "unknown variable" error messages.
       (Bug #31968366)

     * An out-of-memory error occurred when loading large
       amounts of data into tables with full-text search
       indexes. Not all of the memory allocated to the full-text
       search cache was accounted for when inserting data into
       the full-text search auxiliary tables. (Bug #31576731)

     * The precision for a UNION between integer and decimal
       values could be calculated incorrectly, leading to
       truncation of the integer in the result. (Bug #31348202,
       Bug #99567)

     * InnoDB wrote unnecessary warnings to the error log
       indicating that table statistics could not be calculated.
       (Bug #30865032)

     * A secondary index over a virtual column became corrupted
       when the index was built online.

       For UPDATE statements, we fix this as follows: If the
       virtual column value of the index record is set to NULL,
       then we generate this value from the cluster index
       record. (Bug #30556595)

     * DROP DATABASE for a database with a large number of
       tables (one million) could result in an out-of-memory
       condition. (Bug #29634540)

     * The system_time_zone system variable is initialized at
       server startup from the server host settings and the
       mysqld environment, but if the server host time zone
       changed (for example, due to daylight saving time),
       system_time_zone did not reflect the change. Now it does.
       (Bug #29330089, Bug #94257)

     * For upgrades on Ubuntu, if an existing my.cnf file is
       found, it is renamed to my.cnf.bak and a warning is
       issued. (Bug #24486556, Bug #82639)

     * Boolean system variables could be assigned a negative
       value. (Bug #11758439, Bug #50643)

     * Prepared statements did not always make use of index
       extensions (see Use of Index Extensions
(https://dev.mysql.com/doc/refman/8.0/en/index-extensions.html)).
       (Bug #103711, Bug #32897525)

       References: See also: Bug #103710, Bug #32897503.

     * When enabled, the prefer_ordering_index optimizer switch
       had a negative effect on the performance of prepared
       statements. (Bug #103710, Bug #32897503)

       References: See also: Bug #103711, Bug #32897525.

     * Successive INSERT() function calls could sometimes yield
       invalid NULL results. (Bug #103513, Bug #32824978)

     * Some NOT IN subqueries did not return correct results due
       to a regression in NULL handling. (Bug #103331, Bug
       #32773801)

       References: This issue is a regression of: Bug #30473261.

     * Whenever a SELECT ... FOR UPDATE NOWAIT statement was
       unable to obtain a record lock, a message Got error 203
       when reading table ... was written to the error log, even
       though though this is a relatively common occurrence, the
       logging of which led to excessive use of disk space.

       Our thanks to Brian Yue for this contribution. (Bug
       #103157, Bug #32705614)

     * 0 - (MAX(BIGINT) + 1) returned -(MAX(BIGINT) + 1). Now an
       out of range error is returned instead. (Bug #103093, Bug
       #32693863)

