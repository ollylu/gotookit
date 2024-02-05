
## k8s log 2 cloudwatch log

### 第一步: (无需创建NS，ServcieAcount)
    https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/Container-Insights-prerequisites.html

### 第二步:
    https://docs.aws.amazon.com/zh_cn/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs-FluentBit.html#Container-Insights-FluentBit-setup


### 第三步:
    kubectl edit serviceaccount fluent-bit -n amazon-cloudwatch
    添加注释绑定角色:

    annotations:
        eks.amazonaws.com/role-arn: arn:aws:iam::$account_id:role/olly-eks-log-role


### 第三步:
    kubectl edit configmap fluent-bit-config -n amazon-cloudwatch

    1、配置 lua 脚本 configmap
```yaml
apiVersion: v1
data:
  construct_log_group_name.lua: |
    function construct_log_group_name(tag, timestamp, record)
        local pod_name = record["kubernetes"]["pod_name"]
        local namespace = record["kubernetes"]["namespace_name"]
        local deployment_name = string.match(pod_name, "(.+)-[a-z0-9]+-[a-z0-9]+$")

        if deployment_name and namespace then
            local region = os.getenv("AWS_REGION") or "us-east-1"
            local cluster_name = os.getenv("CLUSTER_NAME") or "ollylu-test"
            local log_group_name = string.format("%s-%s-%s-%s", region, cluster_name, namespace, deployment_name)

            record["kubernetes"]["log_group_name"] = log_group_name
        end

        return 1, timestamp, record
    end
kind: ConfigMap
metadata:
  name: fluent-bit-lua-scripts
  namespace: amazon-cloudwatch
```
    2、对DaemonSet配置加载lua脚本
```yaml
      volumes:
      - configMap:
          defaultMode: 420
          name: fluent-bit-lua-scripts
        name: lua-scripts
```
```yaml
        volumeMounts:
        - mountPath: /fluent-bit/etc/lua
          name: lua-scripts
```
    3、配置对所有deployment的日志采集

```yaml
  application-log.conf: |
    [INPUT]
        Name                tail
        Tag                 application.*
        Path                /var/log/containers/fluent-bit*
        multiline.parser    docker, cri
        DB                  /var/fluent-bit/state/flb_log.db
        Mem_Buf_Limit       5MB
        Skip_Long_Lines     On
        Refresh_Interval    10
        Read_from_Head      ${READ_FROM_HEAD}

    [INPUT]
        Name                tail
        Tag                 application.*
        Path                /var/log/containers/cloudwatch-agent*
        multiline.parser    docker, cri
        DB                  /var/fluent-bit/state/flb_cwagent.db
        Mem_Buf_Limit       5MB
        Skip_Long_Lines     On
        Refresh_Interval    10
        Read_from_Head      ${READ_FROM_HEAD}

    [FILTER]
        Name                kubernetes
        Match               application.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_Tag_Prefix     application.var.log.containers.
        Merge_Log           On
        Merge_Log_Key       log_processed
        K8S-Logging.Parser  On
        K8S-Logging.Exclude Off
        Labels              Off
        Annotations         Off
        Use_Kubelet         On
        Kubelet_Port        10250
        Buffer_Size         0

    [OUTPUT]
        Name                cloudwatch_logs
        Match               application.*
        region              ${AWS_REGION}
        log_group_name      /aws/containerinsights/${CLUSTER_NAME}/application
        log_stream_prefix   ${HOST_NAME}-
        auto_create_group   true
        extra_user_agent    container-insights
```


