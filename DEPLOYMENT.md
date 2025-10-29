# Guía de Despliegue en Render

Esta guía te ayudará a desplegar SmartPark en Render de forma gratuita.

## 📋 Requisitos Previos

1. Cuenta en [Render.com](https://render.com) (gratis)
2. Repositorio de GitHub con el código del proyecto
3. Google Maps API Key (opcional pero recomendado)

## 🚀 Método 1: Despliegue Automático con render.yaml (Recomendado)

### Paso 1: Preparar el Repositorio

Asegúrate de que todos los archivos necesarios estén en tu repositorio:
- ✅ `backend/requirements.txt`
- ✅ `backend/build.sh`
- ✅ `render.yaml` (en la raíz)
- ✅ `backend/smartpark/settings.py` (actualizado)
- ✅ `frontend/package.json` (con serve)

### Paso 2: Crear el Blueprint en Render

1. Ve a [Render Dashboard](https://dashboard.render.com/)
2. Click en **"New +"** → **"Blueprint"**
3. Conecta tu repositorio de GitHub
4. Render detectará automáticamente el archivo `render.yaml`
5. Click en **"Apply"**

### Paso 3: Configurar Variables de Entorno

Render creará automáticamente los servicios, pero necesitas configurar:

#### Backend (smartpark-backend):
- `ALLOWED_HOSTS`: Agrega tu dominio de Render (ej: `smartpark-backend.onrender.com`)
- `SECRET_KEY`: Se genera automáticamente
- `DEBUG`: Ya está en `false`

#### Frontend (smartpark-frontend):
- `REACT_APP_API_URL`: `https://smartpark-backend.onrender.com/api`
- `REACT_APP_GOOGLE_MAPS_API_KEY`: Tu API key de Google Maps

### Paso 4: Verificar el Despliegue

- Backend: `https://smartpark-backend.onrender.com/api/`
- Frontend: `https://smartpark-frontend.onrender.com/`

---

## 🛠️ Método 2: Despliegue Manual (Paso a Paso)

Si prefieres hacerlo manualmente:

### A. Desplegar el Backend

1. **Crear Web Service:**
   - Dashboard → New + → Web Service
   - Conecta tu repositorio
   - Configuración:
     - **Name**: `smartpark-backend`
     - **Runtime**: Python 3
     - **Build Command**: `./backend/build.sh`
     - **Start Command**: `cd backend && gunicorn smartpark.wsgi:application`

2. **Configurar Variables de Entorno:**
   ```
   PYTHON_VERSION=3.11.0
   SECRET_KEY=[generar uno nuevo]
   DEBUG=false
   ALLOWED_HOSTS=smartpark-backend.onrender.com
   ```

3. **Crear Base de Datos PostgreSQL:**
   - Dashboard → New + → PostgreSQL
   - **Name**: `smartpark-db`
   - Conecta la base de datos al web service

### B. Desplegar el Frontend

1. **Crear Web Service:**
   - Dashboard → New + → Web Service
   - Conecta tu repositorio
   - Configuración:
     - **Name**: `smartpark-frontend`
     - **Runtime**: Node
     - **Build Command**: `cd frontend && npm install && npm run build`
     - **Start Command**: `cd frontend && npx serve -s build -l 3000`

2. **Configurar Variables de Entorno:**
   ```
   NODE_VERSION=18.18.0
   REACT_APP_API_URL=https://smartpark-backend.onrender.com/api
   REACT_APP_GOOGLE_MAPS_API_KEY=[tu_api_key]
   ```

---

## 🔧 Post-Despliegue

### 1. Actualizar CORS en el Backend

Edita `backend/smartpark/settings.py` y actualiza:
```python
CORS_ALLOWED_ORIGINS = [
    "https://smartpark-frontend.onrender.com",
    # Agrega tu dominio personalizado si tienes uno
]
```

### 2. Verificar la Base de Datos

El script `build.sh` automáticamente:
- Ejecuta las migraciones
- Carga los datos iniciales (10 parqueaderos)

Si necesitas repoblar manualmente:
```bash
# En el shell de Render (Backend → Shell)
python manage.py migrate
python add_parkings.py
```

### 3. Configurar Dominio Personalizado (Opcional)

En cada servicio:
- Settings → Custom Domain
- Agrega tu dominio
- Configura DNS según las instrucciones

---

## ⚠️ Consideraciones Importantes

### Plan Gratuito de Render

- ✅ **Pros**:
  - Despliegue automático desde GitHub
  - HTTPS incluido
  - PostgreSQL gratis (90 días)
  
- ⚠️ **Contras**:
  - Los servicios se "duermen" después de 15 minutos de inactividad
  - Primera carga puede tardar ~1 minuto (cold start)
  - 750 horas/mes de uso (suficiente para un proyecto)

### Base de Datos

En el plan gratuito, PostgreSQL se elimina después de 90 días. Opciones:

1. **Usar SQLite en producción** (no recomendado):
   - Elimina `DATABASE_URL` de las variables de entorno
   - El sistema usará SQLite automáticamente

2. **Migrar a otro servicio** después de 90 días:
   - [ElephantSQL](https://www.elephantsql.com/) (plan gratuito permanente)
   - [Supabase](https://supabase.com/) (plan gratuito generoso)

### Google Maps API

- Configura límites de uso en Google Cloud Console
- El plan gratuito incluye $200/mes de crédito
- Monitorea el uso para evitar cargos

---

## 🐛 Solución de Problemas

### Backend no se conecta

1. **Verificar logs:**
   - Dashboard → smartpark-backend → Logs
   
2. **Errores comunes:**
   - `ModuleNotFoundError`: Falta una dependencia en `requirements.txt`
   - `DisallowedHost`: Agrega el dominio a `ALLOWED_HOSTS`
   - Database error: Verifica que la base de datos esté conectada

### Frontend muestra "Modo Offline"

1. Verifica que `REACT_APP_API_URL` esté configurada correctamente
2. Prueba la URL del backend directamente: `https://smartpark-backend.onrender.com/api/`
3. Revisa la consola del navegador (F12) para errores CORS

### Build Fallido

1. **Backend:**
   ```bash
   # Verifica que build.sh tenga permisos de ejecución
   chmod +x backend/build.sh
   ```

2. **Frontend:**
   - Verifica que todas las dependencias estén en `package.json`
   - Asegúrate de que `serve` esté instalado

---

## 📊 Monitoreo

### Logs en Tiempo Real

```bash
# En Render Dashboard
Backend → Logs (streaming en tiempo real)
Frontend → Logs (streaming en tiempo real)
```

### Métricas

Render proporciona:
- CPU usage
- Memory usage
- Request count
- Response times

---

## 🔄 Actualizaciones

El despliegue es **automático**:

1. Haz push a tu rama principal en GitHub
2. Render detecta el cambio automáticamente
3. Ejecuta el build y despliega

Para desactivar el auto-deploy:
- Settings → Auto-Deploy → Disable

---

## 📝 Checklist de Despliegue

Antes de desplegar, verifica:

- [ ] `requirements.txt` actualizado con todas las dependencias
- [ ] `build.sh` con permisos de ejecución
- [ ] Variables de entorno configuradas correctamente
- [ ] Google Maps API Key configurada (opcional)
- [ ] CORS configurado con el dominio correcto
- [ ] Base de datos PostgreSQL creada y conectada
- [ ] `ALLOWED_HOSTS` incluye el dominio de Render

---

## 🆘 Soporte

Si encuentras problemas:

1. Revisa los logs en Render Dashboard
2. Verifica la [documentación oficial de Render](https://render.com/docs)
3. Consulta la [comunidad de Render](https://community.render.com/)

---

## 🎉 ¡Listo!

Tu aplicación debería estar funcionando en:
- **Backend**: `https://smartpark-backend.onrender.com`
- **Frontend**: `https://smartpark-frontend.onrender.com`

Comparte los enlaces y disfruta de tu app en producción! 🚀
