---
name: clonar-repo
description: Clona un repositorio Git, detecta el tipo de proyecto, instala dependencias y configura el entorno automáticamente
argument-hint: <repo-url> [carpeta-destino]
allowed-tools: [Bash, Read, Glob, Write]
---

# Clonar Repo

Provisioning automático de repositorios. Clona, instala dependencias y deja todo listo para trabajar.

## Argumentos

El usuario invocó este comando con: $ARGUMENTS

- **Primer argumento** (requerido): URL del repositorio Git
- **Segundo argumento** (opcional): carpeta destino. Si no se indica, usar `~/projects/{nombre_repo}`

## Instrucciones

Seguí estos pasos en orden:

### 1. Parsear la URL y definir destino

Extraé el nombre del repo de la URL (sin `.git`). Si el usuario indicó una carpeta destino, usá esa. Si no, usá `~/projects/{nombre_repo}`.

### 2. Clonar el repositorio

```bash
git clone <url> <destino>
```

Si falla, informá el error y detenete.

### 3. Detectar tipo de proyecto e instalar dependencias

Revisá qué archivos existen en la raíz del proyecto clonado y ejecutá el instalador correspondiente:

| Archivo detectado | Comando a ejecutar |
|---|---|
| `pnpm-lock.yaml` | `pnpm install` |
| `yarn.lock` | `yarn install` |
| `bun.lockb` o `bun.lock` | `bun install` |
| `package.json` (sin lockfile anterior) | `npm install` |
| `requirements.txt` | `pip install -r requirements.txt` |
| `Pipfile` | `pipenv install` |
| `pyproject.toml` con `[tool.poetry]` | `poetry install` |
| `pyproject.toml` sin Poetry | `pip install -e .` |
| `Gemfile` | `bundle install` |
| `go.mod` | `go mod download` |
| `Cargo.toml` | `cargo build` |
| `composer.json` | `composer install` |

Si hay múltiples gestores (ej: `package.json` + `requirements.txt`), instalá todos. Priorizá el lockfile específico sobre el genérico.

### 4. Configurar entorno

Si existe `.env.example` o `.env.sample`, copialo a `.env`:

```bash
cp .env.example .env
```

### 5. Reportar resumen

Al finalizar, mostrá un resumen claro:

- Repositorio clonado
- Ubicación en disco
- Tipo de proyecto detectado
- Dependencias instaladas (comando usado)
- Si se creó `.env`
- Cualquier advertencia o error
