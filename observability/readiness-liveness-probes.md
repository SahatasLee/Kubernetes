# Readiness & Liveness Probes

## Readiness Probe

## Liveness Probe

```sh
livenessProbe:
      httpGet:
        path: /live
        port: 8080
      initialDelaySeconds: 80
      periodSeconds: 1
```