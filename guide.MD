## Install Argo Rollouts
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
