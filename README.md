# Kubernates-Kurulumu-Ubuntu-Tr-Kaynak
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






