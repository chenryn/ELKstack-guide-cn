# postfix 日志

postfix 是 Linux 平台上最常用的邮件服务器软件。邮件服务的运维复杂度一向较高，在此提供一个针对 postfix 日志的解析处理方案。方案出自：<https://github.com/whyscream/postfix-grok-patterns>。

因为 postfix 默认通过 syslog 方式输出日志，所以可以选择通过 rsyslog 直接转发给 logstash，也可以由 logstash 读取 rsyslog 记录的文件。下列配置中省略了对 syslog 协议解析的部分。

```
filter {
    # grok log lines by program name (listed alpabetically)
    if [program] =~ /^postfix.*\/anvil$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_ANVIL}" ]
            tag_on_failure => [ "_grok_postfix_anvil_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/bounce$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_BOUNCE}" ]
            tag_on_failure => [ "_grok_postfix_bounce_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/cleanup$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_CLEANUP}" ]
            tag_on_failure => [ "_grok_postfix_cleanup_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/dnsblog$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_DNSBLOG}" ]
            tag_on_failure => [ "_grok_postfix_dnsblog_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/local$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_LOCAL}" ]
            tag_on_failure => [ "_grok_postfix_local_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/master$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_MASTER}" ]
            tag_on_failure => [ "_grok_postfix_master_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/pickup$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_PICKUP}" ]
            tag_on_failure => [ "_grok_postfix_pickup_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/pipe$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_PIPE}" ]
            tag_on_failure => [ "_grok_postfix_pipe_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/postdrop$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_POSTDROP}" ]
            tag_on_failure => [ "_grok_postfix_postdrop_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/postscreen$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_POSTSCREEN}" ]
            tag_on_failure => [ "_grok_postfix_postscreen_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/qmgr$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_QMGR}" ]
            tag_on_failure => [ "_grok_postfix_qmgr_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/scache$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_SCACHE}" ]
            tag_on_failure => [ "_grok_postfix_scache_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/sendmail$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_SENDMAIL}" ]
            tag_on_failure => [ "_grok_postfix_sendmail_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/smtp$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_SMTP}" ]
            tag_on_failure => [ "_grok_postfix_smtp_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/lmtp$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_LMTP}" ]
            tag_on_failure => [ "_grok_postfix_lmtp_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/smtpd$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_SMTPD}" ]
            tag_on_failure => [ "_grok_postfix_smtpd_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/tlsmgr$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_TLSMGR}" ]
            tag_on_failure => [ "_grok_postfix_tlsmgr_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/tlsproxy$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_TLSPROXY}" ]
            tag_on_failure => [ "_grok_postfix_tlsproxy_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/trivial-rewrite$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_TRIVIAL_REWRITE}" ]
            tag_on_failure => [ "_grok_postfix_trivial_rewrite_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    } else if [program] =~ /^postfix.*\/discard$/ {
        grok {
            patterns_dir   => ["/etc/logstash/patterns.d"]
            match          => [ "message", "%{POSTFIX_DISCARD}" ]
            tag_on_failure => [ "_grok_postfix_discard_nomatch" ]
            add_tag        => [ "_grok_postfix_success" ]
        }
    }

    # process key-value data is it exists
    if [postfix_keyvalue_data] {
        kv {
            source       => "postfix_keyvalue_data"
            trim         => "<>,"
            prefix       => "postfix_"
            remove_field => [ "postfix_keyvalue_data" ]
        }

        # some post processing of key-value data
        if [postfix_client] {
            grok {
                patterns_dir   => ["/etc/logstash/patterns.d"]
                match          => ["postfix_client", "%{POSTFIX_CLIENT_INFO}"]
                tag_on_failure => [ "_grok_kv_postfix_client_nomatch" ]
                remove_field   => [ "postfix_client" ]
            }
        }
        if [postfix_relay] {
            grok {
                patterns_dir   => ["/etc/logstash/patterns.d"]
                match          => ["postfix_relay", "%{POSTFIX_RELAY_INFO}"]
                tag_on_failure => [ "_grok_kv_postfix_relay_nomatch" ]
                remove_field   => [ "postfix_relay" ]
            }
        }
        if [postfix_delays] {
            grok {
                patterns_dir   => ["/etc/logstash/patterns.d"]
                match          => ["postfix_delays", "%{POSTFIX_DELAYS}"]
                tag_on_failure => [ "_grok_kv_postfix_delays_nomatch" ]
                remove_field   => [ "postfix_delays" ]
            }
        }
    }

    # Do some data type conversions
    mutate {
        convert => [
            # list of integer fields
            "postfix_anvil_cache_size", "integer",
            "postfix_anvil_conn_count", "integer",
            "postfix_anvil_conn_rate", "integer",
            "postfix_client_port", "integer",
            "postfix_nrcpt", "integer",
            "postfix_postscreen_cache_dropped", "integer",
            "postfix_postscreen_cache_retained", "integer",
            "postfix_postscreen_dnsbl_rank", "integer",
            "postfix_relay_port", "integer",
            "postfix_server_port", "integer",
            "postfix_size", "integer",
            "postfix_status_code", "integer",
            "postfix_termination_signal", "integer",
            "postfix_uid", "integer",

            # list of float fields
            "postfix_delay", "float",
            "postfix_delay_before_qmgr", "float",
            "postfix_delay_conn_setup", "float",
            "postfix_delay_in_qmgr", "float",
            "postfix_delay_transmission", "float",
            "postfix_postscreen_violation_time", "float"
        ]
    }
}
```

配置中使用了一系列自定义 grok 正则，全部内容如下所示。保存成 `/etc/logstash/patterns.d/` 下一个文本文件即可。

```
# common postfix patterns
POSTFIX_QUEUEID ([0-9A-F]{6,}|[0-9a-zA-Z]{15,}|NOQUEUE)
POSTFIX_CLIENT_INFO %{HOST:postfix_client_hostname}?\[%{IP:postfix_client_ip}\](:%{INT:postfix_client_port})?
POSTFIX_RELAY_INFO %{HOST:postfix_relay_hostname}?\[(%{IP:postfix_relay_ip}|%{DATA:postfix_relay_service})\](:%{INT:postfix_relay_port})?|%{WORD:postfix_relay_service}
POSTFIX_SMTP_STAGE (CONNECT|HELO|EHLO|STARTTLS|AUTH|MAIL|RCPT|DATA|RSET|UNKNOWN|END-OF-MESSAGE|VRFY|\.)
POSTFIX_ACTION (reject|defer|accept|header-redirect)
POSTFIX_STATUS_CODE \d{3}
POSTFIX_STATUS_CODE_ENHANCED \d\.\d\.\d
POSTFIX_DNSBL_MESSAGE Service unavailable; .* \[%{GREEDYDATA:postfix_status_data}\] %{GREEDYDATA:postfix_status_message};
POSTFIX_PS_ACCESS_ACTION (DISCONNECT|BLACKLISTED|WHITELISTED|WHITELIST VETO|PASS NEW|PASS OLD)
POSTFIX_PS_VIOLATION (BARE NEWLINE|COMMAND (TIME|COUNT|LENGTH) LIMIT|COMMAND PIPELINING|DNSBL|HANGUP|NON-SMTP COMMAND|PREGREET)
POSTFIX_TIME_UNIT %{NUMBER}[smhd]
POSTFIX_KEYVALUE %{POSTFIX_QUEUEID:postfix_queueid}: %{GREEDYDATA:postfix_keyvalue_data}
POSTFIX_WARNING (warning|fatal): %{GREEDYDATA:postfix_warning}
POSTFIX_TLSCONN (Anonymous|Trusted|Untrusted|Verified) TLS connection established (to %{POSTFIX_RELAY_INFO}|from %{POSTFIX_CLIENT_INFO}): %{DATA:postfix_tls_version} with cipher %{DATA:postfix_tls_cipher} \(%{DATA:postfix_tls_cipher_size} bits\)
POSTFIX_DELAYS %{NUMBER:postfix_delay_before_qmgr}/%{NUMBER:postfix_delay_in_qmgr}/%{NUMBER:postfix_delay_conn_setup}/%{NUMBER:postfix_delay_transmission}
POSTFIX_LOSTCONN (lost connection|timeout|Connection timed out)
POSTFIX_PROXY_MESSAGE (%{POSTFIX_STATUS_CODE:postfix_proxy_status_code} )?(%{POSTFIX_STATUS_CODE_ENHANCED:postfix_proxy_status_code_enhanced})?.*

# helper patterns
GREEDYDATA_NO_COLON [^:]*

# smtpd patterns
POSTFIX_SMTPD_CONNECT connect from %{POSTFIX_CLIENT_INFO}
POSTFIX_SMTPD_DISCONNECT disconnect from %{POSTFIX_CLIENT_INFO}
POSTFIX_SMTPD_LOSTCONN (%{POSTFIX_LOSTCONN:postfix_smtpd_lostconn_data} after %{POSTFIX_SMTP_STAGE:postfix_smtp_stage}( \(%{INT} bytes\))? from %{POSTFIX_CLIENT_INFO}|%{GREEDYDATA:postfix_action} from %{POSTFIX_CLIENT_INFO}: %{POSTFIX_LOSTCONN:postfix_smtpd_lostconn_data})
POSTFIX_SMTPD_NOQUEUE NOQUEUE: %{POSTFIX_ACTION:postfix_action}: %{POSTFIX_SMTP_STAGE:postfix_smtp_stage} from %{POSTFIX_CLIENT_INFO}: %{POSTFIX_STATUS_CODE:postfix_status_code} %{POSTFIX_STATUS_CODE_ENHANCED:postfix_status_code_enhanced}( <%{DATA:postfix_status_data}>:)? (%{POSTFIX_DNSBL_MESSAGE}|%{GREEDYDATA:postfix_status_message};) %{GREEDYDATA:postfix_keyvalue_data}
POSTFIX_SMTPD_PIPELINING improper command pipelining after %{POSTFIX_SMTP_STAGE:postfix_smtp_stage} from %{POSTFIX_CLIENT_INFO}:
POSTFIX_SMTPD_PROXY proxy-%{POSTFIX_ACTION:postfix_proxy_result}: (%{POSTFIX_SMTP_STAGE:postfix_proxy_smtp_stage}): %{POSTFIX_PROXY_MESSAGE:postfix_proxy_message}; %{GREEDYDATA:postfix_keyvalue_data}

# cleanup patterns
POSTFIX_CLEANUP_MILTER %{POSTFIX_QUEUEID:postfix_queueid}: milter-%{POSTFIX_ACTION:postfix_milter_result}: %{GREEDYDATA:postfix_milter_message}; %{GREEDYDATA_NO_COLON:postfix_keyvalue_data}(: %{GREEDYDATA:postfix_milter_data})?

# qmgr patterns
POSTFIX_QMGR_REMOVED %{POSTFIX_QUEUEID:postfix_queueid}: removed
POSTFIX_QMGR_ACTIVE %{POSTFIX_QUEUEID:postfix_queueid}: %{GREEDYDATA:postfix_keyvalue_data} \(queue active\)

# pipe patterns
POSTFIX_PIPE_DELIVERED %{POSTFIX_QUEUEID:postfix_queueid}: %{GREEDYDATA:postfix_keyvalue_data} \(delivered via %{WORD:postfix_pipe_service} service\)
POSTFIX_PIPE_FORWARD %{POSTFIX_QUEUEID:postfix_queueid}: %{GREEDYDATA:postfix_keyvalue_data} \(mail forwarding loop for %{GREEDYDATA:postfix_to}\)

# postscreen patterns
POSTFIX_PS_CONNECT CONNECT from %{POSTFIX_CLIENT_INFO} to \[%{IP:postfix_server_ip}\]:%{INT:postfix_server_port}
POSTFIX_PS_ACCESS %{POSTFIX_PS_ACCESS_ACTION:postfix_postscreen_access} %{POSTFIX_CLIENT_INFO}
POSTFIX_PS_NOQUEUE %{POSTFIX_SMTPD_NOQUEUE}
POSTFIX_PS_TOOBUSY NOQUEUE: reject: CONNECT from %{POSTFIX_CLIENT_INFO}: %{GREEDYDATA:postfix_postscreen_toobusy_data}
POSTFIX_PS_DNSBL %{POSTFIX_PS_VIOLATION:postfix_postscreen_violation} rank %{INT:postfix_postscreen_dnsbl_rank} for %{POSTFIX_CLIENT_INFO}
POSTFIX_PS_CACHE cache %{DATA} full cleanup: retained=%{NUMBER:postfix_postscreen_cache_retained} dropped=%{NUMBER:postfix_postscreen_cache_dropped} entries
POSTFIX_PS_VIOLATIONS %{POSTFIX_PS_VIOLATION:postfix_postscreen_violation}( %{INT})?( after %{NUMBER:postfix_postscreen_violation_time})? from %{POSTFIX_CLIENT_INFO}( after %{POSTFIX_SMTP_STAGE:postfix_smtp_stage})?

# dnsblog patterns
POSTFIX_DNSBLOG_LISTING addr %{IP:postfix_client_ip} listed by domain %{HOST:postfix_dnsbl_domain} as %{IP:postfix_dnsbl_result}

# tlsproxy patterns
POSTFIX_TLSPROXY_CONN (DIS)?CONNECT( from)? %{POSTFIX_CLIENT_INFO}

# anvil patterns
POSTFIX_ANVIL_CONN_RATE statistics: max connection rate %{NUMBER:postfix_anvil_conn_rate}/%{POSTFIX_TIME_UNIT:postfix_anvil_conn_period} for \(%{DATA:postfix_service}:%{IP:postfix_client_ip}\) at %{SYSLOGTIMESTAMP:postfix_anvil_timestamp}
POSTFIX_ANVIL_CONN_CACHE statistics: max cache size %{NUMBER:postfix_anvil_cache_size} at %{SYSLOGTIMESTAMP:postfix_anvil_timestamp}
POSTFIX_ANVIL_CONN_COUNT statistics: max connection count %{NUMBER:postfix_anvil_conn_count} for \(%{DATA:postfix_service}:%{IP:postfix_client_ip}\) at %{SYSLOGTIMESTAMP:postfix_anvil_timestamp}

# smtp patterns
POSTFIX_SMTP_DELIVERY %{POSTFIX_KEYVALUE} status=%{WORD:postfix_status}( \(%{GREEDYDATA:postfix_smtp_response}\))?
POSTFIX_SMTP_CONNERR connect to %{POSTFIX_RELAY_INFO}: (Connection timed out|No route to host|Connection refused)
POSTFIX_SMTP_LOSTCONN %{POSTFIX_QUEUEID:postfix_queueid}: %{POSTFIX_LOSTCONN} with %{POSTFIX_RELAY_INFO}

# master patterns
POSTFIX_MASTER_START (daemon started|reload) -- version %{DATA:postfix_version}, configuration %{PATH:postfix_config_path}
POSTFIX_MASTER_EXIT terminating on signal %{INT:postfix_termination_signal}

# bounce patterns
POSTFIX_BOUNCE_NOTIFICATION %{POSTFIX_QUEUEID:postfix_queueid}: sender (non-delivery|delivery status|delay) notification: %{POSTFIX_QUEUEID:postfix_bounce_queueid}

# scache patterns
POSTFIX_SCACHE_LOOKUPS statistics: (address|domain) lookup hits=%{INT:postfix_scache_hits} miss=%{INT:postfix_scache_miss} success=%{INT:postfix_scache_success}
POSTFIX_SCACHE_SIMULTANEOUS statistics: max simultaneous domains=%{INT:postfix_scache_domains} addresses=%{INT:postfix_scache_addresses} connection=%{INT:postfix_scache_connection}
POSTFIX_SCACHE_TIMESTAMP statistics: start interval %{SYSLOGTIMESTAMP:postfix_scache_timestamp}

# aggregate all patterns
POSTFIX_SMTPD %{POSTFIX_SMTPD_CONNECT}|%{POSTFIX_SMTPD_DISCONNECT}|%{POSTFIX_SMTPD_LOSTCONN}|%{POSTFIX_SMTPD_NOQUEUE}|%{POSTFIX_SMTPD_PIPELINING}|%{POSTFIX_TLSCONN}|%{POSTFIX_WARNING}|%{POSTFIX_SMTPD_PROXY}|%{POSTFIX_KEYVALUE}
POSTFIX_CLEANUP %{POSTFIX_CLEANUP_MILTER}|%{POSTFIX_WARNING}|%{POSTFIX_KEYVALUE}
POSTFIX_QMGR %{POSTFIX_QMGR_REMOVED}|%{POSTFIX_QMGR_ACTIVE}|%{POSTFIX_WARNING}
POSTFIX_PIPE %{POSTFIX_PIPE_DELIVERED}|%{POSTFIX_PIPE_FORWARD}
POSTFIX_POSTSCREEN %{POSTFIX_PS_CONNECT}|%{POSTFIX_PS_ACCESS}|%{POSTFIX_PS_NOQUEUE}|%{POSTFIX_PS_TOOBUSY}|%{POSTFIX_PS_CACHE}|%{POSTFIX_PS_DNSBL}|%{POSTFIX_PS_VIOLATIONS}|%{POSTFIX_WARNING}
POSTFIX_DNSBLOG %{POSTFIX_DNSBLOG_LISTING}
POSTFIX_ANVIL %{POSTFIX_ANVIL_CONN_RATE}|%{POSTFIX_ANVIL_CONN_CACHE}|%{POSTFIX_ANVIL_CONN_COUNT}
POSTFIX_SMTP %{POSTFIX_SMTP_DELIVERY}|%{POSTFIX_SMTP_CONNERR}|%{POSTFIX_SMTP_LOSTCONN}|%{POSTFIX_TLSCONN}|%{POSTFIX_WARNING}
POSTFIX_DISCARD %{POSTFIX_KEYVALUE} status=%{WORD:postfix_status}
POSTFIX_LMTP %{POSTFIX_SMTP}
POSTFIX_PICKUP %{POSTFIX_KEYVALUE}
POSTFIX_TLSPROXY %{POSTFIX_TLSPROXY_CONN}
POSTFIX_MASTER %{POSTFIX_MASTER_START}|%{POSTFIX_MASTER_EXIT}
POSTFIX_BOUNCE %{POSTFIX_BOUNCE_NOTIFICATION}
POSTFIX_SENDMAIL %{POSTFIX_WARNING}
POSTFIX_POSTDROP %{POSTFIX_WARNING}
POSTFIX_SCACHE %{POSTFIX_SCACHE_LOOKUPS}|%{POSTFIX_SCACHE_SIMULTANEOUS}|%{POSTFIX_SCACHE_TIMESTAMP}
POSTFIX_TRIVIAL_REWRITE %{POSTFIX_WARNING}
POSTFIX_TLSMGR %{POSTFIX_WARNING}
POSTFIX_LOCAL %{POSTFIX_KEYVALUE}
```
