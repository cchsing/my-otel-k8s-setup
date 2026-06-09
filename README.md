# my-otel-k8s-setup

https://www.youtube.com/watch?v=PrIYUO51rMU


```bash
helm search repo open-telemetry --versions
export OTEL_VERSION=0.114.0
helm repo add jetstack https://charts.jetstack.io
export CERTMANAGER_VERSION=v1.20.2

helm install \
cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--version $CERTMANAGER_VERSION \
--set crds.enabled=true \
--set startupapicheck.timeout="5ms"

helm install opentelemetry-operator open-telemetry/opentelemetry-operator \
--namespace opentelemetry-operator-system \
--create-namespace \
--version $OTEL_VERSION \
--values=opentelemetry/values.yaml

helm show values open-telemetry/opentelemetry-operators > values.yaml
kubectl create namespace monitoring
kubectl apply -n monitoring -f opentelemetry/otelcol.yaml
```

```bash
helm install opentelemetry-operator open-telemetry/opentelemetry-operator \
--set "manager.collectorImage.repository=ghcr.io/open-telemetry/opentelemetry-collector-releases/opentelemetry-collector-k8s" \
--set admissionWebhooks.certManager.enabled=false \
--set admissionWebhooks.autoGenerateCert.enabled=true \
--namespace otel-test-1 \
--create-namespace

helm install opentelemetry-operator open-telemetry/opentelemetry-operator \
--set admissionWebhooks.certManager.enabled=false \
--set admissionWebhooks.autoGenerateCert.enabled=true \
--namespace otel-test-1
-f ./opentelemetry/

helm show values open-telemetry/opentelemetry-operator
helm get values open-telemetry/opentelemetry-operator

helm install opentelemetry-collector open-telemetry/opentelemetry-collector \
   --set image.repository="otel/opentelemetry-collector-k8s" \
   --set mode=daemonset
helm install otelcol-daemonset open-telemetry/opentelemetry-collector -f ./opentelemetry/otelcol-daemonset.helm.chart.yaml
helm upgrade otelcol-daemonset open-telemetry/opentelemetry-collector -f ./opentelemetry/otelcol-daemonset.helm.chart.yaml
helm uninstall opentelemetry-collector
helm install otelcol-deployment open-telemetry/opentelemetry-collector -f ./opentelemetry/otelcol-deployment.helm.chart.yaml

helm install otel-demo open-telemetry/opentelemetry-demo --namespace otel-test-1
helm template opentelemetry-demo open-telemetry/opentelemetry-demo --namespace otel-test-1 > opentelemetry-demo.yaml
kubectl describe serviceaccount otel-collector -n otel-test-1
kubectl delete serviceaccount otel-collector -n otel-test-1
kubectl --namespace otel-test-1 port-forward svc/frontend-proxy 8080:8080
```

```bash
kubectl apply -f ./opentelemetry/otel-configmap.yaml
kubectl apply -f ./opentelemetry/otel-secret.yaml
```

```bash
helm install opentelemetry-operator open-telemetry/opentelemetry-operator \
--namespace opentelemetry-operator-system \
--create-namespace \
--set "manager.collectorImage.repository=ghcr.io/open-telemetry/opentelemetry-collector-releases/opentelemetry-collector-contrib"

kubectl exec -it <pod-name> -- bin/bash
kubectl debug -it <pod-name> -n <namespace> --image=busybox --target=<container-name>
kubectl rollout restart deployment <deployment_name>
```

My Setup
```bash
helm install opentelemetry-operator open-telemetry/opentelemetry-operator \
--set "manager.collectorImage.repository=ghcr.io/open-telemetry/opentelemetry-collector-releases/opentelemetry-collector-contrib" \
--set admissionWebhooks.certManager.enabled=false \
--set admissionWebhooks.autoGenerateCert.enabled=true \
--namespace opentelemetry \
--create-namespace

# Export the values.yaml for configuration
helm show values open-telemetry/opentelemetry-operator > otelopr-values.yaml

kubectl get crd opentelemetrycollectors.opentelemetry.io -o yaml
kubectl explain opentelemetrycollector.spec

helm install otel-demo open-telemetry/opentelemetry-demo \
--namespace otel-demo \
--create-namespace

kubectl --namespace otel-demo port-forward svc/frontend-proxy 8080:8080
```

```bash
kubectl get service streamlit-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
kubectl port-forward service/streamlit-service 8080:80 --address 0.0.0.0
kubectl rollout restart deployment/my-streamlit-app
```

```bash
kubectl exec -it daemonset-otelcol-collector-jmlhq -n opentelemetry -- \
  curl -s http://172.18.0.2:10255/stats/summary | head -20

kubectl get pod my-streamlit-app-84f8fcf556-gvszs -o jsonpath='{.spec.initContainers[*].name}' -n default
kubectl exec -it my-streamlit-app-577b879479-br8zg -- curl http://deployment-otelcol-collector.opentelemetry.svc.cluster.local:4318
kubectl exec -it my-streamlit-app-577b879479-br8zg -- ping deployment-otelcol-collector.opentelemetry.svc.cluster.local

kubectl rollout restart deployment coredns -n kube-system
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl get svc kube-dns -n kube-system
kubectl describe pod -n kube-system -l k8s-app=kube-dns
dig @10.96.0.10 deployment-otelcol-collector.opentelemetry.svc.cluster.local
nslookup deployment-otelcol-collector.opentelemetry.svc.cluster.local 10.244.0.99
```