## K8s System Troubleshooting

When running a local K8s cluster using `microk8s`, the following system pods are expected to be in `Running` state as part of normal system operation: 

```
ingress       nginx-ingress-microk8s-controller-xbd58    0/1     Running   0          7s
kube-system   calico-kube-controllers-658d97c59c-brp9g   1/1     Running   0          7s
kube-system   calico-node-gwngs                          0/1     Running   0          7s
kube-system   coredns-864597b5fd-hm8qv                   1/1     Running   0          7s
```

To get the list of pods, including system pods, run `kubectl get pods --all-namespaces`.
If any of the system pods (calico, coredns, or nginx) or not in a Running state, or not found, try to recover them by killing the pods:  

```
kubectl delete pods --all --all-namespaces --force || true
```

The system pods should be automatically restarted by K8s.

Apply the following steps if any of the system pods is missing:
* **calico** - re-apply the Calico manifest:
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
* **coredns** - disable and re-enable dns:
```
microk8s disable dns
microk8s enable dns
```
* **nginx** - disable and re-enable ingress:
```
microk8s disable ingress
microk8s enable ingress
```

After each step, inspect the status of the pods. If the pods do not show as Running, repeat the pods kill step above.
If the steps above did not help, try to stop and start microk8s again:

```
microk8s stop
microk8s start
```

Then re-inspect the status of pods.
If none of the above steps helped, try to remove microk8s and re-install it:

```
sudo snap remove microk8s
sudo snap install microk8s --classic
```
