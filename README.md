
# CKA-k8s-buildCluster : Building a Kubernetes Cluster with Kubeadm for CKA exam


Your team is ready to deploy the first version of their new online storefront. The team is anticipating the potential for a high degree of customer usage after deployment, and they want to make use of Kubernetes in order to enable them to respond quickly to changing needs. In order to do this, they need a new Kubernetes cluster. You have been given the task of quickly spinning up a working Kubernetes cluster with one master and two worker nodes.

## install Docker on all three nodes.

* Do the following on all three nodes:

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
sudo apt-get update
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu
sudo apt-mark hold docker-ce
```

* Verify that Docker is up and running with
```sh
sudo systemctl status docker
```
Make sure the Docker service status is active (running)!


## Install Kubeadm, Kubelet, and Kubectl on all three nodes.

* Install the Kubernetes components by running this on all three nodes:
```sh
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet=1.12.7-00 kubeadm=1.12.7-00 kubectl=1.12.7-00
sudo apt-mark hold kubelet kubeadm kubectl
```


## Bootstrap the cluster on the Kube master node.

* On the Kube master node, do this:

```sh
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
That command may take a few minutes to complete.

* When it is done, set up the local kubeconfig:

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Take note that the `kubeadm init` command printed a long `kubeadm join` command to the screen. You will need that `kubeadm join` command in the next step!

* Run the following commmand on the Kube master node to verify it is up and running:

```sh
kubectl version
```
This command should return both a `Client Version` and a `Server Version`.


## Join the two Kube worker nodes to the cluster.

* Copy the kubeadm join command that was printed by the kubeadm init command earlier, with the token and hash. Run this command on both worker nodes, but make sure you add sudo in front of it:

```sh
sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash
```

* Now, on the Kube master node, make sure your nodes joined the cluster successfully:

```sh
kubectl get nodes
```

Verify that all three of your nodes are listed. It will look something like this:

```sh
NAME            STATUS     ROLES    AGE   VERSION
ip-10-0-1-101   NotReady   master   30s   v1.12.2
ip-10-0-1-102   NotReady   <none>   8s    v1.12.2
ip-10-0-1-103   NotReady   <none>   5s    v1.12.2
```
Note that the nodes are expected to be in the `NotReady` state for now.


## Set up cluster networking with flannel.

* Turn on iptables bridge calls on all three nodes:

```sh
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
* Next, run this only on the Kube master node:

```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

Now flannel is installed! Make sure it is working by checking the node status again:

```sh
kubectl get nodes
```

After a short time, all three nodes should be in the `Ready` state. If they are not all `Ready` the first time you run `kubectl get nodes`, wait a few moments and try again. It should look something like this:

```sh
NAME            STATUS   ROLES    AGE   VERSION
ip-10-0-1-101   Ready    master   85s   v1.12.2
ip-10-0-1-102   Ready    <none>   63s   v1.12.2
ip-10-0-1-103   Ready    <none>   60s   v1.12.2
```
