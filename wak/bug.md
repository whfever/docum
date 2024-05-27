# Bug
## k8s
1. pvc status lost  pv edit claimRef  release >>avliable
2. k8s删除pvc terminating 无法删除  patch {metadata}
3. k8s pod status containercreating
kubectl describe pod <pod-name>  
4. pod crashloopbackoff   
RDS  设置白名单 mi
- pvc status terminating 无法回滚
- 强制删除pod
```bash
kubectl patch pv `kubectl get pv | grep Termin | awk '{print $1}'`  -p '{"metadata":{"finalizers":null}}'
kubectl patch volumesnapshot `kubectl get volumesnapshot  | awk '{print $1}'`   --type='json'  -p='[{"op":"replace","path":"/metadata/finalizers","values":"null"}]'
```

```bash

## git
1. rpc failed
```bash
 git config --global http.sslVerify "false"
```