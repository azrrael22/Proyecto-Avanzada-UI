# Dockerización del Frontend - Proyecto Avanzada UI

Este documento explica cómo usar la dockerización del frontend de la aplicación.

## Requisitos Previos

- Docker instalado (versión 20.10 o superior)
- Docker Compose instalado (versión 1.29 o superior)

## Archivos Docker

### 1. `Dockerfile`
Construcción multi-stage que:
- Compila la aplicación Angular en modo producción
- Sirve la aplicación estática con Nginx
- Tamaño final de imagen: ~25MB

### 2. `docker-compose.yml`
Orquestación del contenedor frontend:
- **Puerto**: 4200 (host) → 80 (contenedor)
- **Red**: `app-network` (compatible con backend)
- **Auto-restart**: Configurado

### 3. `nginx.conf`
Servidor web optimizado con:
- Soporte para rutas SPA de Angular
- Compresión Gzip
- Cache de assets estáticos
- Headers de seguridad

### 4. `.dockerignore`
Optimización del contexto de build

## Comandos Principales

### Construir y ejecutar el contenedor
```bash
docker-compose up --build
```

### Ejecutar en modo detached (background)
```bash
docker-compose up -d
```

### Ver logs del contenedor
```bash
docker-compose logs -f
```

### Detener el contenedor
```bash
docker-compose down
```

### Reconstruir sin cache
```bash
docker-compose build --no-cache
docker-compose up -d
```

## Acceso a la Aplicación

Una vez ejecutado el contenedor, acceder a:
```
http://localhost:4200
```

## Integración con Backend

El frontend está configurado para conectarse al backend en:
```
http://localhost:8080
```

### Si tu backend está dockerizado:

1. **Asegúrate de que el backend exponga el puerto 8080**

2. **Si ambos contenedores están en la misma red Docker:**
   - El frontend puede acceder al backend usando el nombre del servicio
   - Ejemplo: `http://nombre-servicio-backend:8080`

3. **Para conectar ambas redes Docker:**

   En el `docker-compose.yml` de tu backend, agrega:
   ```yaml
   networks:
     app-network:
       external: true
   ```

   Luego crea la red externa:
   ```bash
   docker network create app-network
   ```

   Y ejecuta ambos contenedores:
   ```bash
   # En el directorio del backend
   docker-compose up -d

   # En el directorio del frontend
   docker-compose up -d
   ```

### Configuración de CORS en el Backend

Asegúrate de que tu backend permita requests desde:
- `http://localhost:4200` (desarrollo local)
- El origen correspondiente si usas un dominio

## Variables de Entorno

El frontend usa las siguientes URLs de API (configuradas en `src/environments/`):

- **Development**: `http://localhost:8080`
- **Production**: `http://localhost:8080` (modificar según necesidad)

Para cambiar la URL del API en producción, modifica:
- `src/environments/environment.ts`

Luego reconstruye la imagen:
```bash
docker-compose build --no-cache
```

## Troubleshooting

### Error de conexión con el backend
```bash
# Verificar que el backend esté corriendo
curl http://localhost:8080/api/public/statistics

# Verificar logs del frontend
docker-compose logs -f

# Verificar que ambos contenedores estén en la misma red
docker network inspect app-network
```

### Puerto 4200 ya en uso
```bash
# Modificar el puerto en docker-compose.yml
ports:
  - "8080:80"  # Cambia 4200 por otro puerto disponible
```

### Problemas de build
```bash
# Limpiar todo y reconstruir
docker-compose down
docker system prune -a
docker-compose up --build
```

## Arquitectura de la Aplicación

### Build Multi-Stage

**Stage 1: Build**
- Base: `node:20-alpine`
- Instala dependencias con `npm ci`
- Compila Angular en modo producción
- Output: `/app/dist/proyecto-avanzada-ui/browser`

**Stage 2: Production**
- Base: `nginx:alpine`
- Copia build desde Stage 1
- Configura Nginx para SPA
- Tamaño final: ~25MB

### Optimizaciones Aplicadas

1. **Multi-stage build**: Reduce tamaño de imagen
2. **Alpine Linux**: Imágenes base ligeras
3. **Gzip compression**: Reduce tamaño de transferencia
4. **Cache de assets**: Mejora performance
5. **Security headers**: Protección básica

## Mantenimiento

### Actualizar dependencias
```bash
# Modificar package.json
# Reconstruir imagen
docker-compose build --no-cache
docker-compose up -d
```

### Ver uso de recursos
```bash
docker stats proyecto-avanzada-frontend
```

### Inspeccionar contenedor
```bash
docker exec -it proyecto-avanzada-frontend sh
```

## Notas Importantes

- El contenedor sirve la aplicación en modo **producción**
- Los cambios en el código requieren **reconstruir la imagen**
- Para desarrollo local, usa `ng serve` directamente
- El build de producción está optimizado (minificado, tree-shaking, etc.)

## Soporte

Para reportar problemas o solicitar mejoras:
- Revisa los logs: `docker-compose logs -f`
- Verifica la configuración de red
- Asegúrate de que el backend esté accesible
