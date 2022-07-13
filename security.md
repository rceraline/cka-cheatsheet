# Certificates

Extensions used for public keys: `*.crt` or `*.pem`

Extensions used for private keys: `*.key` or `*-key.pem`

# Server Certificates for Servers

- ETCD server: `etcdserver.crt` & `etcdserver.key`
- API server: `apiserver.crt` & `apiserver.key`
- Kubelet server: `kubelet.crt` & `kubelet.key`

# Client Certificates for Clients

- Admin user: `admin.crt` & `admin.key`
- Scheduler: `scheduler.crt` & `scheduler.key`
- Controller Manager: `controller-manager.crt` & `controller-manager.key`
- Kube Proxy: `kube-proxy.crt` & `kube-proxy.key`
- Api Server (Kubelet client): `apiserver-kubelet-client.crt` & `apiserver-kubelet-client.key`
- Api Server (ETCD client): `apiserver-etcd-client.crt` & `apiserver-etcd-client.key`
- Kubelet Server: `kubelet-client.crt` & `kubelet-client.key`

# Certificate Authority (CA)

`ca.crt` & `ca.key`

# Generate Certificates with OpenSSL

## Generate Certificate for a CA

### Generate private key

```
openssl genrsa -out ca.key 2048
```

### Generate a Certificate Signing Request (CSR)

```
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```

### Sign the certificate (self signed by the CA by using the key generated for the CA)

```
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

## Generate Certificate for The admin user

### Generate private key

```
openssl genrsa -out admin.key 2048
```

### Generate a Certificate Signing Request (CSR)

```
openssl req -new -key admin.key -subj "/CN=KUBE-ADMIN/O=system:masters" -out ca.csr
```

## Signe the Certificate With the CA key

```
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

# Role & Role binding

## Check access

```
-- can create deployments?
kubectl auth can-i create deployments

-- check for a specific user
kubectl auth can-i create deployments as dev-user
```

## Service Accounts

## Image Security

Create a secret to store server, username password & email.

Use the built in type `docker-registry`

```
kubectl create secret docker-registry regcred \
  --docker-server=
  --docker-username=
  --docker-password=
  --docker-email=
```

To use the created secret, update the pod template like this:

```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred
```

## Security contexts

```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```

## Network Policies

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```
