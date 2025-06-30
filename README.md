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
       ARGOCD_UI_ADMIN_PASSWORD=SuperAdmin123
sudo -E ansible-playbook -i inventory/hosts.ini playbooks/deploy_argocd_full.yml
```


```bash
source .env
export ARGOCD_AUTH_USER ARGOCD_AUTH_PASS
ansible-playbook -i inventory/hosts.ini playbooks/deploy_argocd_full.yml
``` 

# Cargar variables desde .env
source .env

# Exportar explícitamente las variables necesarias
export ARGOCD_AUTH_USER="$ARGOCD_AUTH_USER"
export ARGOCD_AUTH_PASS="$ARGOCD_AUTH_PASS"
export ARGOCD_UI_ADMIN_PASSWORD="$ARGOCD_UI_ADMIN_PASSWORD"

# Ejecutar el playbook con privilegios y entorno heredado
sudo -E ansible-playbook -i inventory/hosts.ini playbooks/deploy_argocd_full.yml




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



🔐 1. Autenticación de Traefik (middleware BasicAuth)
¿Qué hace?

Protege la URL pública (https://argocd.socialdevs.site) a nivel del proxy Traefik.

Actúa antes de que el tráfico llegue a Argo CD.

Usa credenciales en formato htpasswd (usuario/contraseña), generadas con Ansible y selladas con Sealed Secrets.

Dónde se define:

argocd_auth_user y argocd_auth_pass (env: ARGOCD_AUTH_USER, ARGOCD_AUTH_PASS)

Se usa en:

basic-auth-secret.yaml.j2

argocd-dashboard-middleware.yaml.j2

IngressRoute con middlewares: [...]

Objetivo:

Añadir una capa de seguridad HTTP básica (401) para proteger el acceso externo, incluso si el backend está vulnerable o sin TLS interno.

🔐 2. Autenticación nativa de Argo CD
¿Qué hace?

Es la autenticación real del sistema Argo CD, una vez que el usuario pasa el proxy.

Gestiona accesos, permisos, tokens, SSO, etc.

Usa el usuario admin y su contraseña hasheada (bcrypt) en un Secret.

Dónde se define:

argocd_admin_password_plain → se genera en .env con ARGOCD_UI_ADMIN_PASSWORD

Se hashea con htpasswd -nbBC 12

Se aplica en el Secret argocd-secret

Objetivo:

Permitir login a la UI o API de Argo CD con credenciales seguras.

🔄 Flujo completo de acceso web
less
Copiar
Editar
[ Cliente HTTP(S) ]
        ↓
[ Traefik IngressRoute con TLS + BasicAuth Middleware ] 🔐 (usuario: admin / pass: SuperPassword123)
        ↓
[ Servicio argocd-server en HTTP interno ]
        ↓
[ Login UI Argo CD ] 🔐 (usuario: admin / pass: SuperAdmin123)
        ↓
[ Acceso a Argo CD como plataforma GitOps ]