# Container Registry mirrors of Kubernetes

Bypass GFW, container registry mirrors for Kubernetes developer in China mainland.

## Quick start

- Replace `REPLACE_ME_DOMAIN` with your domain in `caddy/sites-enabled/mirror-registry`
- Replace `REPLACE_ME_WITH_YOUR_CF_API_TOKEN` with your own Cloudflare api token in `docker-compose.yml`

```
docker compose up -d
```

## Registry list

| origin          | mirror                      |
| --------------- | --------------------------- |
| registry.k8s.io | k8s-registry.example.com    |
| docker.io       | docker-registry.example.com |
| quay.io         | quay-registry.example.com   |

## Usage

### Kubeadm

```shell
$ kubeadm config images pull --image-repository=k8s-registry.example.com
[config/images] Pulled k8s-registry.example.com/kube-apiserver:v1.20.6
[config/images] Pulled k8s-registry.example.com/kube-controller-manager:v1.20.6
[config/images] Pulled k8s-registry.example.com/kube-scheduler:v1.20.6
[config/images] Pulled k8s-registry.example.com/kube-proxy:v1.20.6
[config/images] Pulled k8s-registry.example.com/pause:3.2
[config/images] Pulled k8s-registry.example.com/etcd:3.4.13-0
[config/images] Pulled k8s-registry.example.com/coredns:1.7.0
```

### Kubespray

- container image `quay-registry.example.com/kubespray/kubespray:$TAG`

```shell
$ docker pull quay-registry.example.com/kubespray/kubespray:v2.15.1
```

- `roles/download/defaults/main.yml`

```yaml
kube_image_repo: "k8s-registry.example.com"
docker_image_repo: "docker-registry.example.com"
quay_image_repo: "quay-registry.example.com"
```
