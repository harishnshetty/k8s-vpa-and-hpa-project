# K8s-vpa-and-hpa-project

## For more projects, check out  
[https://harishnshetty.github.io/projects.html](https://harishnshetty.github.io/projects.html)

[![Video Tutorial](https://github.com/harishnshetty/image-data-project/blob/812115f2377a020bedc1419cd927a9edd4a11ee1/vpa%20hpa.jpg)](https://youtu.be/M6BxKpSvWa4)

[![Channel Link](https://github.com/harishnshetty/k8s-vpa-and-hpa-project/blob/2dbb594ead2451e5f17cb854180f3b55b760349b/VPA/img.jpg)](https://youtu.be/M6BxKpSvWa4)

# Required Packages installations

- Aws Cli
- Kubectl
- eksctl


# AWS Configure 
```bash
aws configure
```
## 1. AWS CLI Installation

Refer: [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

```bash
sudo apt install -y unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

---

## 2. kubectl Installation

Refer: [kubectl Installation Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg git

# If the folder `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring

# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly

sudo apt-get update
sudo apt-get install -y kubectl bash-completion

# Enable kubectl auto-completion
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -F __start_kubectl k' >> ~/.bashrc

# Apply changes immediately
source ~/.bashrc
```

---

## 3. eksctl Installation

Refer: [eksctl Installation Guide](https://eksctl.io/installation/)

```bash
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl

# Install bash completion
sudo apt-get install -y bash-completion

# Enable eksctl auto-completion
echo 'source <(eksctl completion bash)' >> ~/.bashrc
echo 'alias e=eksctl' >> ~/.bashrc
echo 'complete -F __start_eksctl e' >> ~/.bashrc

# Apply changes immediately
source ~/.bashrc
```

---

## 4. Create EKS Cluster and Nodegroup 
- 4 CPU and 8 GB RAM c6a.xlarge at $0.0935/hr  
- ( 2 Nodes X 6 Hours = 1.22$ ) [ 93.126INR ]
- 0.0935 × 2 nodes =0.187USD/hr    [ 0.187×83=15.521INR/hr]

- t3.medium

```bash
eksctl create cluster \
  --name my-cluster \
  --region ap-south-1 \
  --version 1.33 \
  --nodegroup-name standard-workers \
  --node-type c6a.xlarge \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 4 \
  --node-volume-size 20
```
---

## 5. Update kubeconfig

```bash
aws eks update-kubeconfig --name my-cluster --region ap-south-1
```

To check the nodes in your cluster run
```bash
kubectl get nodes
```
```bash
watch kubectl top node
```

## For VPA Documentation 
Refer: [Kodekloud](https://kodekloud.com/blog/vertical-pod-autoscaler/)


```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler/
```

```bash
kubectl get pods -n kube-system | grep vpa
```

```bash
./hack/vpa-up.sh
```

## Now Go to the VPA Path and apply the manifest files.

| Command                     | Description                             |
|-----------------------------|-----------------------------------------|
| kubectl top pod             | Show resource usage of pods             |
| kubectl describe pod vpa-deployment | Describe the VPA deployment pod     |
| watch kubectl get vpa       | Continuously monitor VPA resources      |
| watch kubectl get pods      | Continuously monitor pod status         |
| watch kubectl get nodes     | Continuously monitor node status        |


## Now Go to the HPA Path and apply the manifest files.


| Command                     | Description                             |
|-----------------------------|-----------------------------------------|
| kubectl top pod             | Show resource usage of pods             |
| watch kubectl get hpa       | Continuously monitor HPA resources      |
| watch kubectl get pods      | Continuously monitor pod status         |
| watch kubectl get nodes     | Continuously monitor node status        |


```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://hpa-svc; done"
```


## you can use normal command to Auto Scale hpa 

```bash
kubectl autoscale deployment hpa-deployment --cpu-percent=50 --min=1 --max=10
```

```bash
kubectl get hpa hpa-deployment --watch
```


# Delete the Cluster 
```bash
eksctl delete cluster --name my-cluster --region ap-south-1
```
