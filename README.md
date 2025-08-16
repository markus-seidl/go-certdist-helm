# go-certdist-helm

Helm chart for [go-certdist](https://github.com/markus-seidl/go-certdist/) to install the server inside an k8s cluster. Note

* that only the server mode is currently tested.
* that you need some way to map the certificates to the certdist pod.

## Example usage

```bash
helm repo add go-certdist https://markus-seidl.github.io/go-certdist-helm/
helm repo update go-certdist

helm upgrade --install -f secrets://values.sops.yaml --namespace certdist certdist go-certdist/go-certdist $*
```

```yaml
# values.yaml, encrypted with sops

image:
  pullPolicy: Always
  
# My lazy way to map the certificates into the container,
# via a symlink on the host and then a hostPath mount
volumes:
  - name: letsencrypt
    hostPath:
      path: /root/k3s_short/letsencrypt-live
      type: Directory

volumeMounts:
  - name: letsencrypt
    mountPath: /certs
    
ingress:
  hosts:
    - host: YOUR_DOMAIN
      paths:
        - path: /
          pathType: Prefix
  tls:
    - hosts:
        - YOUR_DOMAIN

certdist:
  config:
    server:
      port: 8080
      listen_address: 0.0.0.0
      certificate_directories:
        - /certs
    public_age_keys:
      - your-public-age-key-whitelist
```

