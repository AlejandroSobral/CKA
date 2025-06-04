controlplane:~$ kubectl auth can-i list pods --as=system:serviceaccount:default:dev-sa        
yes
controlplane:~$ kubectl auth can-i get pods --as=system:serviceaccount:default:dev-sa 
yes
controlplane:~$ kubectl auth can-i list pods --as=system:serviceaccount:default:dev-sa 
yes
controlplane:~$ kubectl auth can-i list service --as=system:serviceaccount:default:dev-sa 
yes
controlplane:~$ kubectl auth can-i get service --as=system:serviceaccount:default:dev-sa 
yes
controlplane:~$ kubectl auth can-i create service --as=system:serviceaccount:default:dev-sa 
yes
controlplane:~$ kubectl auth can-i create pods --as=system:serviceaccount:default:dev-sa 
yes