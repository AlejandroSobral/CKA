
## Deployment Secret

dep-0.yaml is the original file of the deployment, whereas the dep-1.yaml is the modified version.

To get the base64 encoded version of the parameters, the following has been executed:

```bash
controlplane:~$ echo -n mysql-host | base64 
bXlzcWwtaG9zdA==
controlplane:~$ echo -n root | base64
cm9vdA==
controlplane:~$ echo -n dbpassword | base64
ZGJwYXNzd29yZA==

```