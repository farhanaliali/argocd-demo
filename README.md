
# ArgoCD Installation with SSL and Ingress

## 1. Install ArgoCD

    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

This will create a new namespace, argocd, where Argo CD services and application resources will live.



## Access The Argo CD API Server

By default, the Argo CD  server is not exposed with an external IP. To access the Argocd server choose one of the following techniques to expose the Argo CD  server:

### 1. Service Type NodePort or Load Balancer

Change the argocd-server service type to NodePort:

    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
OR

    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

### 2. Set Up SSL and Ingress

Prepare your SSL certificate (tls.crt) and private key (tls.key) files, and create the secret:

        kubectl create secret tls argocd-tls \
         --cert=/path/to/tls.crt \
         --key=/path/to/tls.key \
         --namespace argocd

Create an Ingress Resource

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: argocd-server-ingress
    namespace: argocd
    annotations:
        nginx.ingress.kubernetes.io/ssl-passthrough: "true"
        nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    spec:
        ingressClassName: nginx
        rules:
        - host: argocd.example.com
          http:
           paths:
            - path: /
           pathType: Prefix
           backend:
              service:
                name: argocd-server
                port:
                 name: https
        tls:
         - hosts:
            - argocd.example.com 
           secretName: argocd-tls # as expected by argocd-server

Apply the Ingress resource

## 3. Access ArgoCD Dashboard

To access ArgoCD over HTTPS:

* Update DNS: Ensure that argocd.example.com points to your Kubernetes Ingress controller's external IP.

* Login to ArgoCD: The default username is admin. To retrieve the initial password, run:

    
    ```bash
    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode

Use the retrieved password to log in to the ArgoCD UI at https://argocd.example.com