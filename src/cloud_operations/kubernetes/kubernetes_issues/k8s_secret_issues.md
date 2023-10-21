# k8s 新版本问题处理

#### 1、创建 sa 不会自动生成相应 secret 问题

> k8s serviceaccount 创建后没有生成对应的 secret

方法如下

方式 1 使用 TokenRequest API 来生成 token，获取方式如下

- 使用 client-go 或者其他 api 调用工具来获取某个 serviceaccount 的 token
- 创建 yaml，使用 kubectl apply -f
- 使用 kubectl create token -n xxx <serviceaccount-name> 来获取一个临时的 token, 默认 1 小时

方式 2 创建 secret token，创建后从 secret 的 token 字段拿就可以了

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-sa-sample
  annotations:
    kubernetes.io/service-account.name: 'sa-name' # 这里填写serviceAccountName
type: kubernetes.io/service-account-token
```

### 参考文章

- [参考文章: k8s serviceaccount 创建后没有生成对应的 secret](https://www.soulchild.cn/post/2945)
- [changelog](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md#urgent-upgrade-notes)
- [相关 pr](https://github.com/kubernetes/kubernetes/pull/108309)
