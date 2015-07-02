# Windows Event Log

前面说过如何在 windows 上利用 nxlog 传输日志数据。事实上，对于 windows 本身，也有类似 syslog 的设计，叫 eventlog。本节介绍如何处理 windows eventlog。

## 采集端配置

### logstash 配置

```
input {
    eventlog {
        #logfile =>  ["Application", "Security", "System"]
        logfile =>  ["Security"]
        type => "winevent"
        tags => [ "caen" ]
    }
}
```

### nxlog 配置

```
## This is a sample configuration file. See the nxlog reference manual about the
## configuration options. It should be installed locally and is also available
## online at http://nxlog.org/nxlog-docs/en/nxlog-reference-manual.html
 
## Please set the ROOT to the folder your nxlog was installed into,
## otherwise it will not start.
 
#define ROOT C:\Program Files\nxlog
define ROOT C:\Program Files (x86)\nxlog
 
Moduledir %ROOT%\modules
CacheDir %ROOT%\data
Pidfile %ROOT%\data\nxlog.pid
SpoolDir %ROOT%\data
LogFile %ROOT%\data\nxlog.log
 
<Extension json>
    Module	xm_json
</Extension>	
 
<Input in>
    Module      im_msvistalog
# For windows 2003 and earlier use the following:
#   Module      im_mseventlog
    Exec	to_json();
</Input>
 
<Output out>
    Module      om_tcp
    Host        10.66.66.66
    Port        5140
</Output>
 
<Route 1>
    Path        in => out
</Route>
```

## Logstash 解析配置

```
input {
  tcp {
    codec => "json"
    port => 5140
    tags => ["windows","nxlog"]
    type => "nxlog-json"
  }
} # end input
 
filter {
  if [type] == "nxlog-json" {
    date {
      match => ["[EventTime]", "YYYY-MM-dd HH:mm:ss"]
      timezone => "Europe/London"
    }
    mutate {
        rename => [ "AccountName", "user" ]
        rename => [ "AccountType", "[eventlog][account_type]" ]
        rename => [ "ActivityId", "[eventlog][activity_id]" ]
        rename => [ "Address", "ip6" ]
        rename => [ "ApplicationPath", "[eventlog][application_path]" ]
        rename => [ "AuthenticationPackageName", "[eventlog][authentication_package_name]" ]
        rename => [ "Category", "[eventlog][category]" ]
        rename => [ "Channel", "[eventlog][channel]" ]
        rename => [ "Domain", "domain" ]
        rename => [ "EventID", "[eventlog][event_id]" ]
        rename => [ "EventType", "[eventlog][event_type]" ]
        rename => [ "File", "[eventlog][file_path]" ]
        rename => [ "Guid", "[eventlog][guid]" ]
        rename => [ "Hostname", "hostname" ]
        rename => [ "Interface", "[eventlog][interface]" ]
        rename => [ "InterfaceGuid", "[eventlog][interface_guid]" ]
        rename => [ "InterfaceName", "[eventlog][interface_name]" ]
        rename => [ "IpAddress", "ip" ]
        rename => [ "IpPort", "port" ]
        rename => [ "Key", "[eventlog][key]" ]
        rename => [ "LogonGuid", "[eventlog][logon_guid]" ]
        rename => [ "Message", "message" ]
        rename => [ "ModifyingUser", "[eventlog][modifying_user]" ]
        rename => [ "NewProfile", "[eventlog][new_profile]" ]
        rename => [ "OldProfile", "[eventlog][old_profile]" ]
        rename => [ "Port", "port" ]
        rename => [ "PrivilegeList", "[eventlog][privilege_list]" ]
        rename => [ "ProcessID", "pid" ]
        rename => [ "ProcessName", "[eventlog][process_name]" ]
        rename => [ "ProviderGuid", "[eventlog][provider_guid]" ]
        rename => [ "ReasonCode", "[eventlog][reason_code]" ]
        rename => [ "RecordNumber", "[eventlog][record_number]" ]
        rename => [ "ScenarioId", "[eventlog][scenario_id]" ]
        rename => [ "Severity", "level" ]
        rename => [ "SeverityValue", "[eventlog][severity_code]" ]
        rename => [ "SourceModuleName", "nxlog_input" ]
        rename => [ "SourceName", "[eventlog][program]" ]
        rename => [ "SubjectDomainName", "[eventlog][subject_domain_name]" ]
        rename => [ "SubjectLogonId", "[eventlog][subject_logonid]" ]
        rename => [ "SubjectUserName", "[eventlog][subject_user_name]" ]
        rename => [ "SubjectUserSid", "[eventlog][subject_user_sid]" ]
        rename => [ "System", "[eventlog][system]" ]
        rename => [ "TargetDomainName", "[eventlog][target_domain_name]" ]
        rename => [ "TargetLogonId", "[eventlog][target_logonid]" ]
        rename => [ "TargetUserName", "[eventlog][target_user_name]" ]
        rename => [ "TargetUserSid", "[eventlog][target_user_sid]" ]
        rename => [ "ThreadID", "thread" ]
    }
    mutate {
        remove_field => [
                    "CurrentOrNextState",
                    "Description",
                    "EventReceivedTime",
                    "EventTime",
                    "EventTimeWritten",
                    "IPVersion",
                    "KeyLength",
                    "Keywords",
                    "LmPackageName",
                    "LogonProcessName",
                    "LogonType",
                    "Name",
                    "Opcode",
                    "OpcodeValue",
                    "PolicyProcessingMode",
                    "Protocol",
                    "ProtocolType",
                    "SourceModuleType",
                    "State",
                    "Task",
                    "TransmittedServices",
                    "Type",
                    "UserID",
                    "Version"
                    ]
    }
  }
 
}
```

