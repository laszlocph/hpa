Create cluster
k3d cluster create laci -p "8081:80@loadbalancer" --k3s-arg '--no-deploy=traefik@server:*' --k3s-arg '--no-deploy=metrics-server@server:*'

Install prometheus-stack and prometheus-adapter
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus-stack prometheus-community/kube-prometheus-stack
helm install prometheus-adapter prometheus-community/prometheus-adapter --values prometheus-adapter-values.yaml

Tutorial
https://github.com/kubernetes-sigs/prometheus-adapter/blob/master/docs/walkthrough.md
https://github.com/kubernetes-sigs/prometheus-adapter/blob/master/docs/walkthrough.md#quantity-values

Troubleshooting

All labels must be applied to ServiceMonitor, otherwise prometheus will not pick it up
kubectl get prometheus prometheus-stack-kube-prom-prometheus -o yaml | grep serviceMonitor -A 3
  serviceMonitorNamespaceSelector: {}
  serviceMonitorSelector:
    matchLabels:
      release: prometheus-stack

Check if ServiceMonitor got picked up
kubectl get secret prometheus-prometheus-stack-kube-prom-prometheus -ojson | jq -r '.data["prometheus.yaml.gz"]' | base64 -d | gunzip | grep -B 10 -A 10 "sample-app"

Check if metric is exposed by k8s api
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests" | jq .
