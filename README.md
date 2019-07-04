# istio-hipster-setup
This demo uses the HELM TEMPLATE to set up istio and install Istio add ons

Enable GKE API if not enabled
gcloud services enable container.googleapis.com

Create cluster
gcloud container clusters create istio --enable-autoupgrade \
    --enable-autoscaling --min-nodes=3 --max-nodes=10 --machine-type=n1-standard-4 --num-nodes=5 --zone=us-central1-a

Authenticate against cluster
gcloud container clusters get-credentials istio --zone us-central1-a --project williamscj-istio-test

Confirm Cluster is set up and observe no pods runnings under default or istio-system namespace
kubectl get pods -n istio-system 
kubectl get pods -n default

Download latest Istio version
https://github.com/istio/istio/releases/ - select relevant OS tar/zip
Unpackage tar/zip and move to desired location

Install Helm client
https://github.com/helm/helm/blob/master/docs/install.md

In terminal navigate into root folder of Istio which was installed above

Create Istio namespace
kubectl create namespace istio-system

Install Istio CRD’s
helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -

Confirm 23 CRD’s installed
kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l

Install Istio add on’s
helm template ./install/kubernetes/helm/istio --name istio --namespace istio-system \
   --set prometheus.enabled=true \
   --set kiali.enabled=true --set kiali.createDemoSecret=true \
   --set "kiali.dashboard.jaegerURL=http://jaeger-query:16686" \
   --set "kiali.dashboard.grafanaURL=http://grafana:3000" \
   --set grafana.enabled=true \
   --set sidecarInjectorWebhook.enabled=true > istio.yaml

Apply Istio values config generated above
kubectl apply -f istio.yaml

Enable Istio auto sidecar injection 
kubectl label namespace default istio-injection=enabled

Git clone Hipster repo and go to the repository directory
https://github.com/GoogleCloudPlatform/microservices-demo.git

Set up Hipster store
kubectl apply -f ./release/kubernetes-manifests.yaml

Check Hipster pods are initialized
kubectl get pods

Get Hipster istio-ingressgateway IP and confirm you can view Hipster store
kubectl get service/frontend-external

Browser Kiali using port forwarding
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001

Browse 127.0.0.1:20001 and login using admin/admin

Repeat port fording to view Grafana and Prometheus

Grafana
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 8080:3000 &

Prometheus
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 8081:9090 &
