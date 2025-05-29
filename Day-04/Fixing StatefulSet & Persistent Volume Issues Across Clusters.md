# 🚨 Fixing StatefulSet & Persistent Volume Issues Across Clusters

## 📌 Introduction

Stateful applications in Kubernetes bring their own set of complexities—especially when dealing with persistent storage across different cloud platforms. In this article, we’ll walk through a **real-world DevOps scenario** where a StatefulSet works flawlessly on AWS EKS but fails on AKS, GKE, and Minikube. You’ll learn how to troubleshoot this issue like a pro.

## 🔍 Problem Statement

A developer reported an issue through a Jira ticket:  
> “A StatefulSet application works on AWS EKS but fails on AKS, GKE, and Minikube.”

As a **DevOps Engineer**, you’re responsible for identifying and resolving Kubernetes issues. Let’s break this down and fix it step-by-step using a sample `StatefulSet` YAML file deploying an NGINX application.

---

## 🛠️ Deep Dive: Understanding the Root Cause

### What We Observed:
- `kubectl get pods`: Only one pod in `Pending` status
- `kubectl describe pod <pod>`:  
  > `Warning: Pod has unbound immediate PersistentVolumeClaims`

### Why It Happens:
- The **StatefulSet uses a PVC with a specific StorageClass (e.g., EBS)** which exists only on AWS EKS.
- Other environments like Minikube do not have this StorageClass available.

### 🚦 Unique StatefulSet Behavior:
- Replicas are created **sequentially**.
- If `pod-0` fails, `pod-1` and `pod-2` will not be scheduled.
  
---

## 📂 Workflow Behind the Scenes

Here's how the components interact:

```text
StatefulSet ➝ PersistentVolumeClaim ➝ StorageClass ➝ Provisioner ➝ PersistentVolume
````

### AWS EKS Example:

* PVC requests a `StorageClass: ebs`
* `ebs.csi.aws.com` provisioner dynamically creates the volume

### Minikube/GKE/AKS:

* Only `standard` StorageClass is available by default
* Using `ebs` will result in unbound PVCs because the provisioner doesn't exist

---

## 🧪 Hands-On Fix: Minikube Example

### ✅ Step-by-Step

1. **Original YAML uses EBS (Fails on Minikube)**:

   ```yaml
   volumeClaimTemplates:
     - metadata:
         name: data
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: ebs
         resources:
           requests:
             storage: 1Gi
   ```

2. **Fix It**: Change `storageClassName` to `standard`

   ```yaml
   storageClassName: standard
   ```

3. **Delete Old PVCs**:

   ```bash
   kubectl delete pvc --all
   ```

4. **Reapply Updated StatefulSet**:

   ```bash
   kubectl apply -f sample-statefulset.yaml
   ```

5. **Watch the Pods Come Up**:

   ```bash
   kubectl get pods -w
   ```

---

## 💡 Practical Insights

### Why Replicas Come Up Sequentially in StatefulSets?

In databases like MySQL or MongoDB:

* `pod-0` might be a **primary node**
* `pod-1`, `pod-2` are **secondary nodes**
* Starting them sequentially ensures **data integrity** and correct cluster behavior

### Troubleshooting Tip:

If pods remain pending, always run:

```bash
kubectl describe pod <pod-name>
```

Look for `UnboundImmediatePersistentVolumeClaims`.

---

## 🔌 Bonus: CSI Drivers for Enterprise Storage

What if you want to use **NetApp**, **Pure Storage**, or other third-party storage solutions?

✅ **Use CSI (Container Storage Interface) drivers**

### Example Flow:

1. Install the CSI driver provided by the vendor (usually via Helm or Operator)
2. Create `StorageClass` referencing the CSI driver
3. Use this class in your PVC

### More on CSI:

* CSI abstracts storage provisioning
* Works across cloud-native and on-prem environments
* [Kubernetes CSI Docs](https://kubernetes.io/docs/concepts/storage/volumes/#csi)

---

## 📚 Best Practices for DevOps Engineers

✅ Always check available `StorageClass` on target clusters:

```bash
kubectl get storageclass
```

✅ Define default `StorageClass` per cluster environment

✅ Use `helm` or `kustomize` to template and dynamically apply environment-specific configs

✅ For production DBs, validate CSI drivers are **certified and supported**

---

## 🚀 Conclusion

StatefulSets are powerful but require **careful orchestration of storage resources**. Cross-cloud deployments will fail if the `StorageClass` and provisioners are not aligned with the environment. As a DevOps Engineer, mastering PVCs, StorageClasses, and CSI drivers enables you to deliver **highly available, cloud-agnostic deployments**.

> ✅ Lesson: Always adapt your storage config to match the underlying Kubernetes infrastructure.

---

## 🙌 Contribute

If you found this useful:

* 🌟 Star the repo
* 🍴 Fork and improve it
* 🛠️ Share your troubleshooting examples

---

> © Bharath Kumar Reddy | [GitHub](https://github.com/BharathKumarReddy2103) | [LinkedIn](https://www.linkedin.com/in/bharath-kumar-reddy2103)
