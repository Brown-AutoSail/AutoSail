# Sockshop-istio
Sockshop demo with Istio service mesh



## Installation/Setup on GKE                                                                                                                                 

1. #### Create GKE Cluster

   1. Create a new GKE cluster with at least 4 standard nodes.

      NOTE: Do NOT check the Istio checkbox. GKE managed Istio is out-of-date .

   2. After the cluster is ready, find "CONNECT" in the cluster detail. Run the supplied command in Cloud Shell to connect the K8s cluster. 

      The command should look something like this:

      ```bash
      gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project autosail-330818
      ```

2. #### Install Istio

   1. Download Istio

      ```bash
      curl -L https://istio.io/downloadIstio | sh -
      cd istio-1.12.0
      export PATH=$PWD/bin:$PATH
      ```

   2. Install Istio

      ```bash
      istioctl install -y
      ```


3. #### Deploy Sockshop

   1. Clone AutoSail repo

      ```bash
      cd ..
      git clone https://github.com/jeremykong-ur/AutoSail.git
      cd AutoSail
      ```

   2. Create namespace

      ```bash
      kubectl create namespace sock-shop
      ```

   3. Set namespace to inject Istio (Enovy) sidecar by default

      ```bash
      kubectl label namespace sock-shop istio-injection=enabled
      ```

   4. Deploy Sockshop Application

      ```bash
      kubectl apply -f Application/Sockshop/1-sock-shop-complete-demo-istio.yaml
      ```

   5. Associate this application with the Istio gateway (open to outside traffic)

      ```bash
      kubectl apply -f Application/Sockshop/2-sockshop-gateway.yaml
      ```

   6. Apply simple Istio Destination Rules

      (We will need to add more rules here to tune)

      ```bash
      kubectl apply -f Application/Sockshop/3-destination-rules.yaml
      ```

   7. After all workloads are in a stable state, you can verify external access

      Get GKE Ingress host and port:

      ```bash
      export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
      export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
      export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
      ```

      Get`GATEWAY_URL`, (i.e. our application address)

      ```bash
      export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
      echo "$GATEWAY_URL"
      ```

4. (Optional) Install telemetry applications.

   ```
   cd ../istio-1.12.0
   kubectl apply -f samples/addons
   kubectl rollout status deployment/kiali -n istio-system
   ```



## Generating Load

We use sockshop's provided scripts to generate traffic:

```bash
docker run --net=host weaveworksdemos/load-test -h $GATEWAY_URL -r 100 -c 2
```

```
Options:
  -d  Delay before starting
  -h  Target host url, e.g. localhost:80
  -c  Number of clients (default 2)
  -r  Number of requests (default 10)
```



## Telemetry

### 1. Prometheus
```bash
istioctl dashboard prometheus
```
### Grafana
```bash
istioctl dashboard grafana
```
### Jaeger
```bash
istioctl dashboard jaeger
```

### Kiali
1. Connect to Kiali.
```bash
istioctl dashboard kiali
```
