## Apply Manifests

```bash
# Create the namespace
kubectl apply -f .infrastructure/namespace.yml

# Create the deployment
kubectl apply -f .infrastructure/deployment.yml

# Create the ClusterIP service
kubectl apply -f .infrastructure/clusterIp.yml

# Create the Horizontal Pod Autoscaler
kubectl apply -f .infrastructure/hpa.yml
```

Check that everything is running:

```bash
kubectl get all -n mateapp
```

## Resource Requests and Limits Explanation

In the deployment.yml, each container includes resource requests and limits:

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

Why These Values:

| Type        | CPU                 | Memory  | Reasoning                                                                                                 |
|-------------|---------------------|---------|-----------------------------------------------------------------------------------------------------------|
| **Request** | `250m` (¼ CPU core) | `64Mi`  | The ToDo app is a lightweight Django application — requires little CPU/memory under normal load.          |
| **Limit**   | `500m` (½ CPU core) | `128Mi` | Allows the container to burst under higher load (e.g., multiple users) without exhausting node resources. |

## Strategy Configuration Explanation

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

Why These Values:

type: RollingUpdate → ensures zero downtime during updates.
maxUnavailable: 1 → at most one pod can be down during deployment.
maxSurge: 1 → allows one extra pod above the desired replicas during the update.

## Horizontal Pod Autoscaler (HPA) Explanation

```yaml
minReplicas: 2
maxReplicas: 5
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```

Why These Values:

minReplicas: 2 → Ensures high availability (app still works even if one pod fails).
maxReplicas: 5 → Allows the app to scale up to 5 pods during heavy traffic.
averageUtilization: 70% → When average CPU or memory usage exceeds 70%, Kubernetes automatically adds pods.

This setup maintains performance under load and cost efficiency when idle.

## Accessing the Application using Port Forward

You can access the application locally from your machine by forwarding a local port to the service.

1) Run port-forward command:

```bash
kubectl port-forward svc/todoapp-service 8081:80 -n mateapp
```

2) Test from your local terminal or browser:

Using curl:

```bash
curl localhost:8081
```

Using a browser: Navigate to http://localhost:8081

## Cleanup

```bash
kubectl delete -f .infrastructure/hpa.yml
kubectl delete -f .infrastructure/deployment.yml
kubectl delete -f .infrastructure/clusterIp.yml
kubectl delete -f .infrastructure/namespace.yml
```