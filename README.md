# Static Docker - Servidor Nginx para Aplicaciones Estáticas/SPA

## 📋 Descripción

Este proyecto es un servidor web **Nginx optimizado** dentro de un contenedor Docker que sirve aplicaciones web estáticas y Single Page Applications (SPAs). Está configurado específicamente para:

- Servir contenido estático de forma eficiente
- Soportar Progressive Web Apps (PWAs) con Service Workers
- Implementar políticas de caché inteligentes
- Manejar routing de SPAs (React, Vue, Angular, etc.)
- Trabajar con dominios HTTPS mediante nginx-proxy y Let's Encrypt

## 🏗️ Estructura del Proyecto

```
staticDocker/
├── docker-compose.yml      # Configuración del contenedor Docker
├── nginx/
│   ├── nginx.conf         # Configuración principal de Nginx
│   └── conf.d/
│       └── server.conf    # Configuración del servidor virtual
├── public/               # Directorio para archivos estáticos a servir
└── README.md            # Este archivo
```

## 🚀 Características Principales

### 1. **Optimizaciones de Rendimiento**
- Compresión GZIP activada para archivos de texto, JavaScript, CSS e imágenes
- Configuración de caché eficiente según tipo de archivo:
  - **Service Worker** (`sw.js`): Sin caché para detectar actualizaciones
  - **Archivos HTML/Manifest**: Sin caché para obtener versiones actualizadas
  - **Archivos estáticos** (JS/CSS con hash): Caché ilimitado (1 año)

### 2. **Soporte para aplicaciones SPA**
- Regla `try_files` que redirige todas las rutas no existentes a `index.html`
- Permite que frameworks como React Router, Vue Router, etc. controlen el routing

### 3. **Configuración PWA**
- Servidor optimizado para Progressive Web Apps
- Soporte completo para manifests y Service Workers

### 4. **Seguridad**
- Se oculta la versión de Nginx (`server_tokens off`)
- Límite de tamaño de cuerpo de solicitud configurado (default: 1MB)

## 📦 Requisitos

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- Una red externa de Docker llamada `wetrust` (si se ejecuta con docker-compose)

## 🔧 Instalación

### 1. Clonar o descargar el proyecto

```bash
git clone <tu-repositorio> staticDocker
cd staticDocker
```

### 2. Preparar los archivos estáticos

Coloca tus archivos estáticos (HTML, CSS, JavaScript, imágenes, etc.) en la carpeta `public/`:

```bash
# Ejemplo: si tienes un build de React
cp -r mi-app/build/* public/
```

### 3. Configurar el dominio (Opcional)

Si deseas usar un dominio específico, edita `docker-compose.yml`:

```yaml
environment:
  VIRTUAL_HOST: "tu-dominio.com"
  LETSENCRYPT_HOST: "tu-dominio.com"
  LETSENCRYPT_EMAIL: "tu-email@dominio.com"
```

Si NO deseas usar nginx-proxy con Let's Encrypt, puedes remover estas variables.

### 4. Crear la red Docker (si no existe)

```bash
docker network create wetrust
```

O si deseas ejecutar sin red externa, comenta la sección `networks` en `docker-compose.yml`.

## ▶️ Ejecutar el Proyecto

### Con Docker Compose (Recomendado)

```bash
docker-compose up -d
```

Esto iniciará el contenedor de Nginx en segundo plano.

### Detener el contenedor

```bash
docker-compose down
```

### Ver logs

```bash
docker-compose logs -f nginx
```

## 🌐 Acceder a la Aplicación

Depende de tu configuración:

- **Localmente**: `http://localhost`
- **Con dominio**: `https://tu-dominio.com` (si está configurado con Let's Encrypt)

## ⚙️ Personalización

### Cambiar el puerto

En `docker-compose.yml`, agrega una sección `ports`:

```yaml
nginx:
  ports:
    - "8080:80"
```

Luego accede en `http://localhost:8080`

### Aumentar límite de tamaño de archivo

En `nginx/nginx.conf`, modifica:

```properties
client_max_body_size 50M;  # Cambiar a tamaño deseado
```

### Agregar más tipos de archivo para compresión

En `nginx/nginx.conf`, en la sección `gzip_types`, agrega los tipos MIME:

```properties
gzip_types text/plain application/javascript text/css application/json image/png image/svg+xml;
```

### Uso con nginx-proxy (docker-gen) y Let's Encrypt

Si estás usando `jwilder/nginx-proxy` con `jrcs/letsencrypt-nginx-proxy-companion`:

1. Asegúrate de que la red `wetrust` existe
2. Las variables `VIRTUAL_HOST`, `LETSENCRYPT_HOST` y `LETSENCRYPT_EMAIL` en `docker-compose.yml` se configurarán automáticamente
3. Los certificados SSL se generarán automáticamente

## 📝 Notas Importantes

- **Caché Service Worker**: El archivo `sw.js` se sirve con `Cache-Control: no-cache` para forzar la revalidación. Esto es crítico para que los cambios se detecten automáticamente.
- **Routing SPA**: La configuración actual redirige todas las rutas no encontradas a `index.html`, lo que permite que tu SPA maneje el routing.
- **Archivo public/**: Debe contener el resultado compilado de tu aplicación (build output).

## 🐛 Solución de Problemas

### El contenedor no inicia

```bash
docker-compose logs nginx
```

Revisa los logs para ver el error específico.

### La aplicación SPA muestra 404 en rutas específicas

Asegúrate de que:
1. Todos los archivos estén en la carpeta `public/`
2. La configuración de routing en `server.conf` está correcta (no ha sido modificada)

### Los archivos no se actualizan

Elimina la caché del navegador (`Ctrl+Shift+Del` o `Cmd+Shift+Del`) y recarga la página.

## 📄 Licencia

Este proyecto está licenciado bajo los términos de la **GNU General Public License v3.0** (GPL-3.0).
Consulta el archivo [LICENSE](LICENSE) para más detalles.

---

**¿Necesitas ayuda?** Revisa la configuración de Nginx o los logs del contenedor Docker.
