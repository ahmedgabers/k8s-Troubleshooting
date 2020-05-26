# k8s-Troubleshooting

## Troubleshooting etcd:
#### - Check logs: 
`docker logs etcd
kubectl logs etcd
docker exec etcd etcdctl`
#### - Check /var/lib/etcd
#### - Configure loglevel to be DEBUG:
`curl -XPUT -d '{"Level":"DEBUG"}' --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) https://localhost:2379/config/local/log`

#### Restore loglevel to be INFO:
`curl -XPUT -d '{"Level":"INFO"}' --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) https://localhost:2379/config/local/log`

#### - Getting etcd metrics:
`curl -X GET --cacert $(docker exec etcd printenv ETCDCTL_CACERT) --cert $(docker exec etcd printenv ETCDCTL_CERT) --key $(docker exec etcd printenv ETCDCTL_KEY) https://localhost:2379/config/local/log https://localhost:2379/metrics`

## Troubleshooting kube-controller-manager:
#### - Find current leader:
`kubectl -n kube-system get endpoints kube-controller-manager -o jsonpath='{.metadata.annotations.control-plane\.alpha\.kubernetes\.io/leader}'`

#### - docker logs -f kube-controller-manager

## Troubleshooting kube-scheduler:
#### - Find current leader
#### - docker logs -f kube-scheduler

## Troubleshooting kubelet:

#### docker logs kubelet
`curl -sLk --cacert /etc/kubernetes/pki/kube-ca.pem --cert /etc/kubernetes/pki/kube-service-account-token.pem --key /etc/kubernetes/pki/kube-service-account-token-key.pem https://127.0.0.1:1020/stats`

## Troubleshooting Generic Pods:

`kubectl get events --field-selector involvedObject.kind=Pod`

`kubectl get pods --all-namespaces -o go-template='{{range .items}}{{if eq .status.phase "Pending"}}{{.spec.nodeName}}{{""}}{{.metadata.name}}{{" "}}{{.metadata.namespace}}{{" "}}{{range .status.conditions}}{{.message}}{{";"}}{{end}}{{"\n"}}{{end}}{{end}}'`

`kubectl get nodes -o custom-columns=Name:.metadata.name,OS:.status.nodeInfo.osImage,KERNEL:.status.nodeInfo.kernelVersion,RUNTIME:.status.nodeInfo.containerRuntimeVersion,KUBELET:.status.nodeInfo.kubeletVersion,KUBEPROXY:.status.nodeInfo.kubeProxyVersion`


`kubectl get nodes -o go-template='{{range .items}}{{$node :=.}}{{range .status.conditions}}{{$node.metadata.name}}{{":"}}{{.type}}{{":"}}{{.status}}{{"\n"}}{{end}}{{end}}'`

## DNS:

`kubectl run -it --rm --restart=Never busybox --image=busybox:1.28 -- nslookup kubernetes.default`
`kubectl run busybox --image=busybox:1.28 --restart=Never --rm -it -- nslookup www.google.com`

## Ingress controller:

#### Check responsiveness of Ingress Controller: 
`kubectl -n ingress-nginx get pods -l app=ingress-nginx -o custom-columns=POD:.metadata.name,NODE:.spec.nodeName,IP:.status.podIP --no-headers | while read ingresspod nodename podip; do echo "=> Testing from ${ingresspod} on ${nodename} (${podip})"; curl -o /dev/null --connect-timeout 5 -s -w 'Connect: %{time_connect}\nStart Transfer: %{time_starttransfer}\nTotal: %{time_total}\nResponse code: %{http_code}\n' -k http://${podip}/healthz; done`

#### Check static NGINX config

`for pod in $(kubectl -n ingress-nginx get pods -l app=ingress-nginx -o custom-columns=NAME:.metadata.name --no-headers); do kubectl -n ingress-nginx exec $pod -- cat /etc/nginx/nginx.conf; done`

#### Output hard to diff, use checksum to find differences:

`for pod in $(kubectl -n ingress-nginx get pods -l app=ingress-nginx -o custom-columns=NAME:.metadata.name --no-headers); do echo $pod; kubectl -n ingress-nginx exec $pod -- cat /etc/nginx/nginx.conf | md5; done`

#### View logging:

`kubectl -n ingress-nginx logs -l app=ingress-nginx`

`kubectl -n ingress-nginx edit ds nginx-ingress-controller --v=2 up to --v=5`
