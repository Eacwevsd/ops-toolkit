# k8s 1.36 集群搭建

1. 克隆四台服务器，修改 IP 和 MAC 地址后，分别设置主机名

    ```bash
    hostnamectl set-hostname k8s-master
    hostnamectl set-hostname k8s-worker1
    hostnamectl set-hostname k8s-worker2
    hostnamectl set-hostname k8s-worker3
    ```

1. 修改 `/etc/hosts` 文件

    ```bash
    192.168.93.137 k8s-master
    192.168.93.138 k8s-worker1
    192.168.93.139 k8s-worker2
    192.168.93.140 k8s-worker3
    ```

1. 禁用 swap 分区

    ```bash
    sudo swapoff -a
    vim /etc/fstab
    # 将 swap 分区注释掉
    # /swapfile   none    swap    sw    0   0
    free -mh
    ```

1. 添加网桥过滤和内核转发配置文件

    ```bash
    sudo cat <<EOF | tee /etc/sysctl.d/kubernetes.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.ipv4.ip_forward = 1
    EOF

    # 添加内核模块
    sudo modprobe br_netfilter
    sudo modprobe overlay

    sudo cat <<EOF | tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

    # 验证生效
    sudo sysctl --system
    ```

1. 安装 containerd

    ```bash
    # 手动部署 containerd
    sudo wget https://github.com/containerd/containerd/releases/download/v2.3.1/containerd-2.3.1-linux-amd64.tar.gz

    sudo tar xf containerd-2.3.1-linux-amd64.tar.gz
    sudo mv bin/* /usr/bin/
    which containerd

    sudo wget -O /etc/systemd/system/containerd.service https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

    sudo vim /etc/systemd/system/containerd.service
    # 替换 ExecStart
    ExecStart=/usr/bin/containerd
    sudo systemctl daemon-reload
    sudo systemctl enable containerd
    sudo systemctl start containerd
    sudo systemctl status containerd

    # 检查
    sudo ctr version

    # 生成配置文件
    sudo mkdir -p /etc/containerd
    sudo containerd config default | sudo tee /etc/containerd/config.toml
    sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
    sudo sed -i 's|registry.k8s.io|registry.aliyuncs.com/google_containers|g' /etc/containerd/config.toml
    sudo systemctl restart containerd
    sudo ctr version
    ```

1. 配置镜像加速

   ```bash
   sudo mkdir -p /etc/containerd/certs.d/docker.io
   sudo tee /etc/containerd/certs.d/docker.io/hosts.toml << 'EOF'
   server = "https://docker.io"

   [host."https://docker.1ms.run"]
    capabilities = ["pull", "resolve"]

   [host."https://docker.m.daocloud.io"]
    capabilities = ["pull", "resolve"]

   [host."https://docker.mirrors.ustc.edu.cn"]
    capabilities = ["pull", "resolve"]

   [host."https://docker.nju.edu.cn"]
    capabilities = ["pull", "resolve"]

   [host."https://registry.docker-cn.com"]
    capabilities = ["pull", "resolve"]
   EOF

   sudo sed -i "s|config_path = ''|config_path = '/etc/containerd/certs.d'|" /etc/containerd/config.toml
   
   sudo systemctl restart containerd
   ```

1. 安装 runc

    ```bash
    sudo wget https://github.com/opencontainers/runc/releases/download/v1.4.2/runc.amd64
    sudo chmod +x runc.amd64
    sudo mv runc.amd64 /usr/local/sbin/runc
    sudo runc --version
    sudo systemctl restart containerd
    ```

1. k8s 软件源准备

    ```bash
    curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.36/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl cri-tools
    sudo apt-mark hold kubelet kubeadm kubectl cri-tools

    # 修复 crict 告警
    sudo tee /etc/crictl.yaml <<EOF
    runtime-endpoint: unix:///run/containerd/containerd.sock
    image-endpoint: unix:///run/containerd/containerd.sock
    timeout: 10
    debug: false
    EOF

    sudo systemctl enable kubelet
    ```

1. 修改默认配置文件，初始化集群

    ```bash
    sudo kubeadm config print init-defaults > kubeadm-config.yaml
    sudo vim kubeadm-config.yaml

    sudo kubeadm init --config=kubeadm-config.yaml
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    # 添加节点，在其他节点执行
    sudo kubeadm join 192.168.93.137:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:1b83ed3b37602e0d821cff0c04771da574028076af98b3786f50baa88184ff12
    
    kubectl get node
    ```

1. 部署网络插件

    ```bash
    kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
    kubectl get pod -A
    ```
