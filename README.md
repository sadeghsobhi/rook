# Quick Start:
https://rook.io/docs/rook/latest/Getting-Started/quickstart/
## A simple Rook cluster is created for Kubernetes with the following kubectl commands and example manifests.
```
git clone --single-branch --branch master https://github.com/rook/rook.git
cd rook/deploy/examples
kubectl create -f crds.yaml -f common.yaml -f operator.yaml
kubectl create -f cluster.yaml
```
## After run cluster show more pod
```
 kubectl get po -n rook-ceph
NAME                                              READY   STATUS      RESTARTS       AGE
csi-cephfsplugin-8lt4p                            2/2     Running     6 (43h ago)    44h
csi-cephfsplugin-m67f7                            2/2     Running     4 (44h ago)    44h
csi-cephfsplugin-provisioner-55588874-5jm9t       5/5     Running     10 (44h ago)   44h
csi-cephfsplugin-provisioner-55588874-pzczh       5/5     Running     15 (43h ago)   44h
csi-cephfsplugin-pzsrj                            2/2     Running     4 (44h ago)    44h
csi-cephfsplugin-r855q                            2/2     Running     0              44h
csi-cephfsplugin-w6b6g                            2/2     Running     0              44h
csi-cephfsplugin-zzrmg                            2/2     Running     0              44h
csi-rbdplugin-2m26z                               2/2     Running     0              44h
csi-rbdplugin-b5gwh                               2/2     Running     4 (44h ago)    44h
csi-rbdplugin-cmm7v                               2/2     Running     0              44h
csi-rbdplugin-dhfvc                               2/2     Running     6 (43h ago)    44h
csi-rbdplugin-provisioner-577dff4756-76lv4        5/5     Running     15 (43h ago)   44h
csi-rbdplugin-provisioner-577dff4756-l4fcx        5/5     Running     10 (44h ago)   44h
csi-rbdplugin-rbf8w                               2/2     Running     4 (44h ago)    44h
csi-rbdplugin-wp2f7                               2/2     Running     0              44h
rook-ceph-crashcollector-so-m2-69cc48678-q9rst    1/1     Running     0              9h
rook-ceph-crashcollector-so-w1-67469dd69c-fmnl2   1/1     Running     0              44h
rook-ceph-crashcollector-so-w3-6978ddb7b9-g8jbz   1/1     Running     0              9h
rook-ceph-crashcollector-so-w4-77ddcdc68-4n6n6    1/1     Running     0              43h
rook-ceph-mgr-a-58f7f9f48-n7ggm                   3/3     Running     0              9h
rook-ceph-mgr-b-5f59f54659-f6bz4                  3/3     Running     0              9h
rook-ceph-mon-a-5f9945687c-8txg9                  2/2     Running     4 (44h ago)    44h
rook-ceph-mon-b-ddcbffb94-6kd2s                   2/2     Running     4 (44h ago)    44h
rook-ceph-mon-c-5b5c799456-k9hn2                  2/2     Running     6 (43h ago)    44h
rook-ceph-operator-6c56647556-97x6n               1/1     Running     2 (44h ago)    44h
rook-ceph-osd-0-7667bfb4ff-sx5tc                  2/2     Running     0              44h
rook-ceph-osd-1-7dd7979c9c-f5t2n                  2/2     Running     0              44h
rook-ceph-osd-2-54b445d594-fx97c                  2/2     Running     0              43h
rook-ceph-osd-prepare-so-m1-jwtch                 0/1     Completed   0              5h51m
rook-ceph-osd-prepare-so-m2-44428                 0/1     Completed   0              5h51m
rook-ceph-osd-prepare-so-m3-cmw6d                 0/1     Completed   0              5h51m
rook-ceph-osd-prepare-so-w1-dvcws                 0/1     Completed   0              5h51m
rook-ceph-osd-prepare-so-w3-g464k                 0/1     Completed   0              5h51m
rook-ceph-osd-prepare-so-w4-pdvqm                 0/1     Completed   0              5h51m
rook-ceph-rgw-my-store-a-7f858d8c75-svvst         2/2     Running     0              9h
```
# Ceph Toolbox
https://rook.io/docs/rook/latest/Troubleshooting/ceph-toolbox
## Launch the rook-ceph-tools pod:
```
kubectl create -f deploy/examples/toolbox.yaml
```
## Wait for the toolbox pod to download its container and get to the running state:
```
kubectl -n rook-ceph rollout status deploy/rook-ceph-tools
```
## Once the rook-ceph-tools pod is running, you can connect to it with:
```
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
```
## Example:
```
ceph status
ceph osd status
ceph df
rados df
```
## When you are done with the toolbox, you can remove the deployment:
```
kubectl -n rook-ceph delete deploy/rook-ceph-tools
```
# Ceph Dashboard
https://rook.io/docs/rook/latest/Storage-Configuration/Monitoring/ceph-dashboard
## 1. deploy ingress nginx controller:
```
# always latest ingress-nginx from main branch:
https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```
## 2. Now create the Ingress config:
```
vim rook/deploy/examples/dashboard-ingress-https.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rook-ceph-mgr-dashboard
  namespace: rook-ceph # namespace:cluster
  annotations:
    kubernetes.io/tls-acme: "true"
  #  nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    #nginx.ingress.kubernetes.io/server-snippet: |
    # proxy_ssl_verify off;
spec:
  ingressClassName: "nginx"
  tls:
    - hosts:
        - domainname.com
      secretName: domainname.com
  rules:
    - host: domainname.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: rook-ceph-mgr-dashboard
                port:
                  name: http-dashboard
```
## 3. Create a secret to keep domain certificate inside kubernetes:
```
kubectl create secret tls my-tls-secret --cert=path/to/certificate.crt --key=path/to/private-key.key
```
## 4. Get Login password from secretfile:
```
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
```
## 5. Add haproxy config
```
frontend http_frontend
        mode http
        bind *:80
        bind *:443 ssl crt /etc/ssl/certs/certificate.pem #alpn h2,http/1.1  ssl-min-ver TLSv1.2
        redirect scheme https code 301 if !{ ssl_fc }
        default_backend   http_servers

backend http_servers
        mode http
        balance roundrobin
        option forwardfor
        http-request set-header X-Forwarded-Port %[dst_port]
        http-request add-header X-Forwarded-Proto https if { ssl_fc }
        default-server check maxconn 5000
        server master1 192.168.x.x:(port svc ingress-nginx-controller) check  ssl verify none
        server master2 192.168.x.x:(port svc ingress-nginx-controller)  check  ssl verify none
        server master3 192.168.x.x:(port svc ingress-nginx-controller)  check  ssl verify none
        server worker1 192.168.x.x:(port svc ingress-nginx-controller)  check  ssl verify none
        server worker2 192.168.x.x:(port svc ingress-nginx-controller)  check  ssl verify none
        server worker3 192.168.x.x:(port svc ingress-nginx-controller)  check  ssl verify none
```
# RADOS Gateway (Object Storage S3)
https://rook.io/docs/rook/v1.12/Storage-Configuration/Object-Storage-RGW/object-storage

# Cleanup
 https://rook.io/docs/rook/v1.12/Getting-Started/ceph-teardown




 









