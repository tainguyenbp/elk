# ELK
```
helm repo add fluent https://fluent.github.io/helm-charts

helm show values fluent/fluent-bit > fluentbit-values.yaml

```



### fluent config 2
```
luaScripts:
  setIndex.lua: |
    function set_index(tag, timestamp, record)
        index = "somePrefix-"
        if record["kubernetes"] ~= nil then
            if record["kubernetes"]["namespace_name"] ~= nil then
                if record["kubernetes"]["container_name"] ~= nil then
                    record["es_index"] = index
                        .. record["kubernetes"]["namespace_name"]
                        .. "-"
                        .. record["kubernetes"]["container_name"]
                    return 1, timestamp, record
                end
                record["es_index"] = index
                    .. record["kubernetes"]["namespace_name"]
                return 1, timestamp, record
            end
        end
        return 1, timestamp, record
    end
## https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/classic-mode/configuration-file
config:
  service: |
    [SERVICE]
        Daemon Off
        Flush {{ .Values.flush }}
        Log_Level {{ .Values.logLevel }}
        Parsers_File /fluent-bit/etc/parsers.conf
        Parsers_File /fluent-bit/etc/conf/custom_parsers.conf
        HTTP_Server On
        HTTP_Listen 0.0.0.0
        HTTP_Port {{ .Values.metricsPort }}
        Health_Check On

  ## https://docs.fluentbit.io/manual/pipeline/inputs
  inputs: |
    [INPUT]
        Name tail
        Path /var/log/containers/*.log
        multiline.parser docker, cri
        Tag kube.*
        Mem_Buf_Limit 5MB
        Skip_Long_Lines On

    [INPUT]
        Name systemd
        Tag host.*
        Systemd_Filter _SYSTEMD_UNIT=kubelet.service
        Read_From_Tail On

  ## https://docs.fluentbit.io/manual/pipeline/filters
  filters: |
    [FILTER]
        Name kubernetes
        Match kube.*
        Merge_Log On
        Keep_Log Off
        K8S-Logging.Parser On
        K8S-Logging.Exclude On

    [FILTER]
        Name lua
        Match kube.*
        script /fluent-bit/scripts/setIndex.lua
        call set_index

  ## https://docs.fluentbit.io/manual/pipeline/outputs
  outputs: |
    [OUTPUT]
        Name es
        Match kube.*
        Type  _doc
        Host elasticsearch-master
        Port 9200
        HTTP_User elastic
        HTTP_Passwd e7uafnP9WARJEaZX
        tls On
        tls.verify Off
        Logstash_Format On
        Logstash_Prefix logstash
        Retry_Limit False
        Suppress_Type_Name On

    [OUTPUT]
        Name es
        Match host.*
        Type  _doc
        Host elasticsearch-master
        Port 9200
        HTTP_User elastic
        HTTP_Passwd e7uafnP9WARJEaZX
        tls On
        tls.verify Off
        Logstash_Format On
        Logstash_Prefix node
        Retry_Limit False
        Suppress_Type_Name On
```

### fluent config
```
config:
  service: |
    [SERVICE]
        Daemon Off
        Flush {{ .Values.flush }}
        Log_Level {{ .Values.logLevel }}
        Parsers_File /fluent-bit/etc/parsers.conf
        Parsers_File /fluent-bit/etc/conf/custom_parsers.conf
        HTTP_Server On
        HTTP_Listen 0.0.0.0
        HTTP_Port {{ .Values.metricsPort }}
        Health_Check On

  ## https://docs.fluentbit.io/manual/pipeline/inputs
  inputs: |
    [INPUT]
        Name tail
        Path /var/log/containers/*.log
        multiline.parser docker, cri
        Tag kube.*
        Mem_Buf_Limit 5MB
        Skip_Long_Lines On

    [INPUT]
        Name systemd
        Tag host.*
        Systemd_Filter _SYSTEMD_UNIT=kubelet.service
        Read_From_Tail On

  ## https://docs.fluentbit.io/manual/pipeline/filters
  filters: |
    [FILTER]
        Name kubernetes
        Match kube.*
        Merge_Log On
        Keep_Log Off
        K8S-Logging.Parser On
        K8S-Logging.Exclude On


  ## https://docs.fluentbit.io/manual/pipeline/outputs
  outputs: |
    [OUTPUT]
        Name es
        Match kube.*
        Index fluent-bit
        Type  _doc
        Host elasticsearch-master
        Port 9200
        HTTP_User elastic
        HTTP_Passwd e7uafnP9WARJEaZX
        tls On
        tls.verify Off
        Logstash_Format On
        Logstash_Prefix logstash
        Retry_Limit False
        Suppress_Type_Name On

    [OUTPUT]
        Name es
        Match host.*
        Index fluent-bit
        Type  _doc
        Host elasticsearch-master
        Port 9200
        HTTP_User elastic
        HTTP_Passwd e7uafnP9WARJEaZX
        tls On
        tls.verify Off
        Logstash_Format On
        Logstash_Prefix node
        Retry_Limit False
        Suppress_Type_Name On
```

### Ref link: https://medium.com/@tech_18484/simplifying-kubernetes-logging-with-efk-stack-158da47ce982
