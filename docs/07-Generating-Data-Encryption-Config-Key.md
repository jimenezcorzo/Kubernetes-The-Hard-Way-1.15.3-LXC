# Generating the Data Encryption Config and Key

## The encryption at rest

 In this tasks, we will generate an encryption key and an encryption config suitable for encrypting Kubernetes *Secrets*. 
 
 Giving the Kubernetes cluster the hability to encrypt cluster data at rest.
 
 ## Generating the Encryption Key
```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```
## Creating the encryption-config.yaml file
```
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

## Transfering the encryption-config.yaml to the controller managers
```
lxc file push encryption-config.yaml controller-0/root/
lxc file push encryption-config.yaml controller-1/root/                                 
lxc file push encryption-config.yaml controller-2/root/         
```

Encryption key and config file generates and transfered to Master nodes.

# 
Return to: [main menu](https://github.com/jimenezcorzo/Kubernetes-The-Hard-Way-15.3-LXC/blob/master/Readme.md)
