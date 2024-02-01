
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
