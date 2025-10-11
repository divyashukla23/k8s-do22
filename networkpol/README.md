## 🧠 What is a NetworkPolicy?

A **NetworkPolicy** defines how Pods are allowed to **communicate with each other and other network endpoints** (ingress & egress).

By default, in Kubernetes:

> All Pods can talk to all other Pods (in all namespaces).

When you apply a NetworkPolicy, you start **restricting** that traffic.

---

## ⚙️ Step 1: Start Minikube and Enable Networking Addon

```bash
minikube start
minikube addons enable ingress
```

> ✅ Important: Minikube must use a CNI (Container Network Interface) plugin that supports network policies.
> The default “bridge” network doesn’t enforce policies.

You can check:

```bash
minikube addons enable cilium
```

or

```bash
minikube start --cni=calico
```

Either **Calico** or **Cilium** supports NetworkPolicy enforcement.

---

## 🧩 Step 2: Create a Namespace for the Demo

```bash
kubectl create ns netpol-demo
```

---

## 🧩 Step 3: Deploy Two Simple Apps (Frontend & Backend)

### frontend.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  namespace: netpol-demo
  labels:
    app: frontend
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
```

### backend.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  namespace: netpol-demo
  labels:
    app: backend
spec:
  containers:
    - name: http
      image: hashicorp/http-echo
      args:
        - "-text=Hello from backend"
      ports:
        - containerPort: 5678
```

Apply them:

```bash
kubectl apply -f frontend.yaml -f backend.yaml
```

Check:

```bash
kubectl get pods -n netpol-demo
```

---

## 🧪 Step 4: Verify That They Can Talk (Before Policy)

Get the backend Pod IP:

```bash
kubectl get pod backend -n netpol-demo -o wide
```

Exec into the frontend Pod and try to connect:

```bash
kubectl exec -it frontend -n netpol-demo -- curl <backend-IP>:5678
```

✅ You should see:

```
Hello from backend
```

That’s because no NetworkPolicy restricts traffic yet.

---

## 🧱 Step 5: Create a NetworkPolicy to Restrict Access

Let’s allow only Pods **with label `app: frontend`** to talk to the backend.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-frontend
  namespace: netpol-demo
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
```

Apply it:

```bash
kubectl apply -f backend-netpol.yaml
```

---

## 🧪 Step 6: Test Again

### ✅ Frontend Pod should still work:

```bash
kubectl exec -it frontend -n netpol-demo -- curl <backend-IP>:5678
```

Output:

```
Hello from backend
```

### 🚫 Any other Pod should now be blocked:

Create a “test” Pod:

```bash
kubectl run testpod --image=busybox -n netpol-demo -- sleep 3600
kubectl exec -it testpod -n netpol-demo -- wget -qO- <backend-IP>:5678
```

You’ll see:

```
wget: download timed out
```

That means your NetworkPolicy is working 🎯

---

## 🧩 Step 7: (Optional) Allow External Access Too

To allow connections from outside the cluster or a specific CIDR:

```yaml
ingress:
  - from:
      - podSelector:
          matchLabels:
            app: frontend
      - ipBlock:
          cidr: 192.168.49.0/24   # adjust based on your minikube subnet
```

You can get your Minikube subnet via:

```bash
minikube ssh "ip addr show docker0"
```

---

## 🧾 Summary

| Step | Action                  | Command                                          |
| ---- | ----------------------- | ------------------------------------------------ |
| 1    | Start Minikube with CNI | `minikube start --cni=calico`                    |
| 2    | Create namespace        | `kubectl create ns netpol-demo`                  |
| 3    | Deploy Pods             | `kubectl apply -f frontend.yaml -f backend.yaml` |
| 4    | Test connectivity       | `curl <backend-IP>:5678`                         |
| 5    | Apply NetworkPolicy     | `kubectl apply -f backend-netpol.yaml`           |
| 6    | Verify restrictions     | test with different Pods                         |

---

