# Generating the Data Encryption Config and Key

Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to [encrypt](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) cluster data at rest.

In this lab you will generate an encryption key and an [encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration) suitable for encrypting Kubernetes Secrets.

## The Encryption Key

Generate an encryption key:

```shell
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File

Create the `encryption-config.yaml` encryption config file:

```shell
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

Copy the `encryption-config.yaml` encryption config file to each controller instance:

```shell
for instance in controller-0 controller-1 controller-2; do
  PUBLIC_IP_ADDRESS=$(az network public-ip show -g kubernetes \
    -n ${instance}-pip --query "ipAddress" -otsv)

  scp -o StrictHostKeyChecking=no encryption-config.yaml kuberoot@${PUBLIC_IP_ADDRESS}:~/
done
```

Next: [Bootstrapping the etcd Cluster](07-bootstrapping-etcd.md)
