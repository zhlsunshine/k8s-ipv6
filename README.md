# k8s-ipv6
This repo is for deployment of kubernetes with IPv6 support, there are 2 folds `ipv6-only` and `dual-stack`, including yaml files on how to install kubernetes components and calico via kubeadm and kubectl

## Reference docs:

- https://kubernetes.io/docs/concepts/services-networking/dual-stack/#enable-ipv4-ipv6-dual-stack
- https://sgryphon.wordpress.com/2021/01/05/kubernetes-on-ipv6-only/
- https://github.com/sgryphon/kubernetes-ipv6
- https://juejin.cn/post/6844903798687662094

## Set network for Ubuntu 18.04       
    Set `zhlsunshine` as the node name at first line of `/etc/hosts`
```
2001:db8:1234:5678::1 zhlsunshine
```

###### Letting iptables see bridged traffic:
###### https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-control-plane-node
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

## Add following network setting file named `ipv6-eno1.network` under the folder `/etc/systemd/network`
```ipv6-eno1.network
[Match]
Name=eno1

[Network]
DHCP=ipv4
Gateway=2001:db8:100:1ff:ff:ff:ff:ff
DNS=2001:41d0:3:163::1

[Address]
Address=2001:db8:1234:5678::1/64

[Route]
Destination=2001:db8:100:1ff:ff:ff:ff:ff
Scope=link
```

```
$ systemctl restart systemd-networkd
$ systemctl enable kubelet.service
```

## Set up the Kubernetes control plane       
```
$ kubeadm init --config=init-config-ipv6.yaml --dry-run | more
$ kubeadm init --config=init-config-ipv6.yaml
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

```
W0913 13:22:19.715101   24654 utils.go:69] The recommended value for "healthzBindAddress" in "KubeletConfiguration" is: 127.0.0.1; the provided value is: ::1
[init] Using Kubernetes version: v1.22.1
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileContent--proc-sys-net-ipv6-conf-default-forwarding]: /proc/sys/net/ipv6/conf/default/forwarding contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

Need to execute below command to handle this error
```
$ echo 1 > /proc/sys/net/ipv6/conf/default/forwarding
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

## Deploy a Container Network Interface (CNI)       
$ kubectl apply -f calico-ipv6.yaml
$ kubectl get all --all-namespaces


## Tips
- Host/Node setting for IPv6 package forwarding and iptable calling for bridge
- Container runtime setting for IPv6 support
- Kubernetes components deployment
