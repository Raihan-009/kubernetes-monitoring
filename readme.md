# Setting up kube-state-metrics and Prometheus Integration Guide

## Prerequisites
- A running Kubernetes cluster
- Access to both Kubernetes and Prometheus servers
- kubectl installed and configured
- Git installed

## Part 1: Installing kube-state-metrics

### 1.1. Log in to the Kubernetes Server
SSH or connect to your Kubernetes server with appropriate credentials.

### 1.2. Clone and Install kube-state-metrics
```bash
# Clone the repository
git clone https://github.com/kubernetes/kube-state-metrics.git

# Navigate to the directory
cd kube-state-metrics/

# Checkout specific version
git checkout v1.8.0

# Apply the Kubernetes manifests
kubectl apply -f kubernetes
```

### 1.3. Create NodePort Service
Navigate back to your home directory and create a new service definition:
```bash
cd ..
```

Create a new file named `kube-state-metrics-nodeport-svc.yml`:
```yaml
kind: Service
apiVersion: v1
metadata:
  namespace: kube-system
  name: kube-state-nodeport
spec:
  selector:
    k8s-app: kube-state-metrics
  ports:
  - protocol: TCP
    port: 8080
    nodePort: 30000
  type: NodePort
```

### 1.4. Apply and Test the Service
```bash
# Apply the service configuration
kubectl apply -f kube-state-metrics-nodeport-svc.yml

# Test the metrics endpoint
curl localhost:30000/metrics
```

## Part 2: Configuring Prometheus

### 2.1. Log in to the Prometheus Server
SSH or connect to your Prometheus server with appropriate credentials.

### 2.2. Edit Prometheus Configuration
Edit the Prometheus configuration file:
```bash
sudo vi /etc/prometheus/prometheus.yml
```

Add the following configuration under the `scrape_configs` section:
```yaml
  - job_name: 'Kubernetes'
    static_configs:
      - targets: ['10.0.0.10:30000']  # Replace 10.0.0.10 with your Kubernetes node's private IP
```

### 2.3. Restart Prometheus
```bash
sudo systemctl restart prometheus
```

### 2.4. Verify Configuration
1. Open your web browser and navigate to:
   ```
   http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090
   ```

2. Navigate to the Expression Browser tab

3. Test the configuration by running this query:
   ```
   kube_pod_status_ready{namespace="default",condition="true"}
   ```

## Troubleshooting

### Common Issues and Solutions

1. **Cannot connect to metrics endpoint:**
   - Verify the NodePort service is running:
     ```bash
     kubectl get svc -n kube-system kube-state-nodeport
     ```
   - Check if the pod is running:
     ```bash
     kubectl get pods -n kube-system | grep kube-state-metrics
     ```

2. **Prometheus cannot scrape metrics:**
   - Verify network connectivity between Prometheus and Kubernetes node
   - Check Prometheus logs:
     ```bash
     sudo journalctl -u prometheus
     ```
   - Verify the private IP address is correct in prometheus.yml

3. **No metrics showing in Prometheus:**
   - Check Prometheus targets page for errors (Status > Targets)
   - Verify the job is properly configured in prometheus.yml
   - Ensure firewall rules allow traffic on port 30000

## Maintenance

### Updating kube-state-metrics
To update to a newer version:
```bash
cd kube-state-metrics
git fetch
git checkout <new-version-tag>
kubectl apply -f kubernetes
```

### Backing Up Configuration
Always backup your configuration files before making changes:
```bash
sudo cp /etc/prometheus/prometheus.yml /etc/prometheus/prometheus.yml.backup
```
