Este es un manual directo para dominar `uv`, diseñado para que pases de cero a profesional, o para que migres tu flujo de trabajo desde `pip` + `venv` sin fricción.

---

### ⚙️ 1. Instalación y Configuración Inicial

`uv` es un binario único escrito en Rust, por lo que su instalación no depende de Python ni de Rust.

**Instalación (macOS/Linux):**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Instalación (Windows PowerShell):**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Alternativas:** También puedes instalarlo vía `pipx install uv`, `pip install uv` o `brew install uv`.

**Verificación:**
```bash
uv --version
```

**Autocompletado (recomendado):**
```bash
# Bash
echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc
# Zsh
echo 'eval "$(uv generate-shell-completion zsh)"' >> ~/.zshrc
# Fish
uv generate-shell-completion fish > ~/.config/fish/completions/uv.fish
```


---

### 🧭 2. Los Tres Modos Mentales de `uv`

Entender estos tres modos es clave para no confundirse al usar `uv`. No son excluyentes; se usan según el contexto.

| Modo | Comando Emblemático | Propósito | ¿Cuándo usarlo? |
| :--- | :--- | :--- | :--- |
| **Proyecto (Recomendado)** | `uv init`, `uv add`, `uv sync`, `uv run` | Gestión completa y reproducible de un proyecto con `pyproject.toml` y `uv.lock`. | **Siempre** que empieces un proyecto nuevo o migres uno existente. Es el flujo profesional. |
| **Interfaz `pip`** | `uv pip install`, `uv pip sync` | Reemplazo directo de `pip` para máxima velocidad en flujos existentes. | Cuando solo quieres acelerar un `requirements.txt` sin cambiar tu estructura de archivos. |
| **Herramientas (`uvx`)** | `uvx <tool>` | Ejecutar herramientas CLI (linters, formateadores) en entornos aislados y efímeros. | Para usar una herramienta **sin instalarla** de forma permanente. Equivale a `pipx run`. |

---

### 🚀 3. Flujo de Trabajo Profesional: Gestión de Proyectos

Este es el flujo que usarás el 90% del tiempo. `uv` reemplaza a `venv`, `pip`, `pip-tools`, `pyenv` y `poetry` en un solo conjunto de comandos.

#### 3.1. Crear un Proyecto (`uv init`)

```bash
# Crea un proyecto de aplicación (por defecto). Genera pyproject.toml, .python-version, README y src/
uv init mi-proyecto

# Crea un proyecto de librería (con --lib). Ideal para publicar en PyPI.
uv init --lib mi-libreria
```


`uv init` crea la estructura moderna: código en `src/`, un `pyproject.toml` con sistema de build y un archivo `.python-version` para fijar el intérprete.

#### 3.2. Gestionar la Versión de Python

`uv` puede instalar y gestionar intérpretes de Python por ti.

```bash
# Instala una versión específica de Python (no necesita que la tengas en el sistema)
uv python install 3.12

# Fija la versión de Python para el proyecto actual (crea/actualiza .python-version)
uv python pin 3.12

# Ver versiones instaladas y disponibles
uv python list
```


#### 3.3. Añadir y Eliminar Dependencias (`uv add` / `uv remove`)

**Nunca edites `requirements.txt` a mano en este flujo.** `pyproject.toml` es la única fuente de verdad.

```bash
# Añadir dependencias de producción
uv add requests fastapi "httpx>=0.20"

# Añadir una dependencia de desarrollo (va a [dependency-groups.dev])
uv add --dev pytest ruff mypy

# Añadir a un grupo específico (ej. para tests)
uv add --group test pytest-cov

# Eliminar una dependencia
uv remove requests
uv remove --dev pytest
```


`uv add` resuelve, actualiza `pyproject.toml` y regenera `uv.lock` automáticamente.

#### 3.4. El `uv.lock`: Reproducibilidad Universal

El archivo `uv.lock` es la clave de la reproducibilidad profesional. **Debes commitearlo a Git** junto con `pyproject.toml`.

- **¿Qué contiene?** La versión exacta de **cada dependencia directa y transitiva**, con hashes de verificación.
- **¿Por qué es "universal"?** Un solo `uv.lock` funciona en macOS, Linux y Windows. `uv` resuelve las dependencias para todas las plataformas y usa marcadores de entorno para instalar lo correcto en cada sistema.

**Comandos de mantenimiento del lockfile:**
```bash
# Actualizar todas las dependencias a sus últimas versiones compatibles
uv lock --upgrade

# Actualizar solo un paquete específico
uv lock --upgrade-package requests

# Forzar la regeneración del lockfile (rara vez necesario)
uv lock
```


#### 3.5. Sincronizar el Entorno (`uv sync`)

`uv sync` es el comando que materializa el entorno virtual a partir de `pyproject.toml` y `uv.lock`.

```bash
# Sincroniza el entorno: instala/actualiza/elimina paquetes para que coincidan exactamente con el lockfile
uv sync
```

**En CI/CD o producción, usa `--locked`:** Esto hace que el comando **falle** si `uv.lock` está desactualizado respecto a `pyproject.toml`. Es una salvaguarda crítica para garantizar despliegues reproducibles.
```bash
uv sync --locked
```

#### 3.6. Ejecutar Código (`uv run`)

`uv run` ejecuta un comando **dentro del entorno del proyecto**, asegurándose primero de que el entorno esté sincronizado.

```bash
# Ejecuta tu aplicación o script
uv run python mi_script.py
uv run mi-proyecto  # Si definiste un entry point

# Ejecuta herramientas de desarrollo instaladas en el proyecto
uv run pytest
uv run ruff check .
```
`uv run` es el equivalente a activar el `venv` y ejecutar el comando, pero sin la activación manual.

---

### 🔄 4. Migración desde `pip` + `venv`

Tienes dos estrategias. Elige según tu urgencia.

#### Estrategia A: Rápida (Solo velocidad, sin cambiar flujo)

Usa la **interfaz `pip`** de `uv`. Es un reemplazo directo que no toca tu estructura de archivos.

```bash
# 1. Crea el entorno con uv (en lugar de python -m venv)
uv venv

# 2. Instala desde tu requirements.txt existente (en lugar de pip install -r)
uv pip install -r requirements.txt

# 3. Para generar un requirements.txt bloqueado (en lugar de pip freeze)
uv pip freeze > requirements.txt

# 4. Sincroniza el entorno exactamente con el requirements.txt (instala faltantes, elimina sobrantes)
uv pip sync requirements.txt
```


Esta estrategia te da la velocidad de `uv` de inmediato, pero **no obtienes la reproducibilidad universal de `uv.lock`**.

#### Estrategia B: Completa (Migración al flujo de proyecto)

Esta es la ruta recomendada para adopción profesional a largo plazo.

```bash
# 1. Inicializa un proyecto uv en el directorio actual. Detectará el código existente.
uv init

# 2. Importa tus dependencias desde requirements.txt.
# uv las añadirá a pyproject.toml y generará el uv.lock universal.
uv add -r requirements.txt
```

**Nota:** Si tu `requirements.txt` mezcla dependencias de producción y desarrollo, el panel de expertos recomienda separarlas antes de importar. Puedes crear `requirements-prod.txt` y `requirements-dev.txt`, importar el primero con `uv add -r` y el segundo con `uv add --dev -r`.

Tras la migración, elimina o archiva `requirements.txt`. Tu flujo ahora es `uv add`, `uv sync` y `uv run`.

---

### 🛠️ 5. Herramientas CLI Globales: `uvx` vs. `uv tool install`

Esta es la evolución de `pipx`. La elección depende de si la herramienta debe ser efímera o permanente.

| | `uvx <tool>` (o `uv tool run`) | `uv tool install <tool>` |
| :--- | :--- | :--- |
| **Propósito** | **Ejecución efímera**. Crea un entorno temporal, ejecuta y descarta. | **Instalación persistente**. Crea un entorno aislado permanente y expone el ejecutable en el `PATH`. |
| **Caso de uso** | Probar una herramienta, ejecutar un linter/formateador puntual. | Instalar herramientas que usarás repetidamente (ej. `ruff`, `black`, `httpie`). |
| **Analogía** | Usar una aplicación web. | Instalar una aplicación de escritorio. |
| **Ejemplo** | `uvx ruff check .` | `uv tool install ruff` → luego `ruff check .` |



**Regla general:** La documentación de `uv` recomienda **`uvx` por defecto** para la mayoría de casos. Reserva `uv tool install` para herramientas que necesitas que estén disponibles de forma global para otros scripts o en una imagen Docker.

---

### 📦 6. Construir y Publicar Paquetes (Para Librerías)

`uv` también reemplaza a `build` y `twine`.

```bash
# Construye las distribuciones de código fuente (sdist) y binaria (wheel) en dist/
uv build

# Sube el paquete a PyPI (o a un índice privado)
uv publish
```


---

### ✅ 7. Checklist de Buenas Prácticas Profesionales

1.  **Commitea siempre `pyproject.toml` y `uv.lock`** a Git. Son la fuente de verdad de tu proyecto.
2.  **En CI/CD, usa `uv sync --locked`** para garantizar que las dependencias resueltas coinciden exactamente con el lockfile. Falla el build si no es así.
3.  **En Docker, usa una imagen oficial de `uv`** (ej. `ghcr.io/astral-sh/uv:python3.12-bookworm-slim`) y ejecuta `uv sync --locked` como paso de instalación.
4.  **Nunca actives el entorno virtual manualmente** en el flujo de proyecto. Usa `uv run` para todo.
5.  **Usa `uvx` por defecto** para herramientas CLI. Solo instala (`uv tool install`) cuando necesites persistencia global.
6.  **Para scripts de un solo archivo con dependencias**, usa los metadatos en línea de PEP 723. `uv run` detectará y gestionará las dependencias automáticamente.

---

### 📋 Resumen: Chuleta de Comandos Esenciales

| Tarea | Comando |
| :--- | :--- |
| **Instalar `uv`** | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **Nuevo proyecto** | `uv init mi-app` |
| **Nueva librería** | `uv init --lib mi-lib` |
| **Instalar Python** | `uv python install 3.12` |
| **Fijar versión de Python** | `uv python pin 3.12` |
| **Añadir dependencia** | `uv add requests` |
| **Añadir dependencia de desarrollo** | `uv add --dev pytest` |
| **Eliminar dependencia** | `uv remove requests` |
| **Sincronizar entorno** | `uv sync` |
| **Sincronizar (CI/producción)** | `uv sync --locked` |
| **Actualizar todas las deps** | `uv lock --upgrade` |
| **Actualizar una dep** | `uv lock --upgrade-package requests` |
| **Ejecutar código** | `uv run python script.py` |
| **Ejecutar herramienta (efímera)** | `uvx ruff check .` |
| **Instalar herramienta (permanente)** | `uv tool install ruff` |
| **Ver árbol de dependencias** | `uv tree` |
| **Construir paquete** | `uv build` |
| **Publicar en PyPI** | `uv publish` |

Este es el arsenal completo. `uv` no es solo "pip más rápido"; es un gestor de proyectos integral que, usado con su flujo de proyecto, elimina las inconsistencias y los tiempos de espera de una vez por todas.
