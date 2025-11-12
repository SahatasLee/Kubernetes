# API Group

<https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.33/#api-overview>

```sh
kubectl api-resources -o wide
```

# Core Groups

```sh
kubectl api-resources --api-group=""

apiVersion: v1
kind: Pod, Service, ConfigMap, Secret
```

Core API Group (/api/v1)
These resources are part of the core group:

Namespaced Resources:

    ConfigMap

    Endpoints

    Event (core version; also available in events.k8s.io)

    PersistentVolumeClaim

    Pod

    PodTemplate

    ReplicationController

    Secret

    Service

Cluster-scoped Resources:

    Namespace

    Node

    PersistentVolume

    ComponentStatus (mostly deprecated in favor of metrics-server)

    ServiceAccount

    LimitRange

    ResourceQuota

# Named Groups

```sh
apiVersion: apps/v1
kind: Deployment, StatefulSet, ReplicaSet
```

| API Group                      | Common Resources                                                        |
| ------------------------------ | ----------------------------------------------------------------------- |
| `apps`                         | Deployments, StatefulSets, DaemonSets, ReplicaSets, ControllerRevisions |
| `batch`                        | Jobs, CronJobs                                                          |
| `autoscaling`                  | HorizontalPodAutoscaler                                                 |
| `rbac.authorization.k8s.io`    | Roles, RoleBindings, ClusterRoles, ClusterRoleBindings                  |
| `policy`                       | PodDisruptionBudget, PodSecurityPolicy (deprecated)                     |
| `networking.k8s.io`            | NetworkPolicy, Ingress, IngressClass                                    |
| `storage.k8s.io`               | StorageClass, VolumeAttachment, CSIDriver, CSINode, CSIStorageCapacity  |
| `coordination.k8s.io`          | Lease                                                                   |
| `node.k8s.io`                  | RuntimeClass                                                            |
| `events.k8s.io`                | Events (newer version of core/v1 Events)                                |
| `authentication.k8s.io`        | TokenReview, SelfSubjectAccessReview                                    |
| `authorization.k8s.io`         | SubjectAccessReview, SelfSubjectRulesReview                             |
| `admissionregistration.k8s.io` | ValidatingWebhookConfiguration, MutatingWebhookConfiguration            |
| `apiextensions.k8s.io`         | CustomResourceDefinition (CRD)                                          |
| `apiregistration.k8s.io`       | APIService (for extending the Kubernetes API with aggregated APIs)      |
| `scheduling.k8s.io`            | PriorityClass                                                           |
| `flowcontrol.apiserver.k8s.io` | FlowSchema, PriorityLevelConfiguration                                  |
