kubectl的几条命令

```shell
kubectl get pods
kubetctl get pods -o wide
kubectl describe pod <pod_name>
kubectl get node -o wide
kubectl get service -o wide
kubectl get namespace -o wide
kubectl create -f file.yaml # 使用yaml文件创建pod
kubectl exec nginx-app ps aux
kubectl logs nginx-app
kubectl get pod nginx-app -o yaml
kubectl get pods nginx-app -o yaml
kubectl run --image=nginx:alpine nginx-app --port=80 # 使用run命令创建pod
```

