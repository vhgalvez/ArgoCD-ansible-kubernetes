# Despliegue automatizado de Argo CD con Ansible + Traefik

El objetivo de este proyecto es instalar Argo CD en tu clúster Kubernetes, exponer la interfaz web mediante Traefik IngressRoute y añadir doble capa de autenticación (BasicAuth en Traefik + login nativo de Argo CD).

## Requisitos

| Requisito                          | Detalle                                      |
|------------------------------------|----------------------------------------------|
| **Ansible**                        | 2.10 o superior                              |
| **Colección Ansible kubernetes.core** | `ansible-galaxy collection install kubernetes.core` |
| **Acceso al clúster (kube-config)** | `~/.kube/config` con contexto por defecto    |
| **Traefik IngressController**       | Versión 2.x con TLS (Let’s Encrypt o similar)|

## Variables sensibles (.env)

Crea un archivo `.env` en la raíz del repo e incluye solo estas claves:

```ini
# Credenciales que protegerán el proxy Traefik (BasicAuth)
ARGOCD_AUTH_USER=admin
ARGOCD_AUTH_PASS=SuperPassword123

# Contraseña para el usuario admin de Argo CD
ARGOCD_UI_ADMIN_PASSWORD=SuperAdmin123
```
> **Nota:** No publiques nunca este archivo en tu repositorio.

## Despliegue paso a paso

1. **(Opcional) Desinstalar Argo CD si ya está desplegado**
   ```bash
   sudo -E ansible-playbook -i inventory/hosts.ini playbooks/uninstall_argocd.yml
   ```

2. **Cargar variables de entorno desde el archivo `.env`**
   ```bash
   source .env
   ```

3. **Exportar explícitamente las variables necesarias para Ansible**
   ```bash
   export ARGOCD_AUTH_USER="$ARGOCD_AUTH_USER"
   export ARGOCD_AUTH_PASS="$ARGOCD_AUTH_PASS"
   export ARGOCD_UI_ADMIN_PASSWORD="$ARGOCD_UI_ADMIN_PASSWORD"
   ```

4. **Ejecutar el playbook de despliegue completo de ArgoCD**
   ```bash
   sudo -E ansible-playbook -i inventory/hosts.ini playbooks/deploy_argocd_full.yml
   ```

## Validar el despliegue

Puedes verificar que ArgoCD esté funcionando correctamente:

```bash
kubectl get pods -n argocd -o wide
kubectl get svc -n argocd
kubectl get ingress -n argocd -o wide
```

Después abre:

```bash
https://argocd.<TU_DOMINIO>
```

- **Traefik BasicAuth** → `admin / SuperPassword123`
- **Login Argo CD** → `admin / SuperAdmin123`

## Capas de seguridad

| Capa                          | Archivo Jinja / Recurso                      | Usuario / Contraseña (ejemplo) |
|-------------------------------|-----------------------------------------------|--------------------------------|
| **BasicAuth (Traefik)**       | `basic-auth-secret.yaml.j2` + `argocd-dashboard-middleware.yaml.j2` | `admin / SuperPassword123`     |
| **Login nativo (Argo CD)**    | `argocd-secret` (contraseña bcrypt)           | `admin / SuperAdmin123`        |

## Flujo de acceso

```text
Cliente ──▶ Traefik Ingress (TLS + BasicAuth) ──▶ Service argocd-server (HTTP) ──▶ UI Argo CD (login nativo)
```

## Desinstalar

1. **Cargar variables de entorno**
   ```bash
   source .env
   ```

2. **Ejecutar el playbook de desinstalación**
   ```bash
   sudo -E ansible-playbook -i inventory/hosts.ini playbooks/uninstall_argocd.yml
   ```
