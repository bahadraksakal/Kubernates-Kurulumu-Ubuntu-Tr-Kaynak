# Kubernates MultiNode Setup Ubuntu TR-EN Documentation

****

- [EN : Description :book: :leftwards_arrow_with_hook:](#en)  
- [TR : Açıklama :book: :leftwards_arrow_with_hook:](#tr)

****

#### [EN]

# Kubernates Install On Ubuntu EN Documentation

Ubuntu v18.04 || Repo containing Kubernates installation with 3 nodes (controller-worker1-worker2) on 22.04 versions and the installation of a sample project in the system. The number of nodes can be increased depending on the situation, but there must be at least 2 nodes (controller-worker1).
You can follow some of the steps below in this video: https://www.youtube.com/watch?v=6_i1hXXviHw

### Preparer:
*[Bahadır Aksakal](https://github.com/bahadraksakal)*

## Requirements
Before starting the installation:

- Ubuntu v18.04 || 3 servers using 22.04 versions.
- Servers must have at least 2 GB Ram and 2 CPUs.
- All servers must be connected to the internet.
- You must have the authority to gain root privileges on all servers with ``sudo -s``.

## Installation Steps

If you receive an error during the installation steps, you can review the 'Common Errors and Solutions' section at the bottom of the page.
Follow all the steps one by one, solutions to possible errors you may encounter and additional notes are added at the bottom of the page.

Open Ubuntu terminal and run the following commands.

Required packages for different package installations:
```
sudo apt-get update
sudo apt install apt-transport-https curl -y
```

## Containerd Setup
Establishing the container structure:

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable " | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io -y
```

## Setting the Config Files of the Containerd Structure:
Creating the containerd configuration file:

```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

## Editing the Config File Named `/etc/containerd/config.toml` in the Containerd Structure.
We open the config file named `/etc/containerd/config.toml` in the containerd structure with nano via the terminal.
Then we find the place where it says 'SystemdCgroup'. We replace the 'false' expression there with 'true'.

```
sudo nano /etc/containerd/config.toml
```
Set SystemdCgroup to true: `SystemdCgroup = true // this is not a command, do not paste it into the terminal and run it!!!`
<br />With the 'ctrl+s' combination, we open it with nano and save the config file we edited.
<br />With the 'ctrl+x' combination, we close the config file we opened and edited with nano.

Restart the Containerd structure for the edits we made to take effect.
```
sudo systemctl restart containerd
```
Don't worry if the terminal does not give any output, some codes do not give any output when run.

## Kubernetes Installation
To install Kubernetes, follow the codes below:
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt install kubeadm kubelet kubectl kubernetes-cni
```

## Turning off swap and configuring network settings
Run the following code to turn off swap and configure network settings:

```
sudo swapoff -a
```

  If there is a swap-related statement in the /etc/fstab file, disabling this statement with nano:
```
sudo nano /etc/fstab
```
In the opened file, there is a line like this: `/swapfile none swap sw`. Make the code `#/swapfile none swap sw` non-functional by placing `#` at the beginning of the line.
With the `ctrl+s` combination, we open the /etc/fstab file we edited with nano and save it.
With the `ctrl+x` combination, we close the /etc/fstab file that we opened and edited with nano.

Enabling the kernel module named br_netfilter:
```
sudo modprobe br_netfilter
```
A kernel module named "br_netfilter" is loaded. This module is a module that provides packet filtering and connection monitoring functions in bridged network configuration (bridge) in Linux.
*The `sudo modprobe br_netfilter` command loses its functionality if the machine is rebooted, etc... If you get an error and want to install from scratch: Run the `sudo modprobe br_netfilter` command again.*

Configuring some settings with sysctl:
```
sudo sysctl -w net.ipv4.ip_forward=1
```
*`sudo sysctl -w net.ipv4.ip_forward=1` command loses its functionality if the machine is rebooted, etc... In case you get an error and want to install from scratch:
Run the `sudo sysctl -w net.ipv4.ip_forward=1` command again.*

## Cluster Startup (Only Run on Your Master|Controller Node!!!)
To stand up the cluster, simply execute the following code on your Master node:
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
*In the following steps, we will install an application called flannel that performs automatic network configuration for Kubernates. In order for this application to work, never make any changes to the address `10.244.0.0/16`!!!*

Creating a `.kube` directory in your `home` directory:
```
mkdir -p $HOME/.kube
```
*this step is important to use `kubectl` without any problems!*

Copying the Kubernetes configuration file to your home directory:
```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
*this step is important to use `kubectl` without any problems!*

Change the ownership of the file:
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
*this step is important to ensure that `kubectl` can be used without any problems and that you do not receive authorization errors!*

## Flannel Installation (Only Run on Your Master|Controller Node!!!)
Run the following codes to install Flannel:
```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
```
*What does Flannel do? Flannel is a networking solution used to provide network connectivity on Kubernetes. Flannel creates a network in the Kubernetes environment where each container has a unique IP address and these containers can communicate with each other. Containers' IP addresses are allocated using a dedicated subnet block for each.*
*A very important system for the Multi-Node structure.*

## Verifying that the Installation is Working Correctly
Apply the codes below, similar outputs from a correct installation are shared below the codes.
```
kubectl get pods --all-namespaces
```
If your setup is correct, you will get a longer output similar to the following:
<br />`
NAMESPACE NAME READY STATUS RESTARTS AGE
default grafana-deployment-nautilus-6864b8bfb9-zcpkl 1/1 Running 0 20m
`

## Create Token (Only Run in Your Master|Controller Node!!!)
Run the following command to create a token
```
kubeadm token create --print-join-command
```
After the code runs, copy the entire token it gives.
## Join with Nodes (Workers)
Paste the token you copied in the step above into the terminal of the worker nodes and run it.
You can use the following command to see whether the operations performed in this step are successful or not:
```
kubectl get nodes
```

## Common Errors and Solutions
### There were some deficiencies in the kubeadm init process and if you want to run the `sudo kubeadm init --pod-network-cidr=10.244.0.0/16` process again, you get an error that some config files are present in the system. Run the codes to solve:
```
sudo swapoff -a
```
```
sudo rm /etc/kubernetes/manifests/kube-apiserver.yaml
sudo rm /etc/kubernetes/manifests/kube-controller-manager.yaml
sudo rm /etc/kubernetes/manifests/kube-scheduler.yaml
sudo rm /etc/kubernetes/manifests/etcd.yaml
```
```
sudo kubeadm reset
```
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### While trying to join with the worker node `[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz" If you get an error like: dial tcp 127.0.0.1:10248: connect: connection refused.` or if there is a problem in the worker node and you are trying to join again, you will get an error that some config files are present. To solve these errors, run the following codes:
```
swapoff -a
```
```
sudo rm -f /etc/kubernetes/kubelet.conf
sudo rm -f /etc/kubernetes/bootstrap-kubelet.conf
sudo rm -f /etc/kubernetes/pki/ca.crt
```
```
sudo systemctl restart kubelet
```
'Join again from the worker node terminal with the token you created on the master'

### If you receive errors such as `port 10250 is in use` or `time out` when trying to join or accessing kubectl in the worker node terminal, run the following commands:
```
sudo systemctl stop kubelet
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/kubelet
```
'Join again from the worker node terminal with the token you created on the master'

### If you get some errors while your Master(Controller) node is accessing kubectl (usually after reboot), run the following codes:
```
mv $HOME/.kube $HOME/.kube.bak
mkdir $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Deploying a Project to the Kubernate System.
*It would be better to perform all operations on the Master (Controller) node*

You can follow the steps below to run a Node.js project on Kubernetes:

### 1. Creating Dockerfile:
In order to convert the Node.js project on the Worker node into a Docker image, you need to create a Dockerfile. Dockerfile specifies the requirements and configuration of the project. Example Dockerfile:

```
# Build base of Docker image
FROM node:14

# Create the application directory
WORKDIR /app

# Copy and install dependencies
COPY package.json .
COPY package-lock.json .
RUN npm install

# Copy the application source code
COPY . .

# Start the application
CMD ["npm", "start"]
```

### 2. Creating the Docker image:
Create the Docker image of your project using Dockerfile. You can upload this image to the local Docker image repository via Docker Daemon. Open a terminal in the same directory as the Dockerfile and adapt the sample code below:

```
docker build -t my-node-app:1.0 .
```

### 3. Installing the Docker image into a Container Registry:
You can install the Docker image into a Container Registry. This step ensures that the image is accessible to all nodes of your Kubernetes cluster. For example, you can use a Container Registry like Docker Hub. To upload the image to Docker Hub, you can follow these steps:

    - Log in to Docker Hub or create an account.
    - Use the `docker login` command to connect to your Docker Hub account via terminal.
    - Use `docker push` command to upload the Docker image to Docker Hub. For example:

    ```
    docker push my-dockerhub-username/my-node-app:1.0
    ```

### 4. Creating Kubernetes Pod and Deployment YAML file:
To run your project on Kubernetes, you need to create a Pod and Deployment YAML file. These files define the creation of a Pod and Deployment resource using the Docker image of your project. For example:

Pod YAML File (node-app-pod.yaml):
```
apiVersion: v1
kind: Pod
metadata:
   name: node-app-pod
spec:
   containers:
     - name: node-app
       image: my-dockerhub-username/my-node-app:1.0
       ports:
         - containerPort: 3000
```

Deployment YAML File (node-app-deployment.yaml):
```
apiVersion: apps/v1
kind: Deployment
metadata:
   name: node-app-deployment
spec:
   replicas: 1
   selector:
     matchLabels:
       app: node-app
   template:
     metadata:
       labels:
         app: node-app
     spec:
       containers:
         - name: node-app
           image: my-dockerhub-username/my-node-app:1.0
           ports:
             - containerPort: 3000
```

### 5.Uploading the project on Kubernetes:
  Apply resources to your Kubernetes cluster using the Pod and Deployment YAML files you created. This will launch your project as a Pod and Deployment. You can use the following commands to implement YAML files:

```
kubectl apply -f node-app-pod.yaml
kubectl apply -f node-app-deployment.yaml
```

By following these steps, you can run your Node.js project in a Kubernetes cluster. You can deploy the project on a separate Worker node or across all nodes.

### 6. Getting the deployed project up and running via the service:

Required file to start the backend (backend-service.yaml):
```
apiVersion: v1
kind: service
metadata:
   name: backend-service
spec:
   selector:
     app: node-app
   ports:
     - protocol: TCP
       port: 3000
       targetPort: 3000
   type: NodePort
```

Run the following command to create the Service using the YAML file:
```
kubectl apply -f backend-service.yaml
```

Once the Service is created, you can use the following command to view the Service details:
```
kubectl get services
```
You can find the running service ips with the command and connect to that service with the relevant port.

You can access your project via the browser with the relevant IP and port:
```
http://<ip>:<port>
```

## Kubernetes Dashboard Setup
Kubernastes dashboard is used to track and manage your workers. It provides you with an easier-to-use visual interface for operations you can perform via the command line.

### Installing the Kubernetes dashboard:
Code that installs Kubernetes Dashboard on kubernetes cluster:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

### Configuring the Kubernetes dashboard:

```
nano admin-user-service-account.yaml
```
Copy the following into the opened file.
```
apiVersion: v1
kind: ServiceAccount
metadata:
   name: admin-user
   namespace: kubernetes-dashboard
```
With the combination of `ctrl+s`, we open the `admin-user-service-account.yaml` file that we have edited with nano and save it.
With the `ctrl+x` combination, we close the `admin-user-service-account.yaml` file that we opened and edited with nano.

```
nano admin-user-cluster-role-binding.yaml
```
Copy the following into the opened file.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
   name: admin-user
roleRef:
   apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: cluster-admin
subjects:
- kind: ServiceAccount
   name: admin-user
   namespace: kubernetes-dashboard
```
With the `ctrl+s` combination, we save the `admin-user-cluster-role-binding.yaml` file that we opened and edited with nano.
With the `ctrl+x` combination, we close the `admin-user-cluster-role-binding.yaml` file that we opened and edited with nano.

We have prepared the necessary files to create a user above, now we apply them to our cluster.
```
kubectl apply -f admin-user-service-account.yaml -f admin-user-cluster-role-binding.yaml
```
After this process, you will get the following output from the console:
`//Example OUTPUT:
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created`

### Creating Tokens for Kubernetes dashboard:
We need to create a token to connect to the Kubernetes dashboard. Run the following code to create the token:
```
kubectl -n kubernetes-dashboard create token admin-user
```
`
//Example OUTPUT:
eyJhbGciOiJSUzI1Ni849urjwndjsncsh892943ıjfwks
`

### Rebooting the Kubernetes dashboard:
Run the following code to publish and access the Kubernetes dashboard locally:,
`Note: If you have run kubectl proxy, definitely do not run the alternative code below.`
```
kubectl proxy
```

In this step, as an alternative to the code above, you can also apply the following code: (Note: Tests were not included.)
`Note: if you ran the code above, definitely do not run the kubectl proxy command.`
```
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:443
```

To access the standing Kubernetes dashboard, go to the following address from a web browser:
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

All transactions have been completed.

****
****


#### [TR]

# Kubernates Kurulumu Ubuntu Tr Dokümantasyon

Ubuntu v18.04 || 22.04 sürümleri üzerinde 3 node ile (controller-worker1-worker2) kubernates kurulumu ve bir adet örnek projenin sistemde ayağa kaldırılmasını içeren repo. Duruma göre node sayısı arttırılabilir fakat en az 2 node (controller-worker1) bulunmalıdır.
Aşağıdaki adımların bir kısmını şu videodan takip edebilirsiniz: https://www.youtube.com/watch?v=6_i1hXXviHw

### Hazırlayan:
*[Bahadır Aksakal](https://github.com/bahadraksakal)*

## Gereksinimler
kuruluma başlamadan önce:

- Ubuntu v18.04 || 22.04 sürümlerini kullanan 3 adet server. 
- Serverler en az 2 GB Ram ve 2 CPU ya sahip olmalıdır.
- Tüm serverler internete bağlı olmalıdır.
- Tüm serverler de ```sudo -s``` ile root yetkisine sahip olabilme yetkiniz bulunmaladır.

## Kurulum Adımları

Kurulum adımlarında hata alırsanız, sayfanın altında bulunan `Sık Karşılaşılan Hatalar Ve Çözümleri` başlığını inceleyebilirsiniz.
Tüm adımları teker teker uygulayın, alabileceğini olası hataların çözümleri ve ek notlar sayfanın en alt kısmına eklenmiştir.

Ubuntu terminalini açın ve aşağıdaki komutları çalıştırın.

Farklı paket kurumlumları için gerekli paketler:
```
sudo apt-get update
sudo apt install apt-transport-https curl -y
```

## Containerd Kurulumu
Containerd yapısının kurulması:

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io -y
```

## Containerd Yapısının Config Dosyalarının Ayarlanması:
Containerd yapılandırma dosyasının oluşturulması:

```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

##  Containerd Yapısındaki `/etc/containerd/config.toml` Adlı Config Dosyasının Düzenlenmesi.
Containerd yapısındaki `/etc/containerd/config.toml` adlı config dosyasını terminal üzerinden nano ile açıyoruz.
Daha sonra `SystemdCgroup` yazan yeri buluyoruz. Oradaki `false` ifadesini `true` ile değiştiriyoruz.

```
sudo nano /etc/containerd/config.toml
```
Set SystemdCgroup to true: `SystemdCgroup = true // bu bir komut değildir terminale yapıştırıp çalıştırmayın !!!`
<br />`ctrl+s` kombinasyonuyla, nano ile açıp düzenlediğimiz config dosyasını kayıt ediyoruz.
<br />`ctrl+x` kombinasyonuyla, nano ile açıp düzenlediğimiz config dosyasını kapatıyoruz.

Yaptığımız düzenlemelerin etkili olabilemesi için Containerd yapısını tekrardan başlatın.
```
sudo systemctl restart containerd
```
Terminal herhangi bir çıktı vermez ise endişlenmeyin, bazı kodlar çalıştırıldıklarında çıktı vermiyor.

## Kubernetes Kurulumu
Kubernetes kurulumu için, aşağıdaki kodları takip edin:
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt install kubeadm kubelet kubectl kubernetes-cni
```

## swap 'ın Kapatılması ve Ağ Ayalarının Yapılandırılması
swap'ı kapamak ve ağ ayarlarını yapılandırmak için aşağıdaki kodu çalıştırın:

```
sudo swapoff -a
```

 /etc/fstab dosyasında, swap ile ilgili bir ifade varsa bu ifadenin nano ile işlevsiz hale getirilmesi:
```
sudo nano /etc/fstab
```
Açılan dosyada bu şekilde `/swapfile    none    swap    sw` bir satır bulumaktadır. Satırın başına `#` koyarak kodu bu şekilde `#/swapfile    none    swap    sw` işlevsiz hale getirin.
`ctrl+s` kombinasyonuyla, nano ile açıp düzenlediğimiz /etc/fstab dosyasını kayıt ediyoruz.
`ctrl+x` kombinasyonuyla, nano ile açıp düzenlediğimiz /etc/fstab dosyasını kapatıyoruz.

br_netfilter isimli kernel modülünü etkinleştirme:
```
sudo modprobe br_netfilter
```
"br_netfilter" adlı bir kernel modülü yüklenir. Bu modül, linux'ta köprülenmiş ağ yapılandırmasında (bridge) paket filtreleme ve bağlantı izleme işlevlerini sağlayan bir modüldür.
*`sudo modprobe br_netfilter` komutu, makine yeniden başlatılırsa vb... işlevini kaybeder. Bir hata almanız durumunda, işin içinden çıkamayıp sıfırdan kurulum yapmak isterseniz: `sudo modprobe br_netfilter` komutunu tekrardan çalıştırın.*

sysctl ile bazı ayarların yapılandırılması:
```
sudo sysctl -w net.ipv4.ip_forward=1
```
*`sudo sysctl -w net.ipv4.ip_forward=1` komutu, makine yeniden başlatılırsa vb... işlevini kaybeder.  Bir hata almanız durumunda, işin içinden çıkamayıp sıfırdan kurulum yapmak isterseniz:
`sudo sysctl -w net.ipv4.ip_forward=1` komutunu tekrardan çalıştırın.*

## Cluster Ayağa Kaldırma (Sadece Master|Controller Nodunuzda Çalıştırın!!!)
Cluster ayağa kaldırmak için aşağıdaki kodu yalnızca Master nodunuzda uygulayın:
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
*İlerki adımlarda Kubernates için otomatik ağ yapılandırması yapan flannel adında bir uygulama kuracağız. Bu uygulamanın çalışabilmesi için kesinlikle `10.244.0.0/16` adresinde değişiklik yapmayın !!!*

`home` dizininizde bir `.kube` dizini oluşturulması:
```
mkdir -p $HOME/.kube
```
*bu adım `kubectl`'in sorunsuz kullanılabilmesi için önemlidir!*

Kubernetes yapılandırma dosyasını ana dizininize kopyalayalanması:
```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
*bu adım `kubectl`'in sorunsuz kullanılabilmesi için önemlidir!*

Dosyanın sahipliğini değiştirin:
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
*bu adım `kubectl`'in sorunsuz kullanılabilmesi ve yetki hatası almamanız için önemlidir!*

## Flannel Kurulumu (Sadece Master|Controller Nodunuzda Çalıştırın!!!)
Flannel kurulumu için aşağıdaki kodları çalıştırın:
```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
```
*Flannel ne işe yarar: Flannel, Kubernetes üzerinde ağ bağlantısını sağlamak için kullanılan bir ağ çözümüdür. Flannel, Kubernetes ortamında her bir konteynerin benzersiz bir IP adresine sahip olduğu ve bu konteynerlerin birbirleriyle iletişim kurabildiği bir ağ oluşturur. Konteynerlerin IP adresleri, her birine ayrılmış bir alt ağ bloğu kullanılarak tahsis edilir.*
*Multi-Node yapısı için çok önemli bir sistem.*

## Kurulumun Doğru Çalıştığının Doğrulanması
Aşağıdaki kodları uygulayın, doğru bir kurulumda gelen çıktıların benzeleri kodların altında paylaşılmıştır.
```
kubectl get pods --all-namespaces
```
Eğer kurulumunuz doğru ise aşağıdakine benzer, daha uzun bir çıktı alıcaksınız:
<br />`
NAMESPACE              NAME                                           READY   STATUS    RESTARTS        AGE
default                grafana-deployment-nautilus-6864b8bfb9-zcpkl   1/1     Running   0               20m
`

## Token Yaratın  (Sadece Master|Controller Nodunuzda Çalıştırın!!!)
Token yaratmak için aşağıdaki komutu çalıştırın
```
kubeadm token create --print-join-command
```
Kod çalıştıktan sonra verdiği tokenin tamamını kopyalayın.
## Node'lar(Worker) ile Join Olun
Yukarıdaki adımda kopyaladığınız tokeni, `worker node`'ların  terminaline yapıştırın ve çalıştırın.
Bu adımda yapılan işlemlerinin başarılı olup olmadığını görmek için aşağıdaki komutu kullanabilirsiniz:
```
kubectl get nodes
```

## Sık Karşılaşılan Hatalar Ve Çözümleri
### kubeadm init işleminde bazı eksiklikler oldu ve tekrardan `sudo kubeadm init --pod-network-cidr=10.244.0.0/16` işlemini çalıştırmak isterseniz, bazı config dosyalarının sistemde bulunduğuna dair bir hata alırsanız. Çözümü için kodları çalıştırın:
```
sudo swapoff -a
```
```
sudo rm /etc/kubernetes/manifests/kube-apiserver.yaml
sudo rm /etc/kubernetes/manifests/kube-controller-manager.yaml
sudo rm /etc/kubernetes/manifests/kube-scheduler.yaml
sudo rm /etc/kubernetes/manifests/etcd.yaml
```
```
sudo kubeadm reset
```
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### Worker node ile join olmaya çalışırken `[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.` şeklinde bir hata alırsanız veya worker node'da bir sorun varsa tekrardan join olmaya çalışıyorsanız bazı config dosyalarının bulunuduğuna dair hata alırsınız, bu hataların çözümü için aşağıdaki aşağıdaki kodları çalıştırın:
```
swapoff -a
```
```
sudo rm -f /etc/kubernetes/kubelet.conf
sudo rm -f /etc/kubernetes/bootstrap-kubelet.conf
sudo rm -f /etc/kubernetes/pki/ca.crt
```
```
sudo systemctl restart kubelet
```
`Master üzerinde yaratmış olduğunuz token ile worker node terminalinden tekrardan join olun`

### Worker node terminalinde join olmaya çalışırken veya kubectl'e erişirken `port 10250 is use` veya `time out` tarzı hatalar alıyorsanız aşağıdaki komutları çalıştırın:
```
sudo systemctl stop kubelet
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/kubelet
```
`Master üzerinde yaratmış olduğunuz token ile worker node terminalinden tekrardan join olun`

### Master(Controller) node'unuz kubectl'e erişirken bazı hatalar alırsanız(genelde reboot sonrası olur), aşağıdaki kodları çalıştırın:
```
mv  $HOME/.kube $HOME/.kube.bak
mkdir $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Kubernate Sistemine Bir Projenin Deploy Edilmesi.
*Tüm işlemleri Master(Controller) node üzerinde yapmanız daha sağlıklı olur*

Bir Node.js projesini Kubernetes üzerinde çalıştırmak için aşağıdaki adımları takip edebilirsiniz:

### 1. Dockerfile Oluşturma:
Worker node üzerindeki Node.js projesini bir Docker imajına dönüştürebilmek için bir Dockerfile oluşturmanız gerekmektedir. Dockerfile, projenin gereksinimlerini ve yapılandırmasını belirtir. Örnek Dockerfile:

```
# Docker imajının temelini oluşturun
FROM node:14

# Uygulama dizinini oluşturun
WORKDIR /app

# Bağımlılıkları kopyalayın ve kurun
COPY package.json .
COPY package-lock.json .
RUN npm install

# Uygulama kaynak kodunu kopyalayın
COPY . .

# Uygulamayı başlatın
CMD ["npm", "start"]
```

### 2. Docker imajının oluşturulması: 
Dockerfile'ı kullanarak projenizin Docker imajını oluşturun. Bu imajı Docker Daemon aracılığıyla yerel Docker imaj deposuna (local Docker image repository) yükleyebilirsiniz. Dockerfile ile aynı dizinde bir terminal açın ve aşağıdaki örnek kodu kendinize göre uyarlayın:

```
docker build -t my-node-app:1.0 .
```

### 3. Docker imajının bir Container Registry'ye yüklenmesi:
Docker imajını bir Container Registry'ye yükleyebilirsiniz. Bu adım, imajın Kubernetes cluster'ınızın tüm düğümleri tarafından erişilebilir olmasını sağlar. Örneğin, Docker Hub gibi bir Container Registry kullanabilirsiniz. İmajı Docker Hub'a yüklemek için şu adımları izleyebilirsiniz:

   - Docker Hub'a oturum açın veya bir hesap oluşturun.
   - Terminal üzerinden Docker Hub hesabınıza bağlamak için `docker login` komutunu kullanın.
   - Docker imajını Docker Hub'a yüklemek için `docker push` komutunu kullanın. Örneğin:

   ```
   docker push my-dockerhub-username/my-node-app:1.0
   ```

### 4. Kubernetes Pod ve Deployment YAML dosyası oluşturma:
Projenizi Kubernetes üzerinde çalıştırmak için bir Pod ve Deployment YAML dosyası oluşturmanız gerekmektedir. Bu dosyalar, projenizin Docker imajını kullanarak bir Pod ve Deployment kaynağının oluşturulmasını tanımlar. Örneğin:

Pod YAML Dosyası (node-app-pod.yaml):
```
apiVersion: v1
kind: Pod
metadata:
  name: node-app-pod
spec:
  containers:
    - name: node-app
      image: my-dockerhub-username/my-node-app:1.0
      ports:
        - containerPort: 3000
```

Deployment YAML Dosyası (node-app-deployment.yaml):
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
        - name: node-app
          image: my-dockerhub-username/my-node-app:1.0
          ports:
            - containerPort: 3000
```

### 5.Kubernetes üzerinde projenin yüklenmesi:
 Oluşturduğunuz Pod ve Deployment YAML dosyalarını kullanarak Kubernetes cluster'ınıza kaynakları uygulayın. Bu, projenizin Pod ve Deployment olarak başlatılmasını sağlayacaktır. YAML dosyalarını uygulamak için aşağıdaki komutları kullanabilirsiniz:

```
kubectl apply -f node-app-pod.yaml
kubectl apply -f node-app-deployment.yaml
```

Bu adımları takip ederek, Node.js projenizi bir Kubernetes cluster'ında çalıştırabilirsiniz. Projeyi ayrı bir Worker node üzerinde veya tüm düğümlerde dağıtabilirsiniz.

### 6. Deploy edilen projenin servis üzerinden ayağa kaldırılması:

Backend'i ayağa kaldırmak için gerekli dosya(backend-service.yaml):
```
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: node-app
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  type: NodePort
```

YAML dosyasını kullanarak Hizmeti oluşturmak için aşağıdaki komutu çalıştırın:
```
kubectl apply -f backend-service.yaml
```

Hizmet oluşturulduktan sonra, Hizmet ayrıntılarını görüntülemek için aşağıdaki komutu kullanabilirsiniz:
```
kubectl get services
```
komutu ile çalışan servis iplerini bulabilir ve ilgili port ile o servise bağlanabilirsiniz.

Tarayıcı üzerinden ilgili ip ve port ile projenize erişebilirsiniz:
```
http://<ip>:<port>
```

## Kubernetes Dashboard Kurulumu
Kubernastes dashboard workerlerinizi takip etmeye ve yönetmeye yarar. Komut satırını üzerinden yapabileceğiniz işlemler için size kullanımı daha kolay bir görsel arayüz sağlar.

### Kubernetes dashboard'ın Kurulması:
Kubernetes Dashboard'u kubernetes cluster'a kuran kod:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

### Kubernetes dashboard'ın Konfigrasyonunun yapılması:

```
nano admin-user-service-account.yaml
```
Açılan dosyanın içine aşağıdakileri kopyala.
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```
`ctrl+s` kombinasyonuyla, nano ile açıp düzenlediğimiz `admin-user-service-account.yaml` dosyasını kayıt ediyoruz.
`ctrl+x` kombinasyonuyla, nano ile açıp düzenlediğimiz `admin-user-service-account.yaml` dosyasını kapatıyoruz.

```
nano admin-user-cluster-role-binding.yaml
```
Açılan dosyanın içine aşağıdakileri kopyala.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
`ctrl+s` kombinasyonuyla, nano ile açıp düzenlediğimiz `admin-user-cluster-role-binding.yaml` dosyasını kayıt ediyoruz.
`ctrl+x` kombinasyonuyla, nano ile açıp düzenlediğimiz `admin-user-cluster-role-binding.yaml` dosyasını kapatıyoruz.

Yukarıda bir kullanıcı oluşturmak için gerekli dosyaları hazırladık, şimdi bunları cluster'ımıza uyguluyoruz.
```
kubectl apply -f admin-user-service-account.yaml -f admin-user-cluster-role-binding.yaml
```
Bu işlemden sonra konsoldan şu şekilde bir çıktı alıcaksınız:
`//Örnek OUTPUT:
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created`

### Kubernetes dashboard'a Token Oluşturma:
Kubernetes dashboard'a bağlanabilmek için token oluşturmamız lazım. Token oluşturmak için aşağıdaki kodu çalıştırın:
```
kubectl -n kubernetes-dashboard create token admin-user
```
`
//Örnek OUTPUT:
eyJhbGciOiJSUzI1Ni849urjwndjsncsh892943ıjfwks
`

### Kubernetes dashboard'ın Ayağa Kaldırıması:
Kubernetes dashboard'ı localde yayına alıp erişebilmek için aşağıdaki kodu çalıştırın:,
`Not: kubectl proxy çalıştırdıysanız kesinlikle aşağıdaki alternatif kodu çalıştırmayın.`
```
kubectl proxy
```

Bu adımda yukarıdaki koda alternatif olarak aşağıdaki koduda uygulayabilirsiniz: (Not:Tets edilmedi.)
`Not: yukarıdaki kodu çalıştırdıysanız kesinlikle kubectl proxy komutunu çalıştırmayın.`
```
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:443
```

Ayağa kalkan Kubernetes dashboard'a erişmek için bir internet tarayıcısı üzerinden aşağıdaki adrese gidin:
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

Tüm işlemler tamamlanmıştır.






