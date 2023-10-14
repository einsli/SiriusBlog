# k8s 基于 sa 创建 kubeconfig 访问（集群版）

## 1、创建 sa

执行以下命令创建 serviceaccount

```shell
kubectl create sa devops
```

## 2、创建 clusterrole

`clusterrole.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: '2022-12-11T07:37:26Z'
  name: devops
  resourceVersion: '2468820'
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/devops
  uid: 21ba0efa-bca7-4a39-868f-f48858ddb591
rules:
  - apiGroups:
      - '*'
    resources:
      - '*'
    verbs:
      - get
      - list
      - watch
      - create
      - delete
      - update
      - patch
```

## 3、创建 clusterrolebinding

`clusterrolebinding.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: devops
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: devops
subjects:
  - kind: ServiceAccount
    name: devops
    namespace: default
```

## 4、生成 kubeconfig

假设 k8s 的 apiserver 地址为：**<https://192.168.1.236:16443>**

假设 k8s 集群的 ca 证书文件目录为：**/etc/kubernetes/pki/ca.pem**

### 4.1 获取 sa token

```shell
kubectl get secret $(kubectl get secret -A | grep devops | awk '{print $2}') -o=jsonpath='{.data.token}' | base64 -d
```

### 4.2 生成 kubeconfig 步骤

```shell
KUBE_APISERVER=https://192.168.1.236:16443
DEVOPS_TOKEN=`kubectl get secret $(kubectl get secret -A | grep devops | awk '{print $2}') -o=jsonpath='{.data.token}' | base64 -d` # 这里为上一步获取token的命令

# 设置集群信息
kubectl config set-cluster kubernetes \ # kubernetes是集群名字
--certificate-authority=/etc/kubernetes/pki/ca.pem \ # 这个ca是k8s集群的ca证书
--embed-certs=true \
--server=${KUBE_APISERVER} \ # 这个是api-server的连接地址，通过kubectl cluster-info能看到
--kubeconfig=devops.kubeconfig

# 设置客户端认证凭据
kubectl config set-credentials devops-user \ # devops-user表示凭据的用户名
--token=${DEVOPS_TOKEN} \ # token可通过第一个步骤中得到
--kubeconfig=devops.kubeconfig

# 设置集群信息和用户的上下文信息
kubectl config set-context devops@kubernetes \ # 设置上下文名字
--cluster=kubernetes \ # kubernetes集群名字，跟设置集群信息中保持一致
--user=devops-user \ # 第二步设置凭据的用户名
--kubeconfig=devops.kubeconfig

# 切换当前上下文
kubectl config use-context devops@kubernetes \
--kubeconfig=./devops.kubeconfig
```

## 5、命令测试

```shell
kubectl --kubeconfig=./devops.kubeconfig get pod -A
```

## 6、使用

将生成的`devops.config`放到 用户家目录下的`.kube`文件下，并重命名为`config`即可，例如

```shell
mkdir ~/.kube

mv devops.config ~/.kube/config
```
