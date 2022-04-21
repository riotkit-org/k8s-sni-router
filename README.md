k8s-sni-router
==============

TLS-sni + HTTP-host router proxy on Kubernetes based on Traefik (optionally) and NGINX. **Designed to work with external hosts outside cluster.**

Features
--------

- Can mix HTTP & TCP traffic routing - operate on same port as normal services does (If using Traefik as ingress controller in Kubernetes)
- Route TLS traffic basing on TLS-SNI without decrypting the traffic
- Route HTTP using host header
- Support for Let's Encrypt pass-through, so the proxied server can normally issue Let's Encrypt certificates using HTTP-based method

Setting up
----------

```yaml
helm repo add riotkit-org https://riotkit-org.github.io/helm-of-revolution/
helm install my-sni-router riotkit-org/sni-router --values ./my-values.yaml
```

Example configuration
---------------------

```yaml

routes:
    - dns: federacja-anarchistyczna.pl
      ip: 185.208.164.110

nginx:
    workerProcesses: 4
    workerConnections: 2048
    metrics:
        allowIp: 127.0.0.1
    httpProxy:
        zoneLength: 20m
        limitConn: 10
    tlsProxy:
        zoneLength: 20m
        limitConn: 50
        connectTimeout: 15s
        timeout: 10800s
        preReadTimeout: 10800s

service:
    # alternatively you can switch to e.g. LoadBalancer type and disable Traefik
    useTraefik: true
    type: ClusterIP

podAnnotations: {}
fullNameOverride: ""

image:
    repository: nginx
    tag: "1.19"
```

Testing
-------

```bash
# where `172.18.0.3` is yours cluster load balancer IP address (e.g. Traefik)
curl -vik --resolve federacja-anarchistyczna.pl:443:172.18.0.3 --resolve federacja-anarchistyczna.pl:80:172.18.0.3 -H 'Host: federacja-anarchistyczna.pl' http://federacja-anarchistyczna.pl -L
```
