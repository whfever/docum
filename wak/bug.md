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

## Docker
- docker engine stopped

```bash
wsl --update
```


## 联调
1. nginx代理
   
   https访问cors  -》http并且带#



# NR
## stn gid null   
1.    set gid=""  repeated
2.   nwgid.sh
## transformwinging  gid   null
"上一级未编码"
ldap_tool -s --name 南网/总调直调/福成变/变压器.#1B
ldap_tool -s --gid 1300F130001420TAB00BAB001
删了重编
1. update  powertransformer set gid=null 
2. nwgid110 --gen-gid powertransformer 117093590328411129 

## transformwinging_2  gid   null

select * from nwgid_tw_sub_ where STN_NID = '113997365584593076'  >>  column:[name]  ,[tran_nid]

select * from powertransformer where nid = '上面的tran_nid'  >> sname

1. update sname ="#1B"
2.  update  powertransformer set gid=null 
3. nwgid110 --gen-gid powertransformer 117093590328411130


1. bakup
   systemctl status telnet.socket
   telnet   ip
ssh -V
mv /etc/ssh /etc/ssh.bak
rpm -qa | grep ssh
2. 
tar xzf openssh-9.0p1.tar.gz
cd
./configure --sysconfdir=/etc/ssh

cp -a contrib/redhat/sshd.init /etc/init.d/sshd
 cp -a /usr/local/bin/ssh-keygen /usr/bin/ssh-keygen