kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/master/bundle.yaml

kubectl port-forward svc/grafana 3000
kubectl port-forward svc/prometheus 9090
