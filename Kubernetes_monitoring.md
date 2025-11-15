#### Kubernetes Monitoring on Minikube Cluster Running on WSL

- **Install Minikube:**  
  Follow the [Minikube Install Guide](https://minikube.sigs.k8s.io/docs/start/) and ensure Docker/VirtualBox is available.  

  ```bash
  minikube start
  ```

- **Install Helm (on your local OS or within WSL):**  

  ```bash
  curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
  ```

- **kubectl installed and configured with your Minikube cluster:**  

  ```bash
  kubectl config use-context minikube
  ```

#### 1. Start Minikube

```bash
minikube start
```

#### 2. Add & Update Helm Repositories

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

#### 3. Create a Monitoring Namespace

```bash
kubectl create namespace monitoring
```

#### 4. Install `kube-prometheus-stack and Grafana` via Helm

```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
helm install grafana grafana/grafana
```

- Uses default configuration out-of-the-box; customizations can be added in future (`-f <values.yaml>`).

#### 5. Access Prometheus, Grafana, Alertmanager UIs

**Prometheus:**  
Non-WSL - Can expose port as nodeport and access  

```bash
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --port=9090 --name=prometheus-server-ext
```

- Obtain NodePort by running `kubectl get svc`.  
- Access via  `http://<minikube-ip>:<nodeport>`.  

For WSL, port-forward instead:  

```bash
kubectl port-forward service/monitoring-kube-prometheus-stack-prometheus -n monitoring 9090:9090
```

Visit: [http://localhost:9090](http://localhost:9090)

**Grafana:**  

- Similar to Prometheus, you can expose as NodePort if not using WSL.  
- In case of WSL, use port forwarding:  

```bash
kubectl port-forward service/monitoring-grafana -n monitoring 3000:80
```

Visit: [http://localhost:3000](http://localhost:3000)  

- Default user: `admin`  
- Default password: retrieve with:  

```bash
kubectl get secret --namespace monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

**Alertmanager:**  

```bash
kubectl port-forward service/monitoring-kube-prometheus-stack-alertmanager -n monitoring 9093:9093
```

Visit: [http://localhost:9093](http://localhost:9093)

#### 6. Clean Up

**Remove the stack:**  

```bash
helm uninstall monitoring -n monitoring
```

**Delete the namespace:**  

```bash
kubectl delete namespace monitoring
```
