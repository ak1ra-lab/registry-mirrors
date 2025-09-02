# ak1ra-lab/registry-mirrors

| registry mirrors             | proxy_remoteurl                | origin            |
| ---------------------------- | ------------------------------ | ----------------- |
| docker-registry.example.com  | `https://registry-1.docker.io` | docker.io         |
| k8s-registry.example.com     | `https://registry.k8s.io`      | registry.k8s.io   |
| gcr-registry.example.com     | `https://gcr.io`               | gcr.io            |
| quay-registry.example.com    | `https://quay.io`              | quay.io           |
| elastic-registry.example.com | `https://docker.elastic.co`    | docker.elastic.co |

## Introduction

本仓库 fork 自 [IceCodeNew/registry-mirrors](https://github.com/IceCodeNew/registry-mirrors) 和 [muzi502/registry-mirrors](https://github.com/muzi502/registry-mirrors), 感谢 [IceCodeNew](https://github.com/IceCodeNew) 和原作者 [muzi502](https://github.com/muzi502). muzi502/registry-mirrors 原本的实现就是 Nginx, IceCodeNew/registry-mirrors 将其适配为 Caddy 并增加了许多其它功能, ~~而我又把 Caddy 改回了 Nginx.~~

因为在我的使用场景中, IceCodeNew/registry-mirrors 项目存在两个小问题,

- 虽然在 container 中直接运行 Caddy 非常"开箱即用", 但是它会占用服务器的 80, 443 端口, 对于非专用的服务器而言, 这样不太合适
- 而 compose.yml 中大部分 registry service 的配置基本一致, 决定通过 Jinja2 模板来创建 compose.yml

## Deployment

本地需要安装好 Ansible,

```shell
# 注意替换 playbook_hosts 与 domain 为你实际的值
ansible-playbook site.yml -e 'playbook_hosts=localhost' -e 'domain=example.com' -e 'certbot_challenge_type=http01' -v -b
```

启用 Nginx 配置的情况下仅生成 Nginx 配置文件, 不会执行 `nginx -s reload`, 需要自行执行 playbook 执行完毕后输出的 certbot 指令以签发 TLS 证书, 在公网场景下, 我们在使用 HTTP01 challenge 签发证书时需要提前为域名添加好 DNS 解析, 内网场景下只能使用 DNS01 challenge, 无需提前配置 DNS 解析.

在 Debian 中安装 certbot 可参考以下指令,

```shell
apt install certbot python3-certbot-nginx python3-certbot-dns-cloudflare
```

## Usage

### docker cli

```shell
docker image pull docker-registry.example.com/library/caddy:2-alpine
docker image pull docker-registry.example.com/library/debian:trixie-slim
```

### docker daemon

在 `/etc/docker/daemon.json` 中加入 `registry-mirrors: []`,

```json
{
  "registry-mirrors": ["https://docker-registry.example.com"]
}
```

### kubeadm

注意, coredns 位于 `registry.k8s.io/coredns/coredns`, 因此在使用 `kubeadm config images pull` 预先拉取镜像时, 需要覆盖 `dns.imageRepository` 的值, 只设定 `--image-repository` 时会尝试拉取 `k8s-registry.example.com/coredns:$TAG`,

```shellsession
# cat kubeadm-config-images.yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
kubernetesVersion: v1.32.8
imageRepository: k8s-registry.example.com
dns:
  imageRepository: k8s-registry.example.com/coredns

# kubeadm config images pull --config kubeadm-config-images.yaml
[config/images] Pulled k8s-registry.example.com/kube-apiserver:v1.32.8
[config/images] Pulled k8s-registry.example.com/kube-controller-manager:v1.32.8
[config/images] Pulled k8s-registry.example.com/kube-scheduler:v1.32.8
[config/images] Pulled k8s-registry.example.com/kube-proxy:v1.32.8
[config/images] Pulled k8s-registry.example.com/coredns/coredns:v1.11.3
[config/images] Pulled k8s-registry.example.com/pause:3.10
[config/images] Pulled k8s-registry.example.com/etcd:3.5.16-0
```

### kubespray

kubespray image repo 相关的配置位于 [`roles/kubespray_defaults/defaults/main/download.yml`](https://github.com/kubernetes-sigs/kubespray/blob/50c5f39a9d3fa674490b1245113960a5184b5c11/roles/kubespray_defaults/defaults/main/download.yml#L88-L91), 可以在 inventory 中覆盖,

```yaml
# gcr and kubernetes image repo define
gcr_image_repo: "gcr-registry.example.com"
kube_image_repo: "k8s-registry.example.com"
kubeadm_image_repo: "{{ kube_image_repo }}"
```

## TODOs

[caddyserver/caddy](https://github.com/caddyserver/caddy) 官方构建不会携带第三方 DNS plugin, 而大多数需要 DNS01 challenge 的场景都需要自行使用 xcaddy 从源码构建, 这种情况下感觉不如用 container 来运行 caddy 方便. 考虑到其它需要反向代理的服务, 把 caddy 直接集成在 registry-mirrors 这个项目感觉不太合适, 或许应该另起一个项目, 以 host network 模式运行 caddy server.
