# Despliegue de ArgoCD en Kubernetes con Ansible

Este proyecto despliega automáticamente ArgoCD en Kubernetes, configurándolo para exponer su interfaz web mediante un servicio `NodePort` y un `Ingress` con Traefik.

## 📁 Estructura del proyecto

```
argocd-ansible/
├── inventory
│   └── hosts.ini
├── playbooks
│   └── deploy_argocd.yml
├── group_vars
│   └── all.yml
├── requirements.yml
└── README.md
```

## 🚀 Requisitos previos

- Ansible instalado en la máquina desde la cual ejecutarás el playbook.
- Acceso al clúster Kubernetes configurado (`~/.kube/config`).
- Colección Ansible para Kubernetes:

```bash
ansible-galaxy collection install -r requirements.yml
```

## 🛠️ Variables globales (`group_vars/all.yml`)

```yaml
argocd_namespace: argocd
ingress_host: argocd.local
node_port: 32004
```

## ▶️ Ejecución del playbook

Ejecuta el siguiente comando desde la raíz del proyecto:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/deploy_argocd.yml
```



```bash
nohup kubectl port-forward -n argocd svc/argocd-server --address 0.0.0.0 32004:80 > /tmp/argocd-port-forward.log 2>&1 &
```
https://192.168.0.15:32004/

## 🌐 Acceso a ArgoCD

Puedes acceder mediante dos opciones:

- **Usando NodePort directo:**
  ```
  http://<IP_NODO_WORKER>:32004
  ```

- **Usando Ingress con Traefik (recomendado):**
  ```
  http://argocd.local
  ```

> **Nota:** Asegúrate de agregar `argocd.local` al archivo `/etc/hosts` apuntando a la IP del Ingress.

## ✅ Validación del despliegue

Puedes verificar que ArgoCD esté funcionando correctamente:

```bash
kubectl get pods -n argocd -o wide

kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl get ingress -n argocd
```


export ARGOCD_AUTH_USER="admin"
export ARGOCD_AUTH_PASS="SuperPassword123"


¡Ahora tienes ArgoCD listo para gestionar tus aplicaciones mediante GitOps en Kubernetes!
