# InnovatechChile - Frontend Web 

Interfaz web desarrollada con Flask y Jinja2 para el proyecto de InnovatechChile. Permite gestionar usuarios consumiendo la API del Backend.

## 📋 Requisitos Previos

- Docker y Docker Compose
- Python 3.11+ (para desarrollo local)
- Navegador web moderno

## 🚀 Inicio Rápido
### Opción 1: Con Docker Compose (Recomendado)
```bash
cd ..
docker-compose up --build
```

La web estará disponible en: `http://localhost:5000`

### Opción 2: Desarrollo Local

```bash
# Crear entorno virtual
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt

# Configurar variables de entorno
cp .env.example .env

# Ejecutar servidor Flask
python app.py
```

## 🐳 Docker

### Construir imagen

```bash
docker build -t innovatech-frontend:latest .
```

### Ejecutar contenedor

```bash
docker run -d \
  --name innovatech-frontend \
  -e BACKEND_URL=http://localhost:3000 \
  -e FLASK_ENV=production \
  -e SECRET_KEY=tu_clave_secreta \
  -p 5000:5000 \
  innovatech-frontend:latest
```

## 📁 Estructura del Dockerfile

El Dockerfile utiliza **multi-stage build** para optimizar:

1. **Stage 1 (Build)**: 
   - Usa `python:3.11-slim` como base
   - Instala dependencias con pip en `--user`

2. **Stage 2 (Runtime)**:
   - Imagen Python slim optimizada
   - Usuario no-root (appuser:1001) por seguridad
   - Health check cada 30 segundos
   - Expone puerto 5000

## 🔐 Variables de Entorno

```env
FLASK_ENV=production
BACKEND_URL=http://backend:3000
SECRET_KEY=clave_secreta_produccion
```

**Nota**: En docker-compose, `BACKEND_URL` debe apuntar a `http://backend:3000` (nombre del servicio)

## 📦 Persistencia de Datos

Los archivos de plantilla (templates) pueden montarse como volúmenes:
- **./templates**: Directorio de plantillas Jinja2
- Permite actualizaciones en caliente sin reiniciar

## 🔄 Pipeline CI/CD

El proyecto incluye un workflow GitHub Actions que:

1. **Build**: Construye la imagen Docker
2. **Push**: Envía a Docker Hub
3. **Deploy**: Descarga e inicia el contenedor en EC2

### Configuración necesaria (GitHub Secrets)

```
DOCKERHUB_USERNAME    → Tu usuario de Docker Hub
DOCKERHUB_TOKEN       → Token de Docker Hub
EC2_HOST              → IP pública de la instancia EC2
EC2_USER              → Usuario SSH (ec2-user o ubuntu)
EC2_SSH_KEY           → Contenido completo de la llave .pem
FLASK_SECRET_KEY      → Clave secreta para Flask
```

## 📝 Rutas de la Aplicación

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/` | Página principal - lista de usuarios |
| GET | `/crear` | Formulario para crear usuario |
| POST | `/crear` | Procesar creación de usuario |
| GET | `/editar/<id>` | Formulario para editar usuario |
| POST | `/editar/<id>` | Procesar edición de usuario |
| GET | `/eliminar/<id>` | Eliminar usuario |

## 📂 Estructura de Carpetas

```
Front_Eval2/
├── Dockerfile
├── requirements.txt
├── app.py                 # Aplicación principal
├── .github/workflows/
│   └── deploy.yml         # Workflow de GitHub Actions
└── templates/
    ├── base.html          # Plantilla base
    ├── index.html         # Listado de usuarios
    ├── crear_usuario.html # Formulario crear
    ├── editar_usuario.html # Formulario editar
    ├── 404.html           # Página error 404
    └── 500.html           # Página error 500
```

## 🧪 Verificar Salud del Contenedor

```bash
docker ps                                  # Ver contenedores activos
docker logs innovatech-frontend            # Ver logs
docker logs -f innovatech-frontend         # Ver logs en tiempo real
docker exec innovatech-frontend python --version  # Verificar Python
```

## 🔄 Actualizar y Redeploy

1. **Hacer cambios en el código/templates**
2. **Commit y push a rama `deploy`**
3. **GitHub Actions dispara automáticamente**
4. **El contenedor se actualiza en EC2**

```bash
git add .
git commit -m "feat: mejorar interfaz de usuarios"
git push origin deploy
```

## 🛠️ Desarrollo Local

```bash
# Activar entorno virtual
source venv/bin/activate  # Windows: venv\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt

# Variables de entorno
export FLASK_ENV=development
export BACKEND_URL=http://localhost:3000
export FLASK_APP=app.py

# Ejecutar servidor (con reload automático)
flask run
```

## 🔌 Comunicación con Backend

El Frontend se comunica con el Backend mediante:

- **Ruta interna (Docker)**: `http://backend:3000` (en docker-compose)
- **Ruta externa (EC2)**: IP pública de EC2 + puerto 3000
- **Método**: Requests HTTP GET/POST/PUT/DELETE

### Ejemplo de integración

```python
import requests

BACKEND_URL = os.getenv('BACKEND_URL', 'http://localhost:3000')

# Obtener usuarios
response = requests.get(f'{BACKEND_URL}/api/usuarios')
usuarios = response.json()
```

## 📚 Buenas Prácticas Implementadas

✅ Multi-stage build para reducir tamaño  
✅ Usuario no-root por seguridad  
✅ Health checks automáticos  
✅ Capas limpias y optimizadas  
✅ Secrets seguros en GitHub  
✅ Red privada entre servicios  
✅ Variables de entorno configurables  
✅ Gestión de errores con templates  

## 🐛 Solucionar Problemas

### El Frontend no ve el Backend
```python
# Verificar BACKEND_URL en producción
print(os.getenv('BACKEND_URL'))

# En docker-compose, debe ser: http://backend:3000
# En EC2, debe ser: http://IP_PUBLICA:3000
```

### Errores en templates
```bash
# Ver logs detallados
docker logs -f innovatech-frontend

# Ejecutar con modo debug (solo desarrollo)
export FLASK_ENV=development
flask run
```

### Puerto 5000 en uso
```bash
# Cambiar puerto en docker-compose.yml
ports:
  - "8080:5000"  # Acceder en localhost:8080
```

## 🔐 Seguridad

- ✅ SECRET_KEY configurado via variables
- ✅ CORS habilitado para comunicación segura
- ✅ Usuario no-root en contenedor
- ✅ Validación de inputs en formularios

## 📄 Licencia

MIT
response = requests.get(f'{BACKEND_URL}/api/usuarios')
usuarios = response.json()

# Ejemplo de petición POST para crear usuario
response = requests.post(f'{BACKEND_URL}/api/usuarios', json=datos_usuario)
```

## Puertos Requeridos

### Para funcionamiento en contenedor:
- **Puerto 5000**: Puerto del servidor frontend Flask (HTTP)
- **Puerto 3000**: Puerto de comunicación con backend API (externo)

### Explicación de puertos:
- **5000**: Es el puerto donde escucha el servidor Flask para servir la aplicación web
- **3000**: Es el puerto del backend API al que el frontend se conecta para obtener/enviar datos

## Variables de Entorno

| Variable | Descripción | Valor por Defecto |
|----------|-------------|-------------------|
| `PORT` | Puerto del servidor Flask | 5000 |
| `DEBUG` | Modo debug (True/False) | False |
| `BACKEND_URL` | URL del backend API | http://localhost:3000 |
| `SECRET_KEY` | Clave secreta para sesiones | clave_secreta_por_defecto |

## Notas Importantes
- El backend API debe estar corriendo antes de iniciar el frontend
- Asegúrate de que las URLs en las variables de entorno sean correctas
- En producción, establece `DEBUG=False` y usa una `SECRET_KEY` segura
- La aplicación está diseñada para funcionar con el backend API de este proyecto
