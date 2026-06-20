# rabbitmq-kubernetes

Kubernetes manifests for running RabbitMQ as an internal message broker for SUNBOYS services.

## What It Deploys

- Namespace `infra`
- Secret with RabbitMQ credentials
- PersistentVolumeClaim for broker data
- StatefulSet with RabbitMQ management image
- ClusterIP Service for AMQP
- ClusterIP Service for the management UI

## Ready-To-Use Targets

| Target | Command | AMQP DNS | Management DNS |
| --- | --- | --- | --- |
| Shared dev broker | `kubectl apply -k k8s/overlays/dev` | `rabbitmq.infra.svc.cluster.local:5672` | `rabbitmq-management.infra.svc.cluster.local:15672` |
| Auth service broker | `kubectl apply -k k8s/overlays/auth-service` | `auth-rabbitmq.infra.svc.cluster.local:5672` | `auth-rabbitmq-management.infra.svc.cluster.local:15672` |
| CI/test broker | `kubectl apply -k k8s/overlays/ci` | `ci-rabbitmq.infra.svc.cluster.local:5672` | `ci-rabbitmq-management.infra.svc.cluster.local:15672` |

## Deploy

```bash
kubectl apply -k k8s/overlays/dev
kubectl -n infra rollout status statefulset/rabbitmq
```

For a prefixed target:

```bash
kubectl apply -k k8s/overlays/auth-service
kubectl -n infra rollout status statefulset/auth-rabbitmq
```

## Connection

Shared dev AMQP URL:

```text
amqp://sunboys_app:sunboys_app_password@rabbitmq.infra.svc.cluster.local:5672/
```

Auth service AMQP URL:

```text
amqp://auth_service:change_me_auth@auth-rabbitmq.infra.svc.cluster.local:5672/
```

## Management UI

RabbitMQ management is kept internal as a ClusterIP service.

Port-forward when needed:

```bash
kubectl -n infra port-forward svc/rabbitmq-management 15672:15672
```

Then open:

```text
http://localhost:15672
```

## Create A New Target

Copy the template overlay:

```bash
cp -r k8s/overlays/template k8s/overlays/my-purpose
mv k8s/overlays/my-purpose/secret.env.example k8s/overlays/my-purpose/secret.env
```

Edit:

- `namePrefix` in `k8s/overlays/my-purpose/kustomization.yaml`
- `app.kubernetes.io/instance` in `k8s/overlays/my-purpose/kustomization.yaml`
- `RABBITMQ_DEFAULT_USER` and `RABBITMQ_DEFAULT_PASS`
- PVC storage size patch if the target needs more or less disk

Then deploy:

```bash
kubectl apply -k k8s/overlays/my-purpose
```

## Production Notes

The checked-in overlays contain development credentials. Before using this in production, replace passwords and avoid committing real secrets. Prefer Sealed Secrets, External Secrets Operator, SOPS, or your cloud secret manager.
