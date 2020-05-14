title: Kubernetes Create User
author: 大丈夫没问题
tags:
  - kubernetes
categories:
  - linux
date: 2020-05-14 21:18:00
---
## Create User

kubectl apply -f eks-admin-service-account.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
```

kubectl apply -f eks-admin-cluster-role-binding.yaml

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-admin
  namespace: kube-system
```

## Get certificate

```
kubectl get secret default-token-cvn2d -o jsonpath="{['data']['ca\.crt']}" | base64 --decode
```

## Get Token

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
```


## Test If the role works

### Setup credential
```
kubectl config set-cluster kubernetes --certificate-authority=ca.crt --server=$K8S_SERVER_URL
kubectl config set-credentials $K8S_USERNAME --token=$K8S_USER_TOKEN
kubectl config set-context aws --cluster=kubernetes --namespace=$K8S_NAMESPACE --user=$K8S_USERNAME
kubectl config use-context aws --user=$K8S_USERNAME
```

### Test command
```
kubectl get pod
```

## Ref

* https://docs.gitlab.com/ee/user/project/clusters/add_remove_clusters.html#existing-eks-cluster