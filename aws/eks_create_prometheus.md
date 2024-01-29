# eks_create_prometheus


## 安装EBS驱动程序
    https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/ebs-csi.html



## 给专用节点添加标签

    kubectl label nodes <node-name> prometheus=dedicated

## 在专用节点上设置污点

    kubectl taint nodes <node-name> prometheus=dedicated:NoSchedule

## 使用 Helm 部署 Prometheus

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

    helm pull prometheus-community/prometheus --untar

### 修改 Helm 配置
    vi prometheus/values.yaml
```yaml
        statefulSet:

            enabled: true


        resources:
            limits:
            cpu: 1000m
            memory: 2048Mi
            requests:
            cpu: 500m
            memory: 512Mi


            statefulsetReplica:
            enabled: true
            replica: 0


        retention: "2d"



        alertmanager:
        ## If false, alertmanager will not be installed
        ##
        enabled: false

        prometheus-pushgateway:
        ## If false, pushgateway will not be installed
        ##
        enabled: false


        storageClass: "gp2"

        size: 20Gi




        tolerations:
            - key: "prometheus"
            operator: "Equal"
            value: "dedicated"
            effect: "NoSchedule"


        affinity:
            nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
                - key: "prometheus"
                operator: In
                values:
                - "dedicated"

```

## 安装
    helm install prometheus ./ -f values.yaml --namespace monitor

## 升级
helm upgrade prometheus ./ -f values.yaml --namespace monitor

## 删除
helm uninstall  prometheus  --namespace monitor