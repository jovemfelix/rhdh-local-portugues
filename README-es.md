# Pruebe localmente con Red Hat Developer Hub (RHDH)

- ¡Bienvenido a RHDH Local!
- ¡La forma más rápida y sencilla para que los ingenieros de plataformas prueben sus catálogos de software, documentos técnicos, complementos, plantillas, personalizaciones de páginas de inicio, configuraciones y más con RHDH!
- RHDH Local es ideal para probar funciones básicas de RHDH (como catálogos de software o documentos técnicos) sin la necesidad de un clúster de Kubernetes.
- RHDH Local también es ideal para probar complementos dinámicos y sus configuraciones.
- Para utilizar RHDH Local, todo lo que necesitas es un conocimiento básico de Docker o Podman, una PC y un navegador web.
- Puedes ejecutarlo en tu computadora portátil, computadora de escritorio o en tu laboratorio en casa.
- Mejor aún, cuando hayas terminado de trabajar, es fácil quitarlo.

> - **RHDH Local NO es un reemplazo de Red Hat Developer Hub**.
> - No intente utilizar RHDH Local como un sistema de producción.
> - RHDH Local está diseñado para ayudar a los desarrolladores individuales a probar varias características de RHDH.
> - No está diseñado para escalar y **no** es adecuado para su uso en equipos (no hay **RBAC**, por ejemplo).
> - Tampoco hay **ningún** soporte _oficial_ (actualmente) para RHDH Local.
> - Usted utiliza RHDH Local bajo su propio riesgo.
> Dicho esto, creemos que es increíblemente útil y cualquier aporte que pueda tener para mejorar RHDH Local será bienvenido.

## Lo que necesitas antes de empezar

Para utilizar RHDH Local, necesitará algunos elementos:

1. Una PC con arquitectura x86 de 64 bits (amd64) (próximamente ARM)
1. Una instalación de Docker o Podman (con los recursos adecuados disponibles)
1. Una conexión a Internet (para descargar imágenes de contenedores, complementos, etc.)
1. (_Opcional_) El cliente de línea de comandos `git` para clonar este repositorio (o puede descargar y extraer el archivo ZIP de GitHub)
1. (_Opcional_) Una cuenta de GitHub (si desea integrar recursos de GitHub con RHDH)
1. (_Opcional_) La herramienta de nodo `npx` (si desea utilizar la autenticación de GitHub en RHDH)
1. (_Opcional_) Una [cuenta Red Hat](https://access.redhat.com/RegistryAuthentication#getting-a-red-hat-login-2) (si desea utilizar una base de datos PostgreSQL)

### Nota para usuarios de Mac M1

- Si está utilizando una Mac Apple Silicon (M1/M2), la imagen RHDH predeterminada (`quay.io/rhdh/rhdh-hub-rhel9:1.4`) **no** es compatible con la arquitectura ARM64.
- Para solucionar esto, puede agregar esta línea a su archivo `.env` (cree el archivo si no existe):

`RHDH_IMAGE=quay.io/rhdh-community/rhdh:siguiente`

- Esta imagen es compatible con `amd64` y `arm64`.

## Introducción a RHDH local

1. Clona este repositorio en una ubicación en tu PC

```sh
git clone https://github.com/redhat-developer/rhdh-local.git
```

1. Vaya a la carpeta `rhdh-local`. 

```sh
cd rhdh-local
```

1. (_Opcional_) Puede crear un archivo `.env` local y anular cualquiera de las variables predeterminadas definidas en el archivo [`default.env`](./default.env) proporcionado. También puedes agregar variables adicionales. En la mayoría de los casos, cuando **no** necesitas GitHub Auth o probar diferentes versiones, puedes omitir este paso y debería funcionar.

1. (_Opcional_) Realizar una anulación de la configuración local. RHDH Local admite modificaciones de configuración específicas del usuario mediante un directorio `configs/` estructurado. **No** es necesario modificar los archivos predeterminados. Sin embargo, si desea personalizar su configuración:

   - Agregue las modificaciones de configuración de su aplicación a: `configs/app-config/app-config.local.yaml`
      > Puede utilizar los archivos `.example.yaml` incluidos para comenzar rápidamente:
      >
      > ```sh
      > cp configs/app-config/app-config.local.example.yaml configs/app-config/app-config.local.yaml
      > cp configs/dynamic-plugins/dynamic-plugins.override.example.yaml configs/dynamic-plugins/dynamic-plugins.override.yaml
      > ```

   - Agregue la configuración de su complemento para que se anule a:
     `configs/dynamic-plugins/dynamic-plugins.override.yaml`
     > El archivo de reemplazo debe comenzar con:
     > ```yaml
     > includes:
     >   - dynamic-plugins.default.yaml
     > ```
     > Esto garantiza que la lista de complementos base se conserve y se amplíe, en lugar de reemplazarse.

   - Agregue cualquier archivo adicional (como credenciales de GitHub) a: `configs/extra-files/`

   > Si están presentes, estos archivos serán cargados automáticamente por el sistema al iniciarse. 
     
   Si necesita funciones que obtengan archivos de GitHub, configúrelas`integrations.github`.
   La forma recomendada es utilizar GitHub Apps. Puede encontrar sugerencias sobre cómo configurarlo en [github-app-credentials.example.yaml](configs/github-app-credentials.example.yaml) o instrucciones más detalladas en la [documentación de Backstage](https://backstage.io/docs/integrations/github/github-apps).

2. Inicie RHDH Local. 
   Este repositorio debería funcionar con `docker compose` usando Docker Engine o `podman-compose` usando Podman. Al utilizar Podman, hay algunas excepciones. Consulta [Problemas conocidos al usar Podman Compose](#known-issues-when-using-podman-compose) para obtener más información.

   ```sh
   podman-compose up -d
   ```

   Si prefieres `docker compose`, puedes simplemente reemplazar `podman-compose` con `docker compose`

   ```sh
   docker compose up -d
   ```

3. Abra [http://localhost:7007](http://localhost:7007) en su navegador para acceder a RHDH.

## Cambiar su configuración
   
Al cambiar `app-config.local.yaml`, debe reiniciar el contenedor `rhdh` para cargar la configuración RHDH actualizada.

```sh
podman-compose stop rhdh && podman-compose start rhdh
```

Al cambiar `dynamic-plugins.yaml`, debe volver a ejecutar el contenedor `install-dynamic-plugins` y reiniciar la instancia RHDH.

```sh
podman-compose run install-dynamic-plugins
podman-compose stop rhdh && podman-compose start rhdh
```

## Cargar complementos dinámicos desde un directorio local
   
Durante la inicialización, el contenedor `install-dynamic-plugins` lee el contenido del archivo de configuración del complemento y activa, configura o descarga cualquier complemento enumerado. RHDH Local admite dos formas de especificar la configuración dinámica del complemento:
   
1. Ruta **predeterminada**: `configs/dynamic-plugins/dynamic-plugins.yaml`

1. Ruta de **anulación** del usuario: `configs/dynamic-plugins/dynamic-plugins.override.yaml` o `configs/dynamic-plugins.yaml`. Si está presente, este archivo anulará automáticamente el valor predeterminado y lo utilizará el contenedor `install-dynamic-plugins`. `configs/dynamic-plugins/dynamic-plugins.override.yaml` tiene prioridad sobre `configs/dynamic-plugins.yaml`.

Además, el directorio `local-plugins` está montado en el contenedor `install-dynamic-plugins` en `/opt/app-root/src/local-plugins`. Cualquier complemento instalado allí se puede activar/configurar de la misma manera (sin descargarlo).

Para cargar complementos dinámicos desde su máquina local:

1. Copie el archivo binario del complemento dinámico al directorio `local-plugins`.
2. Asegúrese de que los permisos permitan al contenedor leer los archivos (por ejemplo, `chmod -R 777 local-plugins` para pruebas rápidas).
3. Configure su complemento en uno de los archivos de configuración compatibles:
- Prefiera `configs/dynamic-plugins/dynamic-plugins.override.yaml` para anulaciones de usuarios locales.
- Si no hay ningún archivo de anulación presente, se utilizará `configs/dynamic-plugins/dynamic-plugins.yaml`.
4. Consulte [Cambiar su configuración](#change-your-configuration) para obtener más información sobre cómo actualizar y recargar configuraciones.

## Opcional: Personalice `.npmrc` para la instalación del complemento
   
Si está instalando complementos dinámicos desde un registro privado o usando un proxy, puede personalizar su propio archivo `.npmrc`. Se proporciona un archivo `.npmrc.example` en el directorio `configs/` como plantilla.
   
1. Copie el archivo de ejemplo para crear su propio `.npmrc`:
   
```sh
cp configs/.npmrc.ejemplo configs/.npmrc
```
   
2. Abra el archivo `.npmrc` recién creado y agregue su configuración, como URL de registro privado o tokens de autenticación:
   
    ```sh
    //registry.npmjs.org/:_authToken=YOUR_TOKEN
    registry=https://your-private-registry.example.com/
    ```

Cuando esté presente, este archivo `.npmrc` se montará automáticamente en el contenedor `install-dynamic-plugins` y la variable de entorno `NPM_CONFIG_USERCONFIG` se configurará para apuntar a él.
   
Si no crea un `.npmrc`, la instalación del complemento seguirá funcionando utilizando la configuración predeterminada del registro público.
   
> Para obtener más información sobre cómo configurar `.npmrc`, consulte la [documentación de configuración de npm](https://docs.npmjs.com/cli/v10/configuring-npm/npmrc).

## Cambiar la imagen del contenedor
   
Puede cambiar entre las versiones [downstream](https://quay.io/repository/rhdh/rhdh-hub-rhel9?tab=tags) y [comunitaria](https://quay.io/repository/rhdh-community/rhdh?tab=tags) de RHDH cambiando el nombre de la imagen del contenedor mantenido por la variable de entorno `RHDH_IMAGE` en su archivo `.env`.
   
Por ejemplo, para utilizar la compilación nocturna de la comunidad, configure la variable de la siguiente manera:

```sh
RHDH_IMAGE=quay.io/rhdh-community/rhdh:next
```

Para utilizar la versión oficial RHDH 1.y, configure la variable de la siguiente manera (reemplace `y` según sea necesario):

```sh
RHDH_IMAGE=quay.io/rhdh/rhdh-hub-rhel9:1.y
```

## Prueba de RHDH en una configuración de proxy corporativo simulada
   
Si desea probar cómo se comportaría RHDH si se implementara en un entorno de proxy corporativo,
   puede ejecutar `podman-compose` o `docker-compose` fusionando los archivos [`compose.yaml`](./compose.yaml) y [`compose-with-corporate-proxy.yaml`](./compose-with-corporate-proxy.yaml).
   
Ejemplo con `podman-compose` (tenga en cuenta que el orden de los archivos YAML es importante):

```sh
podman-compose \
   -f compose.yaml \
   -f compose-with-corporate-proxy.yaml \
   up -d
```

El archivo [`compose-with-corporate-proxy.yaml`](compose-with-corporate-proxy.yaml) incluye un contenedor de proxy específico basado en [Squid](https://www.squid-cache.org/) así como una red aislada, de modo que:

1. Sólo el contenedor proxy tiene acceso al exterior.
2. Todos los contenedores que forman parte de la red interna necesitan comunicarse a través del contenedor proxy para llegar al exterior. Esto se puede hacer con las variables de entorno `HTTP(S)_PROXY` y `NO_PROXY`.

## Limpieza
   
Para restablecer RHDH Local, puede utilizar el siguiente comando. Esto limpiará todos los volúmenes adjuntos al contenedor, pero cualquier cambio de configuración que haya realizado en sus archivos YAML `rhdh-local` permanecerá.

```sh
podman-compose down --volumes
```

Para restablecer todo en el repositorio `rhdh-local` clonado, incluidos los cambios de configuración realizados en sus archivos YAML, intente:

```sh
git reset --hard
```

Para eliminar completamente los contenedores RHDH de su sistema (después de ejecutar `compose down`):

```sh
docker system prune --volumes # For rhdh-local running on docker
podman system prune --volumes # For rhdh-local running on podman
```

### Problemas conocidos al usar Podman Compose
    
Funciona con `podman-compose` solo con imágenes que incluyen la siguiente corrección: https://github.com/redhat-developer/rhdh/pull/1585
    
Las imágenes más antiguas no funcionan en combinación con `podman-compose`.
Esto se debe a https://issues.redhat.com/browse/RHIDP-3939. Actualmente, las imágenes RHDH llenan el directorio dynamic-plugins-root con todos los complementos empaquetados dentro de la imagen.
Antes de que podman monte el volumen en el directorio `dynamic-plugins-root`, copia todos los archivos existentes en el volumen. Cuando se instalan complementos utilizando el script `install-dynamic-plugins.sh`, se crean instalaciones duplicadas de algunos complementos, lo que impide que Backstage se inicie.
    
Esto tampoco funciona con `podman compose` cuando se usa `docker-compose` como proveedor de composición externo en macOS.
    
Falló con

```
install-dynamic-plugins-1  | Traceback (most recent call last):
install-dynamic-plugins-1  |   File "/opt/app-root/src/install-dynamic-plugins.py", line 429, in <module>
install-dynamic-plugins-1  |     main()
install-dynamic-plugins-1  |   File "/opt/app-root/src/install-dynamic-plugins.py", line 206, in main
install-dynamic-plugins-1  |     with open(dynamicPluginsFile, 'r') as file:
install-dynamic-plugins-1  | PermissionError: [Errno 13] Permission denied: 'dynamic-plugins.yaml'
```

Parece que `docker-compose` cuando se usa con podman no propaga correctamente la etiqueta `Z` de SElinux.

## Uso de la base de datos PostgreSQL
   
> **ADVERTENCIA**: De forma predeterminada, se utiliza la base de datos en memoria.
Si desea utilizar PostgreSQL con RHDH, estos son los pasos:
   
> **NOTA**: Debe tener [Inicio de sesión de Red Hat](https://access.redhat.com/RegistryAuthentication#getting-a-red-hat-login-2) para usar la imagen `postgresql`.
   
1. Inicie sesión en el registro de contenedores con las credenciales de inicio de sesión de Red Hat para usar la imagen `postgresql`

   ```sh
   podman login registry.redhat.io
   ```

   Si prefieres `docker`, simplemente reemplaza `podman` con `docker`

   ```sh
   docker login registry.redhat.io
   ```

2. Descomente el bloque de servicio `db` en el archivo [compose.yaml](compose.yaml)

   ```yaml
   db:
     image: "registry.redhat.io/rhel8/postgresql-16:latest"
     volumes:
       - "/var/lib/pgsql/data"
     env_file:
       - path: "./default.env"
         required: true
       - path: "./.env"
         required: false
     environment:
       - POSTGRESQL_ADMIN_PASSWORD=${POSTGRES_PASSWORD}
     healthcheck:
       test: ["CMD", "pg_isready", "-U", "postgres"]
       interval: 5s
       timeout: 5s
       retries: 5
   ```

3. Descomente la sección `db` en la sección `depends_on` del servicio `rhdh` en [compose.yaml](compose.yaml)

   ```yaml
   depends_on:
     install-dynamic-plugins:
       condition: service_completed_successfully
     db:
       condition: service_healthy
   ```

4. Comente la configuración de SQLite en memoria en [`app-config.local.yaml`](configs/app-config.local.yaml)

   ```yaml
   # database:
   #   client: better-sqlite3
   #   connection: ':memory:'
   ```

## Desarrolladores: uso de VSCode para depurar complementos de backend
   
Puede utilizar RHDH-local con un depurador para depurar sus complementos de backend en VSCode. Aquí te explicamos cómo:
   
1. Inicie RHDH-local con el archivo de composición "depuración". 
   
   ```sh
   # en el directorio rhdh-local
   podman-compose up -f compose.yaml -f compose-debug.yaml
   ```
   
2. Abra el código fuente de su complemento en VSCode
3. Exportar el complemento a un complemento RHDH "dinámico"
   
   ```sh
   # en el directorio del código fuente del complemento
   npx @janus-idp/cli@último paquete export-dynamic-plugin
   ```
   
4. Copie el paquete de complemento derivado exportado al directorio `dynamic-plugins-root` en el contenedor `rhdh`. 
   
   ```sh
   # en el directorio del código fuente del complemento
   podman cp dist-dynamic rhdh:/opt/app-root/src/dynamic-plugins-root/<nombre-de-su-complemento>
   ```
   
5. Si su complemento requiere configuración, agréguelo al archivo `app-config.local.yaml` en el directorio `rhdh-local` clonado.

6. Reinicie el contenedor `rhdh`

   ```sh
   # in rhdh-local directory
   podman-compose stop rhdh
   podman-compose start rhdh
   ```

7. Configure el depurador de VSCode para adjuntarlo al contenedor `rhdh`.

   `.vscode/launch.json` example:

   ```json
   {
      "version": "0.2.0",
      "configurations": [
         {
            "name": "Attach to Process",
            "type": "node",
            "request": "attach",
            "port": 9229,
            "localRoot": "${workspaceFolder}",
            "remoteRoot": "/opt/app-root/src/dynamic-plugins-root/<your-plugin-name>",
         }
      ]
   }
   ```

8. Ahora, puedes comenzar a depurar tu complemento usando el depurador de VSCode. El mapeo de origen debería funcionar y usted debería poder insertar puntos de interrupción en sus archivos TypeScript. Si eso no funciona, probablemente necesites ajustar las rutas `localRoot` y `remoteRoot` en `launch.json`.
   
   Siempre que realice cambios en el código fuente de su complemento, repita los pasos 3 a 6.

## Configuración de credenciales de registro

Coloque sus credenciales de registro en `./configs/extra-files` y luego haga referencia al archivo de autenticación en `.env`:

```bash
REGISTRY_AUTH_FILE=/opt/app-root/src/configs/extra-files/auth.json
```

Esto permite que RHDH-local extraiga artefactos OCI de registros como registry.redhat.io sin errores de autenticación.

## Contribución y notificación de problemas

Para informar problemas en este repositorio, use [JIRA](https://issues.redhat.com/browse/RHIDP) con el componente: **RHDH Local**

Para explorar los problemas existentes, puede utilizar esto [Query](https://issues.redhat.com/issues/?filter=-4&jql=project%20%3D%20%22Red%20Hat%20Internal%20Developer%20Platform%22%20%20AND%20component%20%3D%20%22RHDH%20Local%22%20AND%20resolution%20%3D%20Unresolved%20ORDER%20BY%20status%2C%20priority%2C%20updated%20%20%20%20DESC).

¡Las contribuciones son bienvenidas!

## Licencia
   
``txt
Derechos de autor de Red Hat

Con licencia Apache, versión 2.0 (la "Licencia");
No puedes utilizar este archivo excepto de acuerdo con la Licencia.
Puede obtener una copia de la Licencia en

http://www.apache.org/licenses/LICENCIA-2.0

A menos que lo exija la ley aplicable o se acuerde por escrito, el software
Distribuido bajo la Licencia se distribuye "TAL CUAL",
SIN GARANTÍAS NI CONDICIONES DE NINGÚN TIPO, ya sean expresas o implícitas.
Consulte la Licencia para conocer el texto específico que rige los permisos y
limitaciones bajo la Licencia.
```
