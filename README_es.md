# 🚀 Prueba Localmente con Red Hat Developer Hub (RHDH)

RHDH Local permite a los ingenieros de plataforma probar rápidamente:
- Catálogos de software
- Documentación técnica
- Plugins y plantillas
- Personalizaciones de página de inicio
- Configuraciones de RHDH

**¡Sin necesidad de un clúster de Kubernetes!**

> ⚠️ **Notas importantes**  
> - 🚫 **NO** reemplaza al Red Hat Developer Hub oficial  
> - 🏭 **NO** es adecuado para entornos de producción  
> - 👥 **NO** soporta trabajo en equipo (sin RBAC)  
> - 🔧 **NO** tiene soporte oficial - úselo bajo su propio riesgo  
> - 💡 Extremadamente útil para desarrollo individual  

## 📋 Tabla de Contenidos
- [Requisitos](#-prerequisites)
- [Inicio Rápido](#-quick-start)
- [Gestión de Configuración](#-configuration-management)
- [Plugins Dinámicos](#-dynamic-plugins)
- [Configuraciones Avanzadas](#-advanced-configurations)
- [Limpieza](#-cleanup)
- [Soporte y Contribuciones](#-support--contributions)
- [Licencia](#-license)

## 📋 Requisitos

| Requisito              | Obligatorio? | Detalles                         |
|------------------------|--------------|----------------------------------|
| PC x86 64-bit          | Sí           | Soporte ARM próximamente         |
| Docker/Podman          | Sí           | Con recursos adecuados           |
| Conexión a Internet    | Sí           | Para descargar imágenes/plugins  |
| Git                    | Opcional     | Para clonar el repositorio       |
| Cuenta GitHub          | Opcional     | Para integración con GitHub      |
| Node.js (npx)          | Opcional     | Para autenticación GitHub        |
| Cuenta Red Hat         | Opcional     | Para usar PostgreSQL             |

## 🚀 Inicio Rápido

1. **Clona el repositorio**:
   ```bash
   git clone https://github.com/redhat-developer/rhdh-local.git
   cd rhdh-local
   ```

2. **Configuración para Mac ARM (M1/M2)**:
   ```bash
   echo 'RHDH_IMAGE=quay.io/rhdh-community/rhdh:next' >> .env
   ```

3. **Personaliza tu configuración (opcional)**:
   ```bash
   cp configs/app-config/app-config.local.example.yaml configs/app-config/app-config.local.yaml
   cp configs/dynamic-plugins/dynamic-plugins.override.example.yaml configs/dynamic-plugins/dynamic-plugins.override.yaml
   ```

4. **Inicia los contenedores**:
   ```bash
   # Con Docker
   docker compose up -d
   
   # O con Podman
   podman-compose up -d
   ```

5. **Accede a la interfaz**:
   Abre [http://localhost:7007](http://localhost:7007) en tu navegador

## 🔧 Gestión de Configuración

### Actualizando Configuración de App
```bash
podman-compose stop rhdh && podman-compose start rhdh
```

### Actualizando Plugins
```bash
podman-compose run install-dynamic-plugins
podman-compose stop rhdh && podman-compose start rhdh
```

## 🧩 Plugins Dinámicos

Para añadir plugins locales:

1. Copia los archivos a `local-plugins/`
2. Configura los permisos:
   ```bash
   chmod -R 777 local-plugins
   ```
3. Edita el archivo de configuración apropiado:
   - `configs/dynamic-plugins/dynamic-plugins.override.yaml` (preferido)
   - `configs/dynamic-plugins/dynamic-plugins.yaml`

<details>
<summary>📄 Ejemplo de dynamic-plugins.override.yaml</summary>

```yaml
includes:
  - dynamic-plugins.default.yaml
# Tus configuraciones adicionales aquí
```
</details>

## ⚙️ Configuraciones Avanzadas

### 🔄 Cambiando Imágenes de Contenedor
Edita tu `.env`:
```bash
# Para versión comunitaria
RHDH_IMAGE=quay.io/rhdh-community/rhdh:next

# Para versión oficial
RHDH_IMAGE=quay.io/rhdh/rhdh-hub-rhel9:1.y
```

### 🗃️ Usando PostgreSQL
1. Inicia sesión en el registro:
   ```bash
   podman login registry.redhat.io
   ```
2. Descomenta la sección `db` en `compose.yaml`
3. Comenta la configuración SQLite en `app-config.local.yaml`

### 🕵️ Depuración con VSCode
1. Inicia en modo debug:
   ```bash
   podman-compose -f compose.yaml -f compose-debug.yaml up
   ```
2. Configura `launch.json`:
   ```json
   {
     "version": "0.2.0",
     "configurations": [
       {
         "name": "Conectar a RHDH",
         "type": "node",
         "request": "attach",
         "port": 9229,
         "localRoot": "${workspaceFolder}",
         "remoteRoot": "/opt/app-root/src/dynamic-plugins-root/plugin-name"
       }
     ]
   }
   ```

## 🧹 Limpieza
Para eliminar todo:
```bash
# Eliminar contenedores y volúmenes
podman-compose down --volumes

# Limpieza completa (Docker)
docker system prune --volumes

# Limpieza completa (Podman)
podman system prune --volumes
```

## ❓ Soporte y Contribuciones

Para reportar problemas:
- [JIRA](https://issues.redhat.com/browse/RHIDP) (Componente: RHDH Local)

¡Contribuciones son bienvenidas!

## 📜 Licencia
```text
Copyright Red Hat

Licenciado bajo la Licencia Apache, Versión 2.0 (la "Licencia");
no puede usar este archivo excepto en cumplimiento con la Licencia.
Puede obtener una copia de la Licencia en

http://www.apache.org/licenses/LICENSE-2.0
```
