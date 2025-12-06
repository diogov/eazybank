# EazyBank: Microservices Setup and Deployment Guide


### Kubernetes Dashboard Setup

Check the documentation and execute the following commands:
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
```bash
# Add Kubernetes Dashboard Helm repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/

# Deploy a Helm release named "kubernetes-dashboard" using the Kubernetes Dashboard chart
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard \
  --create-namespace --namespace kubernetes-dashboard

# Forward the dashboard service to your local machine
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```

Follow the Kubernetes instructions for creating a sample user:
[Creating a Sample User
](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md).
Create the yaml files following documentation instructions and execute the following commands:

```bash
# Apply the admin user and role binding manifests
kubectl apply -f dashboard-adminuser.yaml
kubectl apply -f dashboard-rolebinding.yaml

# Create a token for the admin user
kubectl -n kubernetes-dashboard create token admin-user

# Apply additional secret if needed
kubectl apply -f secret.yaml

# Retrieve the token value
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath="{.data.token}" | base64 -d
```


### Keycloak Setup

From the `helm` directory, execute:

```bash
helm install keycloak keycloak
```
Access the Keycloak webpage (version 2): http://localhost:7080
Username: `admin`
Password: `admin`

Access the Keycloak webpage (version 3): http://localhost:80
Username: `user`
Password: `password`

Go to **Clients → Create client**:  
-- Client ID: `eazybank-callcenter-cc`  
-- Name: `EazyBank Call Center App`  
Check **Client Authentication** and **Service Account Roles**  
Press **Next** and **Save**  
  
Go to **Realm Roles → Create Role**  
Create roles: `ACCOUNTS`, `CARDS`, `LOANS`  

Go to **Clients → eazybank-callcenter-cc → Service Account Roles → Assign Role → Realm Roles**  
Check roles: `ACCOUNTS`, `CARDS`, `LOANS`


### Code deployment - Version 2
Build and containerize all Spring Boot projects by running:  
`mvn compile jib:dockerBuild`

From the /v2/kubernetes, run:
```bash
kubectl apply -f 1_keycloak.yml
kubectl apply -f 2_configmaps.yaml
kubectl apply -f 3_configserver.yaml
kubectl apply -f 4_eurekaserver.yml
kubectl apply -f 5_accounts.yml
kubectl apply -f 6_loans.yml
kubectl apply -f 7_cards.yml
kubectl apply -f 8_gateway.yml
```

### Code deployment - Version 3

For each Spring Boot project, run:  
`mvn compile jib:dockerBuild`

From the /v3/kubernetes, run:
```bash
kubectl apply -f kubernetes-discoveryserver.yml
```

From /v3/helm run:
```bash
helm install kafka kafka
helm install prometheus kube-prometheus - available at http://localhost:9090/
helm install loki grafana-loki
helm install alloy grafana-alloy
helm install tempo grafana-tempo
helm install grafana grafana -  available at http://localhost:3000/
cd environments
helm install eazybank prod-env
```
