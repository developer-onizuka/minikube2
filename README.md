# 1. Run minikube and specify the number of nodes to spin up.
```
PS C:\Users\developer> minikube.exe start --nodes=2
* Microsoft Windows 10 Pro 10.0.19042 Build 19042 上の minikube v1.18.1
* virtualboxドライバーが自動的に選択されました
* コントロールプレーンのノード minikube を minikube 上で起動しています
* virtualbox VM (CPUs=2, Memory=2200MB, Disk=20000MB) を作成しています...
* Docker 20.10.3 で Kubernetes v1.20.2 を準備しています...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring CNI (Container Networking Interface) ...
* Kubernetes コンポーネントを検証しています...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v4
* 有効なアドオン: storage-provisioner, default-storageclass

* Starting node minikube-m02 in cluster minikube
* virtualbox VM (CPUs=2, Memory=2200MB, Disk=20000MB) を作成しています...
* ネットワーク オプションが見つかりました
  - NO_PROXY=192.168.99.106
  - no_proxy=192.168.99.106
* Docker 20.10.3 で Kubernetes v1.20.2 を準備しています...
  - env NO_PROXY=192.168.99.106
* Kubernetes コンポーネントを検証しています...
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

# 2. Check the nodes which run in the kubernets cluster.
```
PS C:\Users\developer> minikube.exe node list
minikube        192.168.99.106
minikube-m02    192.168.99.107
```

# 3. Put labels on each node.
```
PS D:\k8s\deployment> kubectl.exe label nodes minikube location=tokyo
node/minikube labeled
PS D:\k8s\deployment> kubectl.exe label nodes minikube-m02 location=telaviv
node/minikube-m02 labeled

PS D:\k8s\deployment> kubectl.exe describe node minikube | Select-String "location"

                    location=tokyo

PS D:\k8s\deployment> kubectl.exe describe node minikube-m02 | Select-String "location"

                    location=telaviv

```

# 4. Edit yaml file for Nginx and create Deployment.
```
PS D:\k8s\deployment> cat .\ngingx_1.14.2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
spec:
  selector:
    matchLabels:
      run: nginx-test
  replicas: 2
  template:
    metadata:
      labels:
        run: nginx-test
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

PS D:\k8s\deployment> kubectl.exe create -f .\ngingx_1.14.2.yaml
deployment.apps/nginx-test created
```

# 5. Check if the pods running and which nodes the pods are running at. 
```
PS D:\k8s\deployment> kubectl.exe describe node minikube | Select-String "nginx"
PS D:\k8s\deployment> kubectl.exe describe node minikube-m02 | Select-String "nginx"

  default                     nginx-test-767864879b-mbfvv    0 (0%)        0 (0%)      0 (0%)           0 (0%)           2m33s
  default                     nginx-test-767864879b-w5qsg    0 (0%)        0 (0%)      0 (0%)           0 (0%)           2m33s
```

# 6. Create a new yaml file Nginx will run at minikube-m02 and apply it.
```
PS D:\k8s\deployment> cat .\ngingx_1.14.2_nodeSelector.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
spec:
  selector:
    matchLabels:
      run: nginx-test
  replicas: 2
  template:
    metadata:
      labels:
        run: nginx-test
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
      nodeSelector:
        location: tokyo

PS D:\k8s\deployment> kubectl.exe apply -f .\ngingx_1.14.2_nodeSelector.yaml
```

# 7. Check again if the pods running and which nodes the pods are running at. 
```
PS D:\k8s\deployment> kubectl.exe describe node minikube | Select-String "nginx"

  default                     nginx-test-7876d8684d-j8cr2         0 (0%)        0 (0%)      0 (0%)           0 (0%)         27s
  default                     nginx-test-7876d8684d-vl4l7         0 (0%)        0 (0%)      0 (0%)           0 (0%)         25s

PS D:\k8s\deployment> kubectl.exe describe node minikube-m02 | Select-String "nginx"
```
