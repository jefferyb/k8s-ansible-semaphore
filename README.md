# Ansible Semaphore on Kubernetes

Install Ansible Semaphore on Kubernetes using [kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/), or just copy `ansible-semaphore.yaml` and edit it before applying it

# Deploy Ansible Semaphore with kustomize | example

Create/Update `kustomization.yaml` file with ( replace 'ansible-semaphore.example.com' with your own url ):

```yaml
# Create a kustomization.yaml file
cat <<EOF >./kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: ansible-semaphore
resources:
  - https://github.com/jefferyb/k8s-ansible-semaphore.git

patches:
### Update some values
  - patch: |-
      apiVersion: core/v1
      kind: Secret
      metadata:
        name: semaphore-secret
        namespace: ansible-semaphore
      stringData:
        SEMAPHORE_ADMIN: admin
        SEMAPHORE_ADMIN_NAME: 'Administrator'
      data:
        # use echo -n <string-to-encod-to-base64> | base64 # to encode
        # use echo -n <base64-encoded-string> | base64 -d # to decode
        SEMAPHORE_DB_PASS: QW5zaWJsZVNlbWFwaG9yZQ==
        SEMAPHORE_ADMIN_PASSWORD: QW5zaWJsZVNlbWFwaG9yZQ==
        MYSQL_PASSWORD: QW5zaWJsZVNlbWFwaG9yZQ==
        MYSQL_ROOT_PASSWORD: QW5zaWJsZVNlbWFwaG9yZQ==

  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: semaphore
        namespace: ansible-semaphore
      spec:
        template:
          spec:
            volumes:
              # do not use hostPath, use volumeClaim in real world usecase
              - name: semaphore-db-vol
                hostPath:
                  path: /opt/volumes/semaphore
            containers:
              - name: semaphore
                env:
                  # use head -c32 /dev/urandom | base64 to generate the below value
                  - name: SEMAPHORE_ACCESS_KEY_ENCRYPTION
                    value: LV6A9ITvjz8RhYEhXUYNjOpN8DbFEutyoduARVnICsc=

### Update host value
  - patch: |-
      - op: replace
        path: /spec/rules/0/host
        value: ansible-semaphore.example.com # Replace with your url
    target:
      group: networking.k8s.io
      version: v1
      kind: Ingress
      name: semaphore

# ### Add TLS
#   - patch: |-
#       apiVersion: networking.k8s.io/v1
#       kind: Ingress
#       metadata:
#         name: semaphore
#         namespace: ansible-semaphore
#         annotations:
#           cert-manager.io/cluster-issuer: lets-encrypt
#       spec:
#         tls:
#           - secretName: semaphore-ingress-tls
#             hosts:
#               - ansible-semaphore.example.com # Replace with your url

EOF
```
review the deployment with:

```bash
kubectl kustomize .
```

apply/deploy with run:

```bash
kubectl kustomize . | kubectl apply -f -
#  OR
kubectl apply --kustomize .
```

ref: 
  * https://technekey.com/running-ansible-semaphore-for-gui-in-kubernetes/
  * https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/
