# kubernetes-ftp
Demo Deploy FTP Service [fauria/vsftpd] on Kubernetes

https://github.com/fauria/docker-vsftpd

Deploy service:

```shell
kubectl create -f task-pv-volume.yaml

kubectl get pv task-pv-volume

kubectl create -f task-pv-claim.yaml

kubectl get pvc task-pv-claim

kubectl create -f ftp-deployment.yaml

kubectl create -f ftp-service.yaml

```

Access the service in the K8s cluster:

```bash
kubectl run -it ftp-client --rm --image=maurosoft1973/alpine-lftp --command -- \
  lftp -u user,pass1234 my-ftp-service.default.svc.cluster.local
```

Expose the service to outside by using ingress-nginx:

```bash
kubectl patch configmap tcp-services -n ingress-nginx --patch '{"data":{"21":"default/my-ftp-service:21","31100":"default/my-ftp-service:31100","31101":"default/my-ftp-service:31101","31102":"default/my-ftp-service:31102","31103":"default/my-ftp-service:31103","31104":"default/my-ftp-service:31104","31105":"default/my-ftp-service:31105"}}'

kubectl create -f ftp-ingress.yaml

# Note: the following steps should be done only if it's a minikube cluster
# See: https://minikube.sigs.k8s.io/docs/tutorials/nginx_tcp_udp_ingress/
cat <<-"EOF" > /tmp/ingress-nginx-controller-patch.yaml
spec:
  template:
    spec:
      containers:
      - name: controller
        ports:
         - containerPort: 21
           hostPort: 21
         - containerPort: 31100
           hostPort: 31100
         - containerPort: 31101
           hostPort: 31101
         - containerPort: 31102
           hostPort: 31102
         - containerPort: 31103
           hostPort: 31103
         - containerPort: 31104
           hostPort: 31104
         - containerPort: 31105
           hostPort: 31105
EOF

kubectl patch deployment ingress-nginx-controller --patch "$(cat /tmp/ingress-nginx-controller-patch.yaml)" -n ingress-nginx
```

```bash
# access the service via load balancer
lftp -u user,pass1234 <LB external ip>
```
