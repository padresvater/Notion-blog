---
title: "Graylog自定义钉钉报警"
date: "2025-06-23T09:50:00.000Z"
lastmod: "2025-08-15T10:08:00.000Z"
draft: false
featuredImage: "https://www.notion.so/images/page-cover/nasa_eva_during_skylab_3.jpg"
series: []
authors:
  - "Padresvater"
tags:
  - "SOC"
  - "Graylog"
categories:
  - "网络安全"
NOTION_METADATA:
  object: "page"
  id: "21b52bdf-d31a-80ee-adad-daf931b73713"
  created_time: "2025-06-23T09:50:00.000Z"
  last_edited_time: "2025-08-15T10:08:00.000Z"
  created_by:
    object: "user"
    id: "178d872b-594c-8172-aca1-00024d90b8f5"
  last_edited_by:
    object: "user"
    id: "178d872b-594c-8172-aca1-00024d90b8f5"
  cover:
    type: "external"
    external:
      url: "https://www.notion.so/images/page-cover/nasa_eva_during_skylab_3.jpg"
  icon: null
  parent:
    type: "database_id"
    database_id: "19152bdf-d31a-8124-9a5a-d4550863a1d2"
  archived: false
  in_trash: false
  properties:
    series:
      id: "B%3C%3FS"
      type: "multi_select"
      multi_select: []
    draft:
      id: "JiWU"
      type: "checkbox"
      checkbox: false
    authors:
      id: "bK%3B%5B"
      type: "people"
      people:
        - object: "user"
          id: "178d872b-594c-8172-aca1-00024d90b8f5"
          name: "Padresvater"
          avatar_url: null
          type: "person"
          person:
            email: "margie.drennon.795@my.csun.edu"
    custom-front-matter:
      id: "c~kA"
      type: "rich_text"
      rich_text: []
    tags:
      id: "jw%7CC"
      type: "multi_select"
      multi_select:
        - id: "7bc5806d-2e41-4f6f-ac0c-351eecbfa4ac"
          name: "SOC"
          color: "purple"
        - id: "f48cc65a-f85b-48b6-9e59-c4adbe03003e"
          name: "Graylog"
          color: "green"
    categories:
      id: "nbY%3F"
      type: "multi_select"
      multi_select:
        - id: "94713dd6-904b-4c01-be60-e22f7b310b64"
          name: "网络安全"
          color: "default"
    Last edited time:
      id: "vbGE"
      type: "last_edited_time"
      last_edited_time: "2025-08-15T10:08:00.000Z"
    summary:
      id: "x%3AlD"
      type: "rich_text"
      rich_text: []
    Name:
      id: "title"
      type: "title"
      title:
        - type: "text"
          text:
            content: "Graylog自定义钉钉报警"
            link: null
          annotations:
            bold: false
            italic: false
            strikethrough: false
            underline: false
            code: false
            color: "default"
          plain_text: "Graylog自定义钉钉报警"
          href: null
  url: "https://www.notion.so/Graylog-21b52bdfd31a80eeadaddaf931b73713"
  public_url: null
MANAGED_BY_NOTION_HUGO: true

---


### **日志转发**


Syslog(UDP)统一转发至192.168.22.55:1514


### **Input**


接受日志的节点配置信息如下：


```text
allow_override_date: true
bind_address: 0.0.0.0
charset_name: UTF-8
expand_structured_data: false
force_rdns: false
number_worker_threads: 6
override_source: <empty>
port: 1514
recv_buffer_size: 262144
store_full_message: false
timezone: Asia/Shanghai
```


如需要可额外增加节点，端口配置和以下一致：


```text
- 1514:1514/tcp   # Syslog
- 1514:1514/udp   # Syslog
- 12201:12201/tcp # GELF
- 12201:12201/udp # GELF
```


### **Indices**


用于存储日志，建议一个组件对应一个indices


示例配置以Jumpserver为例（几乎默认配置即可）：


```text
Index prefix:jumpserver_
Shards:1
Replicas:0
Field type refresh interval:5 seconds
Field type profile:Not set
Rotation strategy:Data Tiering
Max. in storage:40 days
Min. in storage:30 days
```


### **Stream**


![](https://alidocs.dingtalk.com/core/api/resources/img/5eecdaf48460cde5158a9e3d63ed0396d0f8e7cf0a29b9e775b8339e1c4c2483a3fbd8acc1090b98fb53e1e6afc12419a156a98577f418d5a53e5f35757e877365fe51156a743dbdb8726b2f4bc8a23d8feb33cb514b937753b6f9242b6be37f?tmpCode=9747d31f-e02a-4ca5-bcb3-cdc8ff4ed3f1)


利用_Rules_来分流形成不同的_Stream_，往往使用这一类报文的**唯一标识字段**_：_


| _**application_name**_ must** **match exactly _**suricata**_
形成suricata stream | _**source**_ must** **match exactly _**jumpserver:**_
形成jumpserver stream |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------- |


### **解析日志分割多字段**


原始日志往往是一长串字符串：


jumpserver: operation_log - {"action": {"label": "Export", "value": "export"}, "datetime": "2025/06/04 18:23:05 +0800", "id": "863130fd-ab64-443f-a2cb-af7b6f3cd368", "org_id": "00000000-0000-0000-0000-000000000002", "org_name": "DEFAULT", "remote_addr": "192.168.109.42", "resource": "Export filtered", "resource_type": "Account", "user": "Administrator(admin)"}


不将json字段分离出来就不能根据某一或某些字段做进一步的区分形成更细分的_stream_从而告警。


解析字段有两种方法：_Extractors_和_Pipelines_


[child_database](25052bdf-d31a-80c5-a31d-e558c1ad041a)


### **Alerts**


得到更细分的字段以后就能将此字段作为Rule分流新的Stream，并以此Stream中的内容作为告警的来源。


以suricata日志中_priority_ = 1的报文为例，此流为[suricata Critical Alerts](http://192.168.22.55:9000/streams/683e9b7c2a907533b84097d3/search)（已知的、成功的攻击，或者对系统/网络有直接威胁的活动）：


### **Events**


为上面的需要告警的Stream创造Event，配置可参考[Graylog - View "Critical Priority 1 Event" Event Definition](http://192.168.22.55:9000/alerts/definitions/683ea0bf2a907533b840a951)


_Fields_字段连接日志源字段内容和告警消息内容，使得告警客户端（钉钉机器人、邮件等）能读取并发送选中字段内容：


| 日志原字段（部分）      | Event -> Fields（部分）      | 告警模板引用格式（部分）                   |
| -------------- | ------------------------ | ------------------------------ |
| alert_message  | ${source.alert_message}  | ${event.fields.alert_message}  |
| classification | ${source.classification} | ${event.fields.classification} |


不设置Fields字段，在告警模板中直接使用${event.<……>}也可以。


最后notification添加配置好的告警客户端。


### **钉钉机器人告警**


在钉钉群中添加webhook机器人，安全验证方式可选择限定IP（公网IP），复制_webhook url_。


alert->notification中配置：


Title


Suricata alert Dingtalk notification customer


Description


钉钉群机器人


Notification Type


http-notification-v2


URL


https://oapi.dingtalk.com/robot/send?access_token=


Basic Authentication

- 

API Key/Secret Sent As

- 

API Key

- 

API Secret

- 

Method


POST


Time Zone


Asia/Shanghai


Content Type


JSON


Body Template


```text
{
 "msgtype": "markdown",
 "markdown": {
 "title": "#### Suricata 安全告警",

 "text": "**消息**: ${event.fields.alert_message}

 **分类**: ${event.fields.classification}

 **源IP**: ${event.fields.source_ip}:${event.fields.source_port}

 **目标IP**: ${event.fields.destination_ip}:${event.fields.destination_port}

 **协议**: ${event.fields.protocol}

 **时间**: ${event.timestamp}"
 }
}
```


### **更新**


[Upgrade Graylog in Docker](https://go2docs.graylog.org/current/upgrading_graylog/upgrading_graylog_in_docker.htm)

