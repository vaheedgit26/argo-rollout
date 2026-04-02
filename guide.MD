## 🚀 🔷 FULL ARCHITECTURE (END-TO-END)

Perfect — let’s build a production-grade, end-to-end GitOps system with:  
- Canary deployments   
- Auto rollback  
- GitHub Actions CI  
- GitOps repo updates  
- Sync back to Git after rollback 

This is exactly how top companies design safe delivery systems 🔥   

```text
Developer push
   ↓
GitHub Actions (CI)
   ↓
Build + Push Image (ECR)
   ↓
Update GitOps repo (dev)
   ↓
ArgoCD sync
   ↓
Argo Rollouts (canary)
   ↓
Prometheus checks metrics
   ↓
✅ Success → promote
❌ Failure → auto rollback
   ↓
Sync rollback → Git
```

### Install Argo Rollouts
```bash
kubectl create namespace argo-rollouts

kubectl apply -n argo-rollouts \
  -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

```text
Q: How do you handle automatic rollback in production?

Answer:

In production, I use Argo Rollouts with canary strategy and integrate it with Prometheus metrics. If the new release violates SLOs like error rate or latency, the rollout is automatically aborted and traffic is shifted back to the stable version without manual intervention.

🔥 That’s top 1% DevOps answer
```
## 🔷 🥇 1. ARGO ROLLOUTS (FULL YAML)   
👉 Replace Deployment with Rollout   
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 3

  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: { duration: 60 }

        - analysis:
            templates:
              - templateName: error-rate-check

        - setWeight: 50
        - pause: { duration: 60 }

        - analysis:
            templates:
              - templateName: error-rate-check

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: 180789647333.dkr.ecr.ap-south-1.amazonaws.com/my-app:sha-abc123
          ports:
            - containerPort: 8080
```

🔷 🔬 2. PROMETHEUS ANALYSIS TEMPLATE   
👉 This is the brain of auto rollback  
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: error-rate-check
spec:
  metrics:
    - name: error-rate
      interval: 30s
      successCondition: result < 0.05
      failureCondition: result >= 0.05

      provider:
        prometheus:
          address: http://prometheus.monitoring.svc.cluster.local:9090
          query: |
            sum(rate(http_requests_total{status=~"5.."}[1m]))
            /
            sum(rate(http_requests_total[1m]))
```
🔥 What happens:
```text
If error rate >= 5% → rollout FAILS → AUTO ROLLBACK
```
