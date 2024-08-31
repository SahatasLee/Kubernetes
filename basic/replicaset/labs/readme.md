# Replicaset Lab

```
How many PODs exist on the system?

In the current(default) namespace.

$ kubectl get po
No resources found in default namespace.

How many ReplicaSets exist on the system?

In the current(default) namespace.

$ kubectl get replicasets.apps 
No resources found in default namespace.

How about now? How many ReplicaSets do you see?

We just made a few changes!

$ kubectl get replicasets.apps 
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       20s

How many PODs are DESIRED in the new-replica-set? 

$ 4

What is the image used to create the pods in the new-replica-set?

$ kubectl describe replicasets.apps 
Name:         new-replica-set
Namespace:    default
Selector:     name=busybox-pod
Labels:       <none>
Annotations:  <none>
Replicas:     4 current / 4 desired
Pods Status:  0 Running / 4 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  name=busybox-pod
  Containers:
   busybox-container:
    Image:      busybox777
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  2m19s  replicaset-controller  Created pod: new-replica-set-btjcv
  Normal  SuccessfulCreate  2m19s  replicaset-controller  Created pod: new-replica-set-jzkg5
  Normal  SuccessfulCreate  2m19s  replicaset-controller  Created pod: new-replica-set-8fn7c
  Normal  SuccessfulCreate  2m19s  replicaset-controller  Created pod: new-replica-set-6kbpd

$ busybox777

How many PODs are READY in the new-replica-set?

$ 0

Why do you think the PODs are not ready?

$ kubectl get po
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-8fn7c   0/1     ImagePullBackOff   0          4m57s
new-replica-set-6kbpd   0/1     ImagePullBackOff   0          4m57s
new-replica-set-jzkg5   0/1     ImagePullBackOff   0          4m57s
new-replica-set-btjcv   0/1     ImagePullBackOff   0          4m57s

$ kubectl describe po new-replica-set-8fn7c 
Name:             new-replica-set-8fn7c
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.23.42.6
Start Time:       Sun, 28 Apr 2024 15:29:18 +0000
Labels:           name=busybox-pod
Annotations:      <none>
Status:           Pending
IP:               10.42.0.11
IPs:
  IP:           10.42.0.11
Controlled By:  ReplicaSet/new-replica-set
Containers:
  busybox-container:
    Container ID:  
    Image:         busybox777
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo Hello Kubernetes! && sleep 3600
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jxmt4 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-jxmt4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  5m24s                  default-scheduler  Successfully assigned default/new-replica-set-8fn7c to controlplane
  Normal   Pulling    3m58s (x4 over 5m23s)  kubelet            Pulling image "busybox777"
  Warning  Failed     3m57s (x4 over 5m22s)  kubelet            Failed to pull image "busybox777": failed to pull and unpack image "docker.io/library/busybox777:latest": failed to resolve reference "docker.io/library/busybox777:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     3m57s (x4 over 5m22s)  kubelet            Error: ErrImagePull
  Warning  Failed     3m32s (x6 over 5m22s)  kubelet            Error: ImagePullBackOff
  Normal   BackOff    15s (x20 over 5m22s)   kubelet            Back-off pulling image "busybox777"

$ The image busybox777 doesn't exist.

$ kubectl delete po new-replica-set-8fn7c 
pod "new-replica-set-8fn7c" deleted

$ kubectl get po
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-6kbpd   0/1     ImagePullBackOff   0          7m4s
new-replica-set-jzkg5   0/1     ImagePullBackOff   0          7m4s
new-replica-set-btjcv   0/1     ImagePullBackOff   0          7m4s
new-replica-set-jglj2   0/1     ErrImagePull       0          14s

Why are there still 4 PODs, even after you deleted one?

$ ReplicaSet ensures that desired number of PODs always run.

Create a ReplicaSet using the replicaset-definition-1.yaml file located at /root/.

There is an issue with the file, so try to fix it.

Run the command: You can check for apiVersion of replicaset by command kubectl api-resources | grep replicaset

kubectl explain replicaset | grep VERSION and correct the apiVersion for ReplicaSet.

Then run the command: kubectl create -f /root/replicaset-definition-1.yaml


$ kubectl api-resources | grep replicaset
replicasets                       rs           apps/v1                           true         ReplicaSet

$ kubectl explain replicaset | grep VERSION
VERSION:    v1

$ kubectl explain replicaset
GROUP:      apps
KIND:       ReplicaSet
VERSION:    v1

DESCRIPTION:
    ReplicaSet ensures that a specified number of pod replicas are running at
    any given time.
    
FIELDS:
  apiVersion    <string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind  <string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata      <ObjectMeta>
    If the Labels of a ReplicaSet are empty, they are defaulted to be the same
    as the Pod(s) that the ReplicaSet manages. Standard object's metadata. More
    info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec  <ReplicaSetSpec>
    Spec defines the specification of the desired behavior of the ReplicaSet.
    More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

  status        <ReplicaSetStatus>
    Status is the most recently observed status of the ReplicaSet. This data may
    be out of date by some window of time. Populated by the system. Read-only.
    More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

Fix the issue in the replicaset-definition-2.yaml file and create a ReplicaSet using it.

This file is located at /root/.

Hint!!!

Line 9 and 13 should match.

Delete the two newly created ReplicaSets - replicaset-1 and replicaset-2

$ kubectl delete replicasets.apps replicaset-1 replicaset-2 
replicaset.apps "replicaset-1" deleted
replicaset.apps "replicaset-2" deleted

Fix the original replica set new-replica-set to use the correct busybox image.

Either delete and recreate the ReplicaSet or Update the existing ReplicaSet and then delete all PODs, so new ones with the correct image will be created.

$ kubectl edit replicasets.apps new-replica-set 
replicaset.apps/new-replica-set edited

$ kubectl get po
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-6kbpd   0/1     ImagePullBackOff   0          23m
new-replica-set-btjcv   0/1     ImagePullBackOff   0          23m
new-replica-set-jzkg5   0/1     ImagePullBackOff   0          23m
new-replica-set-jglj2   0/1     ImagePullBackOff   0          16m

$ kubectl delete po --all
pod "new-replica-set-6kbpd" deleted
pod "new-replica-set-btjcv" deleted
pod "new-replica-set-jzkg5" deleted
pod "new-replica-set-jglj2" deleted

$ kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
new-replica-set-78t58   1/1     Running   0          6s
new-replica-set-q55cm   1/1     Running   0          6s
new-replica-set-58mcc   1/1     Running   0          6s
new-replica-set-zxbjh   1/1     Running   0          6s

Scale the ReplicaSet to 5 PODs.

Use kubectl scale command or edit the replicaset using kubectl edit replicaset.

$ kubectl scale --replicas=5 rs/new-replica-set

>> https://kubernetes.io/docs/reference/kubectl/generated/kubectl_scale/

$ kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
new-replica-set-78t58   1/1     Running   0          3m3s
new-replica-set-q55cm   1/1     Running   0          3m3s
new-replica-set-58mcc   1/1     Running   0          3m3s
new-replica-set-zxbjh   1/1     Running   0          3m3s
new-replica-set-cn9jr   1/1     Running   0          15s


Now scale the ReplicaSet down to 2 PODs.

Use the kubectl scale command or edit the replicaset using kubectl edit replicaset.

$ kubectl edit rs new-replica-set 
replicaset.apps/new-replica-set edited

$ kubectl get po
NAME                    READY   STATUS        RESTARTS   AGE
new-replica-set-58mcc   1/1     Running       0          4m59s
new-replica-set-zxbjh   1/1     Running       0          4m59s
new-replica-set-q55cm   1/1     Terminating   0          4m59s
new-replica-set-78t58   1/1     Terminating   0          4m59s
new-replica-set-cn9jr   1/1     Terminating   0          2m11s
```