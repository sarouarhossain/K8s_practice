----------------------------------------------------------------------------
All nodes
----------------------------------------------------------------------------

### step 1: initial configuration for docker before installing it
1. `
sudo mkdir /etc/docker
`  
`
ls /etc/docker/
`  

2. Create the daemon.json config file that Docker will use  
```
cat <<EOF | sudo tee /etc/docker/daemon.json
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
},
"storage-driver": "overlay2"
}
EOF
```  

3. Verify  
`
cat /etc/docker/daemon.json
`  

### step 2: update your packages, and install dependencies.

we will install following dependencies

1. apt-transport-https
2. ca-certificates
3. curl 
4. gnupg
4. lsb-release

`sudo apt-get update`  
`sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release`  


### step 3: update repositories and gpg keys for docker

1. 
``` 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```    

2. 
```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null 
```    

3. `sudo apt-get update`  

### step 4: add the packages and gpg key for kubernetes.
1. 
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable"
```
2. `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`

3. 
```
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list 
deb https://apt.kubernetes.io/ kubernetes-xenial main 
EOF
```
4. `sudo apt-get update`  

### step 5: installation of docker and kubernetes

```
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.14.5-00 kubeadm=1.14.5-00 kubectl=1.14.5-00
```

`sudo apt-mark hold docker-ce kubelet kubeadm kubectl`  

### step 6: enable iptable bridge call

```
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
```  

`sudo modprobe br_netfilter`  
`sudo sysctl -p`  

----------------------------------------------------------------------------
Only on Master node
----------------------------------------------------------------------------

### step 1: initialise the cluster

`sudo kubeadm init --pod-network-cidr=10.244.0.0/16`  


### step 2: set up local kubeconfig

`mkdir -p $HOME/.kube`  
`sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`  
`sudo chown $(id -u):$(id -g) $HOME/.kube/config`  

### step 3: install flannel networking

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```  

it will output a command which can be used by workers to join in the cluster

----------------------------------------------------------------------------
All Worker nodes
----------------------------------------------------------------------------
 ### step 1: join the cluster

 ```
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
 ```

Demo : kubeadm join 172.31.23.219:6443 --token pdizpa.mgkalhwkqvg6du82 --discovery-token-ca-cert-hash sha256:ada97328950955800b294666d679bfb543b53b5e4f695cb02650417a6bde9ec1

----------------------------------------------------------------------------
## Verification
----------------------------------------------------------------------------
On master server run the following command -

```kubectl get nodes```

