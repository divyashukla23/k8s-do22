You are tasked with deploying and managing a simple web application on Kubernetes using Minikube.
This exercise will help you understand Kubernetes Objects like Services, StatefulSets, and Horizontal Pod Autoscalers (HPA).
Tasks
Deployment + Service (Basic Access)
Create a Deployment with 2 replicas of an nginx container.
Expose this Deployment using a NodePort Service so it can be accessed from your host machine.
Verify access by running:
   curl http://<minikube-ip>:<nodeport>
StatefulSet (Sticky Identity + Persistence)
Create a StatefulSet with 2 replicas of nginx.
Each Pod should have its own PersistentVolumeClaim (PVC).
Inside each Pod, create a custom index.html (e.g., Hello from web-0 and Hello from web-1).
Verify that when Pods are deleted and recreated, their data persists.

Horizontal Pod Autoscaler (Scaling Out)
Deploy a simple workload (e.g., php-apache or a stress container).
Create an HPA for this Deployment with:
Minimum: 1 replica
Maximum: 5 replicas
Target CPU utilization: 50%
Generate load using a BusyBox Pod:

  kubectl run -it loadgen --rm --image=busybox:1.28 --restart=Never -- \  /bin/sh -c "while true; do wget -q -O- http://php-apache; done"
Observe how the HPA automatically scales the replicas up.
Deliverables
YAML manifests for:
Deployment + Service
StatefulSet + PVC
HPA
Proof of execution:
Curl output from the NodePort Service.
StatefulSet Pods (web-0, web-1) showing persisted data.
HPA scaling replicas under load (kubectl get hpa -w).