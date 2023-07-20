# Kubernates-Kurulumu-Ubuntu-Tr-Kaynak
Ubuntu v18.04 || 22.04 sürümleri üzerinde 3 node ile (controller-worker1-worker2) kubernates kurulumu ve bir adet örnek projenin sistemde ayağa kaldırılmasını içeren repo. Duruma göre node sayısı arttırılabilir fakat en az 2 node (controller-worker1) bulunmalıdır.

## Gereksinimler
kuruluma başlamadan önce:

- Ubuntu v18.04 || 22.04 sürümlerini kullanan 3 adet server. 
- Serverler en az 2 GB Ram ve 2 CPU ya sahip olmalıdır.
- Tüm serverler internete bağlı olmalıdır.
- Tüm serverler de ```sudo -s``` ile root yetkisine sahip olabilme yetkiniz bulunmaladır.

## Kurulum Adımları
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
Daha sonra `SystemdCgroup` yazan yeri buluyoruz. Oradki `false` ifadesini `true` ile değiştiriyoruz.

```
sudo nano /etc/containerd/config.toml
```
Set SystemdCgroup to true: `SystemdCgroup = true // bu bir komut değildir terminale yapıştırıp çalıştırmayın !!!`
`ctrl+s` kombinasyonu ile nano ile açıp düzenlediğimiz config dosyasını kayıt ediyoruz.
`ctrl+x` kombinasyonu ile nano ile açıp düzenlediğimiz config dosyasını kapatıyoruz.

Yaptığımız düzenlemelerin etkili olabilemesi için Containerd yapısını tekrardan başlatın.
```
sudo systemctl restart containerd
```
Terminal herhangi bir çıktı vermez ise endişlenmeyin, bazı kodlar çalıştırıldıklarında çıktı vermiyor.

## Kubernetes Kurulumu
Kubernetes kurulumu için, aşağıdaki kodları takip edin:
BU ADIMDA HATA OLABİLİR KANCA KOY EN ALTA ÇÖZÜM EKLE!
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

If there are any swap entries in the /etc/fstab file, remove them using a text editor such as nano:
```
sudo nano /etc/fstab
```
Açılan dosyada bu şekilde `/swapfile    none    swap    sw` bir satır bulumaktadır. Satırın başına `#` koyarak kodu bu şekilde `#/swapfile    none    swap    sw` işlevsiz hale getirin.
`ctrl+s` kombinasyonu ile nano ile açıp düzenlediğimiz /etc/fstab dosyasını kayıt ediyoruz.
`ctrl+x` kombinasyonu ile nano ile açıp düzenlediğimiz /etc/fstab dosyasını kapatıyoruz.

Enable kernel modules
```
sudo modprobe br_netfilter
```
*`sudo modprobe br_netfilter` komutu, makine yeniden başlatılırsa vb... işlevini kaybeder. Kalıcı ayarı set etmek için örnek bir kodu aşağıda paylaşacağım.*
*kalıcı kodu test etmedim. Makineler yeniden başlatıldıktan sonra hata alırsanız ve işin içinden çıkamayıp sıfırdan kurulum yapmak isterseniz:
kalıcı kodu kullanın veya `sudo modprobe br_netfilter` komutunu tekrardan girin.*

Add some settings to sysctl
```
sudo sysctl -w net.ipv4.ip_forward=1
```
*`sudo sysctl -w net.ipv4.ip_forward=1` komutu, makine yeniden başlatılırsa vb... işlevini kaybeder. Kalıcı ayarı set etmek için örnek bir kodu aşağıda paylaşacağım.*
*kalıcı kodu test etmedim. Makineler yeniden başlatıldıktan sonra hata alırsanız ve işin içinden çıkamayıp sıfırdan kurulum yapmak isterseniz:
kalıcı kodu kullanın veya `sudo sysctl -w net.ipv4.ip_forward=1` komutunu tekrardan girin.*

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

## Kurulumun Doğru Çalıştığını Doğrulanması
Aşağıdaki kodları uygulayın, doğru bir kurulumda gelen çıktıların benzeleri kodların altında paylaşılmıştır.
```
kubectl get pods --all-namespaces
```

## Node'lar ile Join Olun
To add nodes to the cluster, run the kubeadm join command with the appropriate arguments on each node. The command will output a token that can be used to join the node to the cluster.
