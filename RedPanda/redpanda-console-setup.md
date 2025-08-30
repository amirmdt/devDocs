# Redpanda Console Setup with Authentication

## Introduction

[Redpanda](https://redpanda.com) is a modern, Kafka-compatible, streaming data platform.  
It is designed to be **fast**, **simple**, and **cost-efficient**, with no ZooKeeper dependency.  
It provides a **console UI** that allows developers and operators to manage clusters, topics, and messages.

By default, the **Redpanda Console UI** is accessible without authentication, which poses a **security risk**.  
In this guide, we set up **basic authentication** for the Redpanda Console using **Kubernetes Ingress + NGINX**.

---

## Steps

### 1. Install Redpanda Console

We deploy the Redpanda Console using the official **Helm chart**.

Example `redpanda-console-values.yaml`:

```yaml
replicaCount: 1

service:
  type: ClusterIP
  port: 8080

config:
  kafka:
    brokers:
      - kafka-broker-0-external.kafka-test.svc.cluster.local:9094
      - kafka-broker-1-external.kafka-test.svc.cluster.local:9094
      - kafka-broker-2-external.kafka-test.svc.cluster.local:9094

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: redpanda-console-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
  hosts:
    - host: redpanda-console.local.astek.ir
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: sslkeys
      hosts:
        - redpanda-console.local.astek.ir
```

---

### 2. Create Basic Authentication Secret

We use `htpasswd` to generate a username and password.

```bash
sudo apt-get install apache2-utils -y

# Create htpasswd file with first user
htpasswd -c auth redpanda

# Add more users (do NOT use -c, otherwise it overwrites)
htpasswd auth user2
htpasswd auth user3
```

Now create the Kubernetes secret:

```bash
kubectl create secret generic redpanda-console-auth   --from-file=auth
```

This secret is referenced by the Ingress annotations.

---

### 3. Deploy the Console

```bash
helm upgrade --install redpanda-console redpanda/console   -f redpanda-console-values.yaml -n kafka-test
```

After deployment, access the console at:

👉 `https://redpanda-console.local.astek.ir`  
Login using one of the usernames/passwords created earlier.

---

## Notes

- Redpanda Enterprise has built-in authentication for the console.  
  Since we used the **open-source version**, we applied authentication at the **Ingress level**.

- You can manage multiple usernames and passwords in the same secret (`htpasswd` file).

- TLS is handled by the `sslkeys` secret in the Ingress definition.

---

## Summary

- Deployed **Redpanda Console** via Helm.  
- Configured **NGINX Ingress** with **basic authentication**.  
- Secured access using `htpasswd` credentials.  
- Ensured console access is now limited to authenticated users.

✅ The Redpanda Console is now **secured and production-ready**.
