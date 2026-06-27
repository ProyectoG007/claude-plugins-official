---
name: levantar
description: Levanta el entorno de desarrollo completo del proyecto actual. Detecta el stack, inicia servicios (DB, backend, frontend) y reporta estado.
argument-hint: [todo|backend|frontend|db] [--clean]
allowed-tools: [Bash, Read, Glob, Write]
---

# Levantar Entorno de Desarrollo

Levantá todo tu entorno de desarrollo con un solo comando.

## Argumentos

El usuario invocó este comando con: $ARGUMENTS

- **Sin argumentos o `todo`**: Levanta DB + backend + frontend
- **`backend`**: Solo el backend
- **`frontend`**: Solo el frontend
- **`db`**: Solo la base de datos
- **`--clean`**: Limpia caches/builds antes de levantar

## Instrucciones

### 1. Detectar el stack del proyecto

Buscá en la raíz del proyecto estos archivos para identificar cada componente:

**Base de datos:**
- `docker-compose.yml` o `docker-compose.yaml` → buscar servicios `postgres`, `mysql`, `redis`, `mongo`
- `compose.yml` o `compose.yaml` → mismo check

**Backend:**
- `go.mod` → Go (usar Air si `.air.toml` existe, sino `go run`)
- `package.json` en raíz o `backend/` → Node.js
- `requirements.txt` o `pyproject.toml` → Python
- `Cargo.toml` → Rust
- `Gemfile` → Ruby

**Frontend:**
- `pubspec.yaml` en raíz o `frontend/` → Flutter
- `package.json` con scripts `dev`/`start` en `frontend/` → Node frontend (React, Vue, etc.)

### 2. Verificar qué ya está corriendo

Antes de levantar, verificá si los servicios ya están activos:

```bash
# Docker
docker ps 2>/dev/null | grep -i postgres
# Puertos comunes
lsof -i :8080 2>/dev/null  # backend Go
lsof -i :3000 2>/dev/null  # frontend
lsof -i :5432 2>/dev/null  # PostgreSQL
```

Si algo ya está corriendo, informá y no lo dupliques.

### 3. Levantar servicios en orden

**Orden recomendado: DB → Backend → Frontend**

#### Base de datos (si hay docker-compose):
```bash
docker-compose up -d postgres  # o el servicio que corresponda
```
Esperá a que esté healthy antes de continuar.

#### Backend Go (con Air para hot reload):
```bash
cd backend  # o la carpeta que corresponda
export PATH="/opt/flutter/bin:/root/go/bin:$PATH"
air &
```
Si no hay Air configurado pero existe `go.mod`:
```bash
go run cmd/server/main.go &  # o el entry point que detectes
```

#### Backend Node.js:
```bash
npm run dev &  # o yarn dev, pnpm dev
```

#### Frontend Flutter:
```bash
cd frontend  # o la carpeta que corresponda
export PATH="/opt/flutter/bin:$PATH"
export CHROME_EXECUTABLE=/usr/bin/chromium-browser
flutter run -d chrome &
```

#### Frontend Node (React/Vue/Next):
```bash
cd frontend
npm run dev &
```

### 4. Si se pidió `--clean`

Antes de levantar, limpiar:
```bash
# Go
cd backend && go clean -cache
# Flutter
cd frontend && flutter clean && flutter pub get
# Node
rm -rf node_modules && npm install
# Docker
docker-compose down -v && docker-compose up -d
```

### 5. Verificar que todo está corriendo

Después de levantar, verificá:
```bash
# Checkear puertos activos
lsof -i :8080 -i :3000 -i :5432 2>/dev/null
# Checkear procesos
ps aux | grep -E "(air|flutter|node|go)" | grep -v grep
```

### 6. Reportar resumen

Mostrá una tabla con el estado de cada servicio:

| Servicio | Estado | Puerto | Comando |
|----------|--------|--------|---------|
| PostgreSQL | ✅ Running | :5432 | docker-compose |
| Backend (Go) | ✅ Running | :8080 | air |
| Frontend (Flutter) | ✅ Running | :3000 | flutter run -d chrome |

Incluí:
- URLs de acceso (ej: `http://localhost:3000`)
- Si se creó/copió algún `.env`
- Advertencias o errores encontrados
- Sugerencia: "Usá `/levantar` de nuevo para ver el estado actual"

### 7. Generar dev.sh si no existe

Si el proyecto no tiene un `dev.sh`, ofrecé crearlo basándote en el stack detectado. Usá el template en `scripts/dev.sh.template` como base.
