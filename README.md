# Despliegue de ArgoCD en Kubernetes con Ansible

Este proyecto despliega automáticamente ArgoCD en Kubernetes, configurándolo para exponer su interfaz web mediante un servicio `NodePort` y un `Ingress` con Traefik.


## 🚀 Requisitos previos

- Ansible instalado en la máquina desde la cual ejecutarás el playbook.
- Acceso al clúster Kubernetes configurado (`~/.kube/config`).
- Colección Ansible para Kubernetes:


# .env
ARGOCD_UI_ADMIN_PASSWORD=SuperAdmin123
ARGOCD_AUTH_USER=admin
ARGOCD_AUTH_PASS=SuperPassword123


export ARGOCD_UI_ADMIN_PASSWORD="admin"


export ARGOCD_AUTH_USER="admin"
export ARGOCD_AUTH_PASS="SuperPassword123"


# Despliegue completo de ArgoCD con Ansible

```bash
source .env
export ARGOCD_AUTH_USER="$ARGOCD_AUTH_USER" \
       ARGOCD_AUTH_PASS="$ARGOCD_ADMIN_PASSWORD"
sudo -E ansible-playbook -i inventory/hosts.ini playbooks/deploy_argocd_full.yml
```


```bash
source .env
export ARGOCD_AUTH_USER ARGOCD_AUTH_PASS
ansible-playbook -i inventory/hosts.ini playbooks/deploy_argocd_full.yml
``` 


sudo -E ansible-playbook -i inventory/hosts.ini playbooks/uninstall_argocd.yml

## ✅ Validación del despliegue

Puedes verificar que ArgoCD esté funcionando correctamente:

```bash
kubectl get pods -n argocd -o wide

kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl get ingress -n argocd
kubectl get ingress -n argocd -o wide
```