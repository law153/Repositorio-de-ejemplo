# Guía de despliegue

Este repositorio incluye un workflow de ejemplo para desplegar a un servidor remoto usando SSH y rsync:

Archivo del workflow: .github/workflows/deploy.yml

Secrets requeridos (configurar en Settings → Secrets and variables → Actions):

SSH_PRIVATE_KEY: clave privada SSH (sin passphrase o con passphrase gestionada por el runner). Debe corresponder a la clave pública autorizada en el servidor.
SSH_HOST: host o IP del servidor remoto (ej. example.com)
SSH_USER: usuario SSH en el servidor remoto (ej. ubuntu)
TARGET_DIR: ruta en el servidor donde se subirán los archivos (ej. /var/www/my-site)

**Notas de uso:**

El workflow detecta proyectos Node/Python de forma sencilla y ejecuta pasos básicos de build si encuentra package.json o requirements.txt/pyproject.toml.
Asegúrate de que el servidor remoto tenga permisos y suficiente espacio para recibir los archivos.
Puedes adaptar el workflow a Docker Hub, GitHub Pages, AWS, etc., cambiando la sección de deploy.

**Integración con Jira/Atlassian**

Este workflow ahora crea un GitHub Deployment y marca su estado como success. Si instalas la integración oficial
de Atlassian ↔️ GitHub (por ejemplo, la app "Atlassian for GitHub"), Jira Cloud puede detectar estos despliegues y
mostrarlos en la pestaña "Deployments" cuando haya asociaciones con commits o issues.

**Pasos recomendados:**

Instala la integración Atlassian ↔ GitHub desde el marketplace o desde la configuración de tu proyecto/organización.
Asegúrate de que tus commits incluyan claves de Jira (p. ej. PROJ-123) o que tus cambios estén vinculados mediante la app.
Opcional: en lugar de la integración, puedes usar la API de Deployments de Jira para reportar despliegues directamente (ver notas anteriores).

---

## Nuevo workflow de ejemplo ✅

He añadido un workflow de ejemplo en `.github/workflows/deploy.yml` que:

- Crea un GitHub Deployment y actualiza su estado (success/failure).
- Soporta `workflow_dispatch` con entradas `environment` y `test_only` para pruebas.
- Usa `webfactory/ssh-agent` + `rsync` para subir archivos a un servidor remoto via SSH.
- Detecta proyectos Node/Python y corre un build básico si aplica.
- Incluye una sección comentada para reportar despliegues a la API de Jira (opcional).

### Secrets recomendados 🔧

- `SSH_PRIVATE_KEY` (clave privada SSH)
- `SSH_HOST` (host o IP del servidor)
- `SSH_USER` (usuario SSH)
- `TARGET_DIR` (directorio destino en el servidor)

Opcionales: `SSH_PORT`, `RSYNC_EXCLUDES` (patrones separados por espacios), `JIRA_API_TOKEN`, `JIRA_USER_EMAIL`.

### Uso rápido 💡

- Ejecutar automáticamente al hacer push a `main`.
- Ejecutar manualmente desde Actions → Run workflow (o con `gh workflow run`) usando `environment` y `test_only=true` para pruebas.

---

## Ejemplo: crear un Deployment desde cero (chrnorm/deployment-action) 🔁

He añadido un workflow de ejemplo en `.github/workflows/create-deployment.yml` que sigue la recomendación de la acción `chrnorm/deployment-action` del Marketplace. Este workflow crea explícitamente un **Deployment** con estado inicial `pending`, ejecuta un paso de despliegue (demo) y luego marca el deployment como `success` o `failure` según corresponda.

Entradas (workflow_dispatch): `environment`, `ref`, `transient`, `description`, `task`, `test_only`.

Nota: este workflow está configurado para **ejecución manual solamente** (trigger: `workflow_dispatch`). Ejecuta el workflow desde **Actions → Run workflow** en GitHub o usando la CLI como se indica abajo.

Requisitos y notas:

- **Permisos**: el job necesita `deployments: write` (está incluido en el ejemplo).
- **Token**: por defecto se usa `GITHUB_TOKEN`. Si necesitas que el deployment *dispare otros workflows* o genere eventos que activen workflows, considera usar un token con más permisos (p. ej. `GITHUB_DEPLOY_TOKEN` como secret) porque `GITHUB_TOKEN` puede limitar la creación de otros workflow runs.

Uso rápido:

- Ejecutar desde Actions → Run workflow y completar `environment`/`transient` según sea necesario.
- Con GitHub CLI (ejemplo):
  `gh workflow run create-deployment.yml -f environment=staging -f test_only=true`

---

## Verificación rápida de `workflow_dispatch` ✅

Si recibes un error HTTP 422 al intentar disparar un workflow, normalmente significa que la versión del workflow que está en GitHub (rama `main`) no incluye `workflow_dispatch` o que el workflow que intentas ejecutar no está en la rama/commit que indicas.

Pasos para verificar y probar (recomendado):

1. Asegúrate de haber commit/pusheado los archivos del workflow a `main`:
   - `git add .github/workflows && git commit -m "Add workflows for manual dispatch" && git push origin main`
2. Lista los workflows remotos y confirma que el workflow soporta `workflow_dispatch`:
   - `gh workflow list --repo ${OWNER}/${REPO}`
   - `gh workflow view create-deployment.yml --repo ${OWNER}/${REPO} --yaml`
3. Prueba con un workflow de test (añadido: `.github/workflows/dispatch-test.yml`):
   - Ejecuta: `gh workflow run dispatch-test.yml -f test=ok`
   - O desde la UI: Actions → Dispatch test → Run workflow → rellenar `test` → Run.
4. Si prefieres usar la API con ID, primero confirma el ID:
   - `gh workflow list --repo ${OWNER}/${REPO}` (usa el ID o nombre que veas ahí).

> Consejo: ejecutar por `filename` (p. ej. `dispatch-test.yml`) evita errores por IDs que cambian entre commits.

---

¿Quieres que haga el commit por ti (si no lo has hecho) o prefieres que te guíe en los pasos para pushear y ejecutar el `dispatch-test` desde tu terminal?