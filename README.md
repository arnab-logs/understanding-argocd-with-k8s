![Kubernetes](https://img.shields.io/badge/Kubernetes-326ce5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-FFCB2B?style=for-the-badge&logo=kubernetes&logoColor=black)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EC1943?style=for-the-badge&logo=argocd&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

---

# Getting Started with ArgoCD on Minikube

This repository is my personal exploration to **understand and experiment with ArgoCD** on a local Minikube cluster. Here, I explore its core components, deploy a sample application, and experience GitOps workflows in practice.

---

##   Prerequisites

Before starting, make sure you have:

* **Minikube** installed ([installation guide](https://minikube.sigs.k8s.io/docs/start/))
* **kubectl** installed and configured ([installation guide](https://kubernetes.io/docs/tasks/tools/))
* Basic understanding of Kubernetes concepts like **pods**, **services**, and **deployments**

---

## 1️⃣ Setting Up the Minikube Cluster

First, we need a Kubernetes cluster. Minikube makes this easy for local testing.

```bash
minikube start
```

You can check the cluster status:

```bash
minikube status
```

Once the cluster is up, we can move on to installing ArgoCD.

---

## 2️⃣ Installing ArgoCD

ArgoCD provides [detailed documentation](https://argo-cd.readthedocs.io/en/stable/). We'll follow the **Getting Started** guide.

The easiest way to install ArgoCD is using **plain manifests**:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### What this does:

* Creates a **namespace** called `argocd`.

  > It’s recommended to deploy controllers in their own namespace to keep resources organized.
* Installs all ArgoCD components, including:

  * **Custom Resource Definitions (CRDs)**
  * **Service accounts, roles, role bindings**
  * Core controllers and supporting services

Check the pods:

```bash
kubectl get pods -n argocd
```

<img width="2934" height="418" alt="image" src="https://github.com/user-attachments/assets/e12d7249-ed25-4d80-8a34-5277faef9d46" />

You should see multiple pods coming up. Let’s understand them.

---

## 3️⃣ Understanding ArgoCD Components

| Component                      | Purpose                                                                                                                                                              |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **argocd-server**              | The API server that users interact with for **UI and CLI configurations**. This is where you log in, create apps, and manage ArgoCD.                                 |
| **repo-server**                | Communicates with your **Git-based VCS** (GitHub, Bitbucket, etc.) to fetch manifests. ArgoCD works only with Git-based repositories.                                |
| **redis**                      | Stores state and caches information for ArgoCD components.                                                                                                           |
| **notification-controller**    | Introduced in **v2.5**, it handles notifications for application events (e.g., deployment success/failure, sync status). Useful for Slack, email, or webhook alerts. |
| **argocd-dex-server**          | Handles **Single Sign-On (SSO)** via OIDC or OAuth. If you integrate with Google, GitHub, or LDAP, Dex manages authentication.                                       |
| **application-set-controller** | Allows dynamically creating and managing multiple applications based on Git repos or cluster resources. Useful when you have many apps with similar configurations.  |
| **application-controller**     | Core of ArgoCD. Continuously reconciles the **desired state in Git** with the **actual state in your Kubernetes cluster** and reports any discrepancies.             |

---

## 4️⃣ Accessing ArgoCD UI

To interact with ArgoCD via a browser, we need to expose the **argocd-server** service. First, check the services:

```bash
kubectl get svc -n argocd
```

<img width="2934" height="462" alt="image" src="https://github.com/user-attachments/assets/ffebc63e-c305-4f02-847b-0a74221a5a12" />


### Why we need a service

* Pods run the ArgoCD components, but to access the **UI externally**, we need a **service** to route traffic.
* By default, `argocd-server` uses **ClusterIP** (internal to the cluster). We’ll use NodePort for external access.

<img width="2934" height="1156" alt="image" src="https://github.com/user-attachments/assets/8302301c-2e30-41aa-ba23-a3757ca27d11" />

### Accessing via Minikube

```bash
minikube service list -n argocd
```

<img width="2934" height="652" alt="image" src="https://github.com/user-attachments/assets/21f6af86-67dc-4937-8292-c8eb8a0b8426" />

```bash
minikube service argocd-server -n argocd
```

<img width="2934" height="780" alt="image" src="https://github.com/user-attachments/assets/365e3f88-fa8c-41c3-a5f3-956a490c8095" />

* Minikube creates a tunnel and provides an external URL (HTTP and HTTPS).
* Open the URL in your browser to access the ArgoCD UI.

<img width="2934" height="1728" alt="image" src="https://github.com/user-attachments/assets/4afd2635-87d2-4887-991a-3a7694936cb0" />

---

## 5️⃣ Logging in to ArgoCD

Since we haven’t configured SSO, we’ll log in with **admin credentials**:

* **Username:** `admin`
* **Password:** Stored in a Kubernetes secret:

 You can describe the secret and manually decode the base64 value:

```bash
kubectl get secret -n argocd argo-cd-admin-secret -o yaml
echo <base64-encoded-password> | base64 --decode
```

<img width="624" height="704" alt="image" src="https://github.com/user-attachments/assets/b939df35-e544-4dce-81dc-f7a675b75609" />

This will take you to the ArgoCD UI.

<img width="2926" height="1172" alt="image" src="https://github.com/user-attachments/assets/012d5ffc-7711-42d4-b6d1-c071debb5655" />

---

## 6️⃣ Deploying a Sample Application

We’ll use a repository maintained by ArgoCD: [argocd-example-apps](https://github.com/argoproj/argocd-example-apps).

### Steps:

1. Click **Create Application** in the UI.
2. Fill in the details:

   * **Application Name:** `my-first-time`
   * **Project:** `default`
   * **Sync Policy:** `Automatic` (deploys app immediately)
   * **Repository URL:** `https://github.com/argoproj/argocd-example-apps`
   * **Path:** `guestbook`
   * **Cluster URL:** default
   * **Namespace:** default

<img width="2926" height="1092" alt="image" src="https://github.com/user-attachments/assets/c9de58f4-8bcb-4bc5-ac1e-834948031299" />

<img width="2926" height="1312" alt="image" src="https://github.com/user-attachments/assets/e6f8e3cd-6912-4ac7-b784-a26ea1784faf" />

3. Click **Create**.

ArgoCD will fetch the **deployment.yaml** and **service.yaml** from the GitHub repo and deploy them to your cluster.

<img width="2928" height="1560" alt="image" src="https://github.com/user-attachments/assets/2f1aa397-c4d8-468f-b1a0-19280ad6da05" />

<img width="2928" height="1506" alt="image" src="https://github.com/user-attachments/assets/39e35d21-f28b-426e-bd80-d6248da21a7a" />

Verify in terminal:

```bash
kubectl get deploy
```

<img width="2928" height="154" alt="image" src="https://github.com/user-attachments/assets/6c6c6614-5507-4c7f-aa97-92d608151308" />

You’ll see the deployment created by ArgoCD. The UI will also show the application status.

<img width="2928" height="1506" alt="image" src="https://github.com/user-attachments/assets/6b42da8c-aea7-441c-ad5e-717c6245c844" />

<img width="2928" height="598" alt="image" src="https://github.com/user-attachments/assets/8e23d213-5d5a-4e84-a26f-cb8f5b0d09fd" />

---

## 7️⃣ Continuous Deployment with ArgoCD

The magic of ArgoCD is **continuous reconciliation**:

1. Make a change to the GitHub repo (e.g., update `deployment.yaml`).
2. ArgoCD automatically detects the change.
3. The new state is applied to your Kubernetes cluster.

> ArgoCD ensures your cluster **always matches the desired state in Git**.
> You can also use **Helm charts** instead of raw manifests if needed.

---

## Key Takeaways

* ArgoCD enables **GitOps-based deployment** for Kubernetes.
* Core components handle reconciliation, Git integration, caching, and optional SSO.
* Services and NodePort/Minikube tunnels allow UI access.
* Continuous deployment is automatic once configured.
* Supports flexible folder structures and deployment methods, including Helm.

---

## What I Learned

* How to set up a local Kubernetes cluster using Minikube.
* How to install ArgoCD using manifests and understand its components.
* The role of argocd-server, repo-server, application-controller, redis, notification-controller, and dex-server.
* How to access the ArgoCD UI via NodePort or Minikube tunnels.
* How ArgoCD implements GitOps workflows, continuously reconciling Git repository state with the cluster.
* How to deploy applications from GitHub, monitor their status, and automatically apply updates.
* Practical experience with continuous deployment principles and managing Kubernetes resources declaratively.

---

## References

* [ArgoCD Official Docs](https://argo-cd.readthedocs.io/en/stable/)
* [ArgoCD Example Apps](https://github.com/argoproj/argocd-example-apps)
* [Minikube Documentation](https://minikube.sigs.k8s.io/docs/start/)

---
