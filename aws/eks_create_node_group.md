# 需要创建的角色

## 建与EKS集群关联的OIDC提供商，请按照以下文档创建OIDC提供商。

https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/enable-iam-roles-for-service-accounts.html 

# 远程登录节点，搜集日志的方法

    1. 为当前实例绑定的角色AmazonEKSNodeRole添加AmazonSSMManagedInstanceCore权限，重启当前实例。

    2. 实例重启后，应该可以通过EC2控制台->选择需要登录的实例-> 连接-> 会话管理器登录相关实例。

    3. 登录实例后执行以下命令进行日志收集

    curl -O https://raw.githubusercontent.com/awslabs/amazon-eks-ami/master/log-collector-script/linux/eks-log-collector.sh 


# 创建CNI插件角色

    https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/cni-iam-role.html