# Módulo de Preguntas · App Aprueba

Módulo de práctica de Aprueba, la app de preparación para la PAES de la empresa Alloxentric. Permite al estudiante recibir preguntas de las pruebas que eligió, responderlas contra reloj, ver su resultado con la explicación paso a paso y la habilidad evaluada, administrar su cuota diaria y pedir la revisión de una pregunta que considere errónea. Es una app Flutter que corre en navegador, Android e iOS, con un backend en FastAPI sobre Firestore.

## Tecnologías y prerrequisitos

- Lenguaje y framework de la app: Flutter 3.22 o superior, con Dart 3.3 o superior. Usa Riverpod, GoRouter, Dio y Drift.
- Lenguaje y framework del backend: Python 3.12 con FastAPI y Pydantic v2, y `uv` como gestor de paquetes.
- Base de datos: Firestore. En local se usa el emulador de Firestore, que requiere Java 21.
- Autenticación: Firebase Authentication con correo y contraseña.
- Otras dependencias: Node.js, para ejecutar la CLI de Firebase con `npx`; Google Chrome; y Docker Desktop, opcional, para levantar el backend con contenedores.

## Instalación

Los comandos están probados en Windows PowerShell. En macOS o Linux se usan `npx` y `curl` en lugar de `npx.cmd` y `curl.exe`.

```powershell
git clone https://github.com/Jere4k1080/aprueba_app_modulo_preguntas.git
cd aprueba_app_modulo_preguntas

# Dependencias de la app
flutter pub get
dart run build_runner build --delete-conflicting-outputs

# Dependencias del backend
cd backend
uv venv --python 3.12
uv pip install -r requirements.txt -r requirements-dev.txt
cd ..
```

`build_runner` es obligatorio: genera el código de la caché local, y sin él la app no compila.

## Configuración

El backend lee sus variables desde `backend/.env`, que no se versiona. Se crea a partir del ejemplo:

```powershell
cp backend/.env.example backend/.env
```

| Variable | Valor en local | Para qué sirve |
|---|---|---|
| `APP_ENV` | `local` | Entorno de ejecución |
| `FIRESTORE_EMULATOR_HOST` | `127.0.0.1:8085` | Conecta el backend al emulador de Firestore |
| `FIRESTORE_EMULATOR_PROJECT_ID` | `demo-aprueba` | Proyecto del emulador; el prefijo `demo-` impide tocar datos reales |
| `FIREBASE_PROJECT_ID` | `aprueba-app-modulo-preguntas` | Proyecto de Firebase contra el que se validan los inicios de sesión |
| `ALLOWED_ORIGINS` | `http://localhost:5050` | Origen de la app web que el backend acepta |
| `SEED_DEMO_UID` y `SEED_DEMO_NEW_UID` | UID de las cuentas de demostración | Cargar los datos de prueba con esas cuentas. Los entrega el equipo |

`backend/.env.example` trae además `PORT`, `QUOTA_RESET_HOUR_LOCAL`, `QUOTA_RESET_TIMEZONE`, `ALLOWED_ORIGIN_PATTERN` y `SEED_ALLOW_REMOTE`, que en local se dejan con el valor del ejemplo.

`FIREBASE_SERVICE_ACCOUNT_BASE64` solo se usa en el despliegue y nunca se escribe en el repositorio ni en el archivo local.

La app recibe la dirección del backend al ejecutarse, con `--dart-define=API_BASE_URL`.

## Uso

En local se necesitan tres terminales, desde la raíz del repositorio:

```powershell
# Terminal 1: emulador de Firestore
npx.cmd firebase-tools emulators:start --only firestore --project demo-aprueba

# Terminal 2: datos de prueba y API en el puerto 4000
cd backend
uv run python -m app.seed
uv run uvicorn app.main:app --port 4000

# Terminal 3: app web en el puerto 5050
flutter run -d chrome --web-port=5050 --dart-define=API_BASE_URL=http://localhost:4000/api/v1
```

Con Docker, `docker compose up --build` levanta el emulador y la API juntos, y la app se ejecuta igual que en la terminal 3. El emulador del contenedor parte vacío: los datos de prueba se cargan desde otra terminal, con los UID de `backend/.env`.

```powershell
docker compose run --rm api python -m app.seed
```

Ejemplo mínimo: la API responde su estado con la estructura común de todas sus respuestas.

```powershell
curl.exe http://localhost:4000/api/v1/health
```

```json
{ "data": { "status": "ok", "service": "aprueba-questions-backend", "version": "1.0.0", "uptimeSeconds": 5, "environment": "local" }, "error": null, "meta": { "requestId": "req_...", "timestamp": "..." } }
```

En la app, se inicia sesión con una cuenta de demostración, se elige una materia y se responde una pregunta para ver el resultado. Las credenciales las entrega el equipo.

Para correr las pruebas:

```powershell
flutter test
cd backend
uv run pytest
```

Las pruebas del backend que necesitan Firestore se omiten si el emulador no está encendido.

La versión publicada está en https://aprueba-app-modulo-preguntas.vercel.app, y el estado de su API en https://aprueba-app-modulo-preguntas-api.vercel.app/api/v1/health.

## Enfoque de trabajo en equipo

Extreme Programming (XP), con iteraciones semanales, pruebas escritas antes que el código y revisión de cada cambio por otro integrante · Product Backlog y tablero Kanban en Trello, y solicitudes de cambio en GitHub · Tres entregas funcionales, cada una aceptada en producción al cerrarla.

## Estructura del proyecto

```
lib/                 app Flutter
  features/practice/ módulo de preguntas
test/                pruebas de la app
backend/app/         API en FastAPI
backend/tests/       pruebas del backend
docs/                bitácora de decisiones y diccionario de datos
compose.yaml         contenedores para el entorno local
firestore.rules      reglas de seguridad de Firestore
```

## Autores

- Martin Alonso Espinoza Morales · Encargado de seguimiento y de pruebas
- Jeremías Danilo Fernández Millacura · Programador backend
- Sebastián Roberto Acevedo Araya · Programador frontend

Sección: PTY4614-002D · Docente guía: Mallén González · elia.gonzalezg@profesor.duoc.cl (contacto alternativo: mallen@mallen.cl)

Empresa contraparte: Alloxentric, mediante la Incubadora Duoc UC San Bernardo.

## Licencia

No aplica. El código base pertenece a la empresa contraparte y el repositorio no se distribuye bajo una licencia de código abierto.
