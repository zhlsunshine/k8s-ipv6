# k8s-ipv6
this repo is for deployment of kubernetes with IPv6 only

## Reference docs:
https://sgryphon.wordpress.com/2021/01/05/kubernetes-on-ipv6-only/
https://github.com/sgryphon/kubernetes-ipv6
https://juejin.cn/post/6844903798687662094

## Set network for Ubuntu 18.04       
    Set `zhlsunshine` as the node name at first line of `/etc/hosts`
2001:db8:1234:5678::1 zhlsunshine

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
$ kubeadm init --config=init-config-ipv6.yaml --dry-run | more
$ kubeadm init --config=init-config-ipv6.yaml
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
