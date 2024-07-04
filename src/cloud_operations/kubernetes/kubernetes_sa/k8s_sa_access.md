# kubereetes 基于 sa 权限访问控制

## 案例一、在 ops 命名空间下，创建 sa-devops 账号，只给读权限，访问集群也是 view

### 1、在对应 namespace 创建 sa

`service_account.yaml`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-devops
  namespace: ops
```

```shell
kubectl apply -f service_account.yaml
```

**注： 从 1.24 开始创建 sa 就不会自动生成 secret 了,需要手动创建**

### 2、创建 secret

`account_secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sa-devops
  annotations:
    kubernetes.io/service-account.name: 'sa-devops' # 这里填写serviceAccountName
type: kubernetes.io/service-account-token
```

```shell
kubectl apply -f account_secret.yaml
```

### 3、创建 Role

它定义了该命名空间内的访问权限

`account_role.yaml```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: ops
  name: view-role
rules:
  - apiGroups: ['']
    resources: ['*']
    verbs: ['get', 'list', 'watch']
  - apiGroups: ['apps']
    resources: ['deployments']
    verbs: ['get', 'list', 'watch']
```

```shell
kubectl apply -f account_role.yaml
```

### 4、创建 RoleBinding

将 ServiceAccount 绑定到 Role

`account_role_binding.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  annotations:
  name: view-role-binding
  namespace: ops
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: view-role
subjects:
  - kind: ServiceAccount
    name: view-user
    namespace: app-prod
```

```
kubectl apply -f account_role_binding.yaml
```

### 5、创建 clusterrole

`clusterrole.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-view-role
rules:
  - apiGroups: [''] # "" 表示核心 API 组
    resources: ['*'] # 所有资源
    verbs: ['get', 'list', 'watch'] # view 权限
```

```shell
kubectl apply -f clusterrole.yaml
```

### 6、创建 clusterrolebinding

`clusterrolebinding.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: view-role-binding
subjects:
  - kind: ServiceAccount
    name: view-user
    namespace: app-prod
roleRef:
  kind: ClusterRole
  name: cluster-view-role
  apiGroup: rbac.authorization.k8s.io
```

```shell
kubectl apply -f clusterrolebinding.yaml
```

### 5、获取访问 token

```shell
kubectl get secret sa-devops -n ops -o jsonpath={".data.token"} | base64 -d
```

这样对整个集群和 ops 命名空间只有只读权限
