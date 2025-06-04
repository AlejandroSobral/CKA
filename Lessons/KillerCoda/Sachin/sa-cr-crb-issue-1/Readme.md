
```bash
controlplane:~$ kubectl auth can-i list pods --as=system:serviceaccount:default:prod-sa
yes
controlplane:~$ kubectl auth can-i create services --as=system:serviceaccount:default:prod-sa
no
controlplane:~$ kubectl auth can-i get services --as=system:serviceaccount:default:prod-sa
no
controlplane:~$ kubectl auth can-i list services --as=system:serviceaccount:default:prod-sa
```



kubectl auth can-i list pods --as=system:serviceaccount:default:prod-sa

kubectl auth can-i create services --as=system:serviceaccount:default:prod-sa
kubectl auth can-i get services --as=system:serviceaccount:default:prod-sa
kubectl auth can-i list services --as=system:serviceaccount:default:prod-sa