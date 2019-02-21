#### 1. ����CoreDNS
- KubeDNSΪ�ϰ汾��DNS��Ŀǰ�Ƽ�ʹ��CoreDNS
- ����CoreDNS��ʽ����:
```
cd CoreDNS; tree
.
������ coredns.yaml.sed
������ deploy.sh

$ ./deploy.sh | kubectl apply -f -
```
#### 2. �Ƴ����ϵ�kube-dns���鿴������

```
$ kubectl delete --namespace=kube-system deployment kube-dns
$ kubectl get pods -n kube-system |grep coredns;kubectl cluster-info |grep CoreDNS
$ kubectl exec -ti busybox -- nslookup my-apache
```

- ����ο��ĵ���
https://jimmysong.io/kubernetes-handbook/practice/coredns.html
- �ٷ��ṩ�ű���ַ��
https://github.com/coredns/deployment/blob/master/kubernetes/deploy.sh
https://github.com/coredns/deployment/blob/master/kubernetes/coredns.yaml.sed