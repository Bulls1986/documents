SaaS平台整合Windows和Linux主机日志机器对应Docker容器日志的方案
==========================================

*   [SaaS平台整合Windows和Linux主机日志机器对应Docker容器日志的方案](#SaaS整合Windows和Linux容器日志方案-SaaS平台整合Windows和Linux主机日志机器对应Docker容器日志的方案)
    *   [一、总体说明](#SaaS整合Windows和Linux容器日志方案-一、总体说明)
    *   [二、总体架构](#SaaS整合Windows和Linux容器日志方案-二、总体架构)
    
    *   [三、安装步骤](#SaaS整合Windows和Linux容器日志方案-三、安装步骤)
        *   [Graylog安装](#SaaS整合Windows和Linux容器日志方案-Graylog安装)
        *   [Nxlog安装](#SaaS整合Windows和Linux容器日志方案-Nxlog安装)
        *   [Graylog Sidecar安装](#SaaS整合Windows和Linux容器日志方案-GraylogSidecar安装)
    *   [四、Graylog配置](#SaaS整合Windows和Linux容器日志方案-四、Graylog配置)
    *   [五、Linux配置](#SaaS整合Windows和Linux容器日志方案-五、Linux配置)
    *   [六、Windows配置](#SaaS整合Windows和Linux容器日志方案-六、Windows配置)
    *   [七、Docker配置](#SaaS整合Windows和Linux容器日志方案-七、Docker配置)
    *   [八、使用kafka的方案](#SaaS整合Windows和Linux容器日志方案-八、使用kafka的方案)

  

_一、总体说明_
--------

由于以前的SaaS平台只支持Linux主机及容器，而新版本的SaaS平台从功能上要支持Windows主机和容器，因此，对应的日志采集方案需要在以前的Linux主机的基础上添加Windows主机和容器日志的采集。

整个方案在SaaS平台既有的方案中，将Linux主机上使用的系统组件rsyslog换成支持多种平台的nxlog，使用graylog的Collector完成配置的日志采集的集中管理。

  

_二、总体架构_
--------

![](/download/attachments/1639587/log.png?version=3&modificationDate=1536897730295&api=v2)  

---------------------------------------------------------------------------------------------

整体架构如上图所示：

*   在Linux和Windows主机上安装Nxlog和Graylog Sidecar；
*   在Graylog服务端分别问Linux和Windows两类主机配置采集规则；
*   由Graylog Sidecar负责同步配置信息，并管理Nxlog；
*   由Nxlog根据规则将纯文本和json格式的日志通过GELF协议提交到Graylog对应的端口12201和22201，完成日志采集。

_三、安装步骤_
--------

1.  ### _Graylog安装_
    
    _参考：[http://docs.graylog.org/en/2.4/pages/installation.html](http://docs.graylog.org/en/2.4/pages/installation.html)_
    
2.  ### _Nxlog安装_
    
    _Linux参考：[https://nxlog.co/documentation/nxlog-user-guide#deploy_rhel](https://nxlog.co/documentation/nxlog-user-guide#deploy_rhel)_
    
    _Windows参考：[https://nxlog.co/documentation/nxlog-user-guide#deploy_windows](https://nxlog.co/documentation/nxlog-user-guide#deploy_windows)_
    
3.  ### _Graylog Sidecar安装_
    
    _参考：[http://docs.graylog.org/en/2.4/pages/collector_sidecar.html#installation](http://docs.graylog.org/en/2.4/pages/collector_sidecar.html#installation)_
    

_四、Graylog配置_
-------------

1.  _graylog配置两个index，一个给系统日志，一个给应用日志使用；_
2.  _根据配置两个index配置两个stream来对应，使用日志字段"_x\_log\_type_"来区分，为system的进入系统日志index，为application的进入应用日志index；_
3.  _配置GELF的两个input，一个端口为12201，一个端口为22201，分别接受plaintext日志和json格式的日志；在22201的input上配置json的Extractor，用于提取json日志中的所有字段；_
4.  _配置4个Collector配置项，分别为：linux，linux-docker-app，windows，windows-docker-app；_
5.  _在linux配置项中，配置input为tcp-syslog-listener，监听端口514；配置output为gelf-tcp-output，地址为graylog服务器的地址，端口为12201；并添加额外的字段：x_platform=linux，x\_log\_type=system，x\_log\_format=plaintext；Snippet配置为系统默认即可；并设置tag为“linux”；linux“_
6.  __在linux-docker-app配置项中直接配置Nxlog的_Snippet，内容为：___
    
```
<Processor process_buffer_plaintext>
  Module pm_buffer
  MaxSize 16384
  Type Mem
</Processor>
<Processor process_buffer_json>
  Module pm_buffer
  MaxSize 16384
  Type Disk
</Processor>
<Input input_plaintext>
    Module im_tcp
    Host 127.0.0.1
    Port 1514
    Exec parse_syslog_bsd();
</Input>
<Input input_json>
        Module im_tcp
        Host 127.0.0.1
        Port 2514
        Exec parse_syslog_bsd();
</Input>
<Output output_plaintext>
    Module om_tcp
    Host 192.168.2.67
    Port 12201
    OutputType  GELF_TCP
    Exec $short_message = $raw_event; # Avoids truncation of the short_message field.
    Exec $gl2_source_collector = 'b97bae40-4ac5-40b0-8048-64050831ed8a';
    Exec $collector_node_id = 'repository-01';
    Exec $Hostname = hostname_fqdn();
        Exec $x_log_format = "plaintext";
    Exec $x_log_type = "application";
        Exec $x_platform = "linux";
</Output>
<Output output_json>
        Module om_tcp
        Host 192.168.2.67
        Port 22201
        OutputType  GELF_TCP
        Exec $short_message = $raw_event; # Avoids truncation of the short_message field.
        Exec $gl2_source_collector = 'b97bae40-4ac5-40b0-8048-64050831ed8a';
        Exec $collector_node_id = 'repository-01';
        Exec $Hostname = hostname_fqdn();
        Exec $x_log_format = "json";
        Exec $x_log_type = "application";
        Exec $x_platform = "linux";
</Output>
<Route route_plaintext_log>
  Path input_plaintext => process_buffer_plaintext => output_plaintext
</Route>
<Route route_json_log>
  Path input_json => process_buffer_json => output_json
</Route>
```
    
    __，并设置tag为“linux-docker-app”；__
    
7.  _在windows配置项中_配置windows-event-log,内容为：__
    
```
<QueryList>\
    <Query Id="0">\
        <Select Path="Application">*</Select>\
        <Select Path="System">*</Select>\
        <Select Path="Security">*</Select>\
    </Query>\
</QueryList>
```
    
    __；配置output为gelf-tcp-output，地址为graylog服务器的地址，端口为12201；并添加额外的字段：x_platform=windows，x\_log\_type=system，x\_log\_format=plaintext；Snippet配置为系统默认即可；并设置tag为“windows”；__
    
8.  __在windows-docker-app配置项中配置，__

_五、Linux配置_
-----------

_要采集linux系统日志，需要在系统的rsyslog配置中将所有系统日志转发到514端口，因此需要修改/etc/rsyslog.conf文件，添加如下内容在最后一行后，重启rsyslog服务。内容如下：_

**rsyslog配置**

[?](#)

`*.* @``@127``.0.``0.1``:``514`

_六、Windows配置_
-------------

_windows系统日志直接使用windows-event-log进行采集，不需要进行额外配置。_

_七、Docker配置_
------------

__不管是linux容器还是windows容器，_Docker容器启动时根据容器各自的日志输出格式，确定采用syslog输出的地址，对应的启动参数如下：_

**启动参数**

[?](#)

`-``-log``-driver``=syslog`

`-``-log``-opt` `syslog``-address``=tcp://127.0.0.1:2514`

`-``-log``-opt` `tag=nestvision/paas``-spring``-boot``-demo`

_其中：_

1.  _syslog-address的端口由日志格式确定，plaintext格式的日志使用1514端口，json格式的日志使用2514端口；_
2.  tag为容器的镜像名称；
    
3.  跟SaaS平台对接时，在根据具体情况设置tag格式，并在graylog配置解析即可按照提取租户等信息到特定的日志字段；
    

_八、使用kafka的方案_
--------------

_使用kafka的方案如下，前提需要使用nxlog的商用版（商用版才支持写入到kafka中）。该方案的优势在对于海量的日志写入时：_

1.  _使用kafka集群先行接受所有的日志，再使用graylog进行后续处理，避免在海量日志写入时由于graylog日志处理不过来导致的阻塞；_
2.  _避免graylog集群异常停机时，日志的丢失；_

_![](/download/attachments/1639587/log2.png?version=1&modificationDate=1536905453369&api=v2)  
_

_使用GELF Kafka input模块，需要在客户机提交日志时将日志转成GELF所需的格式，以下配置为将syslog日志转成输出到kafka，graylog使用gelf kafka input进行处理的例子：  
_

**示例**

[?](#)

`define ROOT /usr/bin`

`<``Extension` `gelf>`

`Module xm_gelf`

`</``Extension``>`

`<``Extension` `syslog>`

`Module xm_syslog`

`</``Extension``>`

`<``Extension` `json>`

`Module  xm_json`

`</``Extension``>`

`<``Processor` `process_buffer_plaintext>`

`Module pm_buffer`

`MaxSize 16384`

`Type Mem`

`</``Processor``>`

`User root`

`Group root`

`Moduledir /opt/nxlog/libexec/nxlog/modules`

`#Moduledir /usr/libexec/nxlog/modules`

`CacheDir /var/spool/collector-sidecar/nxlog`

`PidFile /var/run/graylog/collector-sidecar/nxlog.pid`

`define LOGFILE /var/log/graylog/collector-sidecar/nxlog.log`

`LogFile %LOGFILE%`

`LogLevel INFO`

`<``Extension` `logrotate>`

`Module  xm_fileop`

`<``Schedule``>`

`When    @daily`

`Exec    file_cycle('%LOGFILE%', 7);`

`</``Schedule``>`

`</``Extension``>`

`<``Input` `input_plaintext>`

`Module im_tcp`

`Host 127.0.0.1`

`Port 1514`

`Exec parse_syslog();`

`Exec $foo = $Message; delete($Message); rename_field("foo","message");`

`Exec $foo = lc($SyslogSeverity); delete($SyslogSeverity); rename_field("foo","syslog_severity");`

`Exec $foo = $SyslogSeverityValue; delete($SyslogSeverityValue); rename_field("foo","syslog_severity_code");`

`Exec $foo = lc($Severity); delete($Severity); rename_field("foo","severity_label");`

`Exec $foo = $SeverityValue; delete($SeverityValue); rename_field("foo","severity");`

`Exec $foo = lc($SyslogFacility); delete($SyslogFacility); rename_field("foo","syslog_facility");`

`Exec $foo = $SyslogFacilityValue; delete($SyslogFacilityValue); rename_field("foo","syslog_facility_code");`

`Exec $foo = $SourceName; delete($SourceName); rename_field("foo","sysloghost");`

`Exec $foo = $ProcessID; delete($ProcessID); rename_field("foo","pid");`

`Exec delete($EventReceivedTime);`

`Exec $host = "repository-01";`

`Exec to_json();`

`</``Input``>`

`<``Output` `output_kafka_plaintext>`

`Module om_kafka`

`BrokerList 192.168.2.67:9092`

`Topic plaintext`

`Exec $gl2_source_collector = 'b97bae40-4ac5-40b0-8048-64050831ed8a';`

`Exec $collector_node_id = 'repository-01';`

`Exec $message = "abcd";`

`Exec $Hostname = hostname_fqdn();`

`Exec $x_log_format = "plaintext";`

`Exec $x_log_type = "system";`

`Exec $x_platform = "linux";`

`</``Output``>`

`<``Route` `route_plaintext_log>`

`Path input_plaintext => process_buffer_plaintext => output_kafka_plaintext`

`</``Route``>`

*   页面:
    
    [SaaS整合Windows和Linux容器日志方案](/pages/viewpage.action?pageId=1639587)
    

