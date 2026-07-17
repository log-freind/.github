# Log Friends MicroK8s Ingress

This folder stores non-secret MicroK8s manifests for the Log Friends deployment.

## Ingress

`06-ingress.yaml` routes one external HTTP entrypoint to the existing services.

```text
/          -> log-friends-console-web:3000
/api       -> log-friends-console:8080
/ingest    -> log-friends-console:8080
/actuator  -> log-friends-console:8080
```

## Apply

```bash
microk8s enable ingress
microk8s kubectl apply -f 01-configmap.example.yaml
microk8s kubectl apply -f 06-ingress.yaml
microk8s kubectl rollout restart deployment/log-friends-console -n log-friends
microk8s kubectl rollout restart deployment/log-friends-console-web -n log-friends
microk8s kubectl get ingress -n log-friends
```

## Check

```bash
curl http://192.168.0.38/
curl http://192.168.0.38/api/log-catalog/apps
curl http://192.168.0.38/actuator/health
```

Secrets and environment-specific values should stay in Kubernetes Secret or local
operations notes, not in this repository.
