rnd-cluster1

setup of grafana loki


helm install
helm repo add grafana https://grafana.github.io/helm-charts 
Kubectl create ns monitoring 
Kubectl apply -n monitoring pvc.yaml
helm install loki grafana/loki-stack --namespace monitoring --create-namespace --values loki-stack-values.yaml  
kubectl get secret --namespace monitoring loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode; echo 
kubectl apply -f monitor-ingress.yaml

so now address is not coming so install nginx-ingress controller.

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml

pods was in pending as nodes were tainted.
kubectl taint nodes rnd-node-pool-tllr2 workloadKind=database:NoSchedule-

rollout restart nginx pods
jobs also if  poss, either delete pod

kubectl get svc -n ingress-nginx


kubectl -n monitoring get ingress

takje the ip, map in cloudflare

access the url : https://monitor-test.skill-mine.com/login

so now grafana/loki setup is done with the above steps
-----------------------------------------


now promtail setup in both clusters : 

existing cluster where grafana/loki is setup: 


1. helm install promtail grafana/promtail --create-namespace -n monitoring --values <promtail-values.yaml path>
2. kubectl -n monitoring get po 
3. In grafana-ui >> explore >> data-source >> labels >> cluster >> logs



so this done.

so now create 1 loki node-port service? why?? 
so basically ye cluster log push kr pa ra h kyuki yeah p cluster-ip service lga h so within the clustwer accessibke h. but new cluster me yea other cluster me if you want this service to be accesible you need nodeport service. 

if you want this service to be accessible from other cluster we need to expose this service to nodeport service. 


To expose your Loki service using a NodePort, you need to create a new Service or patch the existing one.

kubectl expose service loki -n monitoring --type=NodePort --name=loki-nodeport --port=3100 --target-port=3100


--------------------------------------------

so now in other cluster: 

- helm install promtail grafana/promtail --create-namespace -n monitoring --values promtail-values.yaml

now in promtails.yaml: 

promtail-values.yaml:-

tolerations:
  - operator: Exists
config:
  clients:
    - url: http://loki-port.skill-mine.com:32595/loki/api/v1/push
      external_labels:
        cluster: rnd-cluster-3  #existing cluster name


so now in grafana/loki cluster setup:

kubectl get no  -o wide
take any external-ip: 209.38.121.59
take the port of the service nodeport: 31915


promtail-values.yaml:-

tolerations:
  - operator: Exists
config:
  clients:
    - url: http://209.38.121.59:31915/loki/api/v1/push
      external_labels:
        cluster: rnd-cluster-3


so now install this in the new cluster: 

helm install promtail grafana/promtail --create-namespace -n monitoring --values promtail-values.yaml

promtail-values.yaml:-

tolerations:
  - operator: Exists
config:
  clients:
    - url: http://209.38.121.59:31915/loki/api/v1/push
      external_labels:
        cluster: rnd-cluster-3


now if you will check in grafana ui, you will get the cluster. 
