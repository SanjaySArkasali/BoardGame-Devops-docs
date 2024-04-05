# K8 Master-Worker configuration

to use Kubernetes in the ec2 at least we need 4 GB ram and 2 CPU's that is t2 medium ot higher.


## Installation

Kubernetes pre-install configuration (common).

```bash
sudo apt update

sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock

sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
```
kubernetes components installation (common)

```bash 
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
```
---
## Create the Kubernetes cluster

As output will get a join command which we will use on the worker nodes to form the Cluster, save this so in future we can expand cluster
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

ex: (run this on worker nodes to join then to master)
```bash
kubeadm join 172.31.95.1:6443 --token 2yyyu9.wmfnlo9alfs7tlzw \
--discovery-token-ca-cert-hash sha256:9b760e42c3d7735c6faba274234d3384f76c8468d70d1a254cf3e738fcbf1baf
```
--- 

## Configure Kubernetes Cluster on master node

the kubernetes expects the config file in this directory which we will copy from kubernetes installation this contains the kubernetes coniguration which is nessesary for kubectl and ensure the user has all the nessesary permissions

```bash	
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Deploying Networking solution (calico)
	
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
	
Deploy Ingress controller
	
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```
	
Check the nodes using
```bash
kubectl get nodes
```


---

	
## RBAC - Role Based Access Control
Create a namespace 
```bash
kubectl create ns <namespace>
```
Creating service account
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: <namespace>
```
then run the file 
```bash
kubectl apply -f <file>.yaml
```
Create Role
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: <namespace>
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```
Bind this role to service account
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: <namespace> 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role 
subjects:
- namespace: <namespace> 
  kind: ServiceAccount
  name: jenkins 
```
Create a secret for Jenkins to access the cluster
```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: myserviceaccount

```
[Additional token resource](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#:~:text=To%20create%20a%20non%2Dexpiring,with%20that%20generated%20token%20data.)

then run the command with the namespace
```bash
kubectl apply -f <sec.yaml> -n <namespace> 
```
run this command to get the secret
```bash
kubectl describe secret mysecretname -n <namespace>
```
add this to credentials in Jenkins as secret text




## Setting up Kubernetes deployment with Jenkins 

create roles as given above and then add get the token and add in credentials then 
   - go to pipeline syntax withKubeConfig
   - then choose the credentials
   - go to master to get the end point in ```cd ~/.kube``` -> ```cat config ```get the server end point

like this:
```yaml
        stage('Deploy To Kubernetes') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {
                        sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }
```
this is a example template for deployment-service.yaml
```yaml
apiVersion: apps/v1
kind: Deployment # Kubernetes resource kind we are creating
metadata:
  name: <app-name>-deployment
spec:
  selector:
    matchLabels:
      app: <app-name>
  replicas: 2 # Number of replicas that will be created for this deployment
  template:
    metadata:
      labels:
        app: <app-name>
    spec:
      containers:
        - name: <app-name>
          image: adijaiswal/<app-name>:latest # Image that will be used to containers in the cluster
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080 # The port that the container is running on in the cluster


---

apiVersion: v1 # Kubernetes API version
kind: Service # Kubernetes resource kind we are creating
metadata: # Metadata of the resource kind we are creating
  name: <app-name>-ssvc
spec:
  selector:
    app: <app-name>
  ports:
    - protocol: "TCP"
      port: 8080 # The port that the service is running on in the cluster
      targetPort: 8080 # The port exposed by the service
  type: LoadBalancer # type of the service.
```

