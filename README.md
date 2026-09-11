# Módulo de Preguntas — App Aprueba

> Núcleo de práctica de la aplicación móvil de **Aprueba**, plataforma de preparación para exámenes de admisión universitaria (PAES).

Proyecto de Título (Capstone) · Grupo 9 · Duoc UC San Bernardo · 2026

![Flutter](https://img.shields.io/badge/Flutter-3.22%2B-02569B?logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-3.3%2B-0175C2?logo=dart&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-Express-339933?logo=nodedotjs&logoColor=white)
![Firestore](https://img.shields.io/badge/Firestore-Firebase-FFCA28?logo=firebase&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![Metodología](https://img.shields.io/badge/Metodolog%C3%ADa-Extreme%20Programming-1A365D)

---

## Tabla de contenidos

- [Descripción](#descripción)
- [Estado del proyecto](#estado-del-proyecto)
- [Alcance](#alcance)
- [Tecnologías](#tecnologías)
- [Arquitectura](#arquitectura)
- [Instalación y ejecución](#instalación-y-ejecución)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Metodología de trabajo](#metodología-de-trabajo)
- [Convenciones](#convenciones)
- [Equipo](#equipo)

---

## Descripción

Aprueba es una plataforma freemium de práctica intensiva para estudiantes que rinden exámenes estandarizados de admisión universitaria, con mercado inicial en Chile y expansión proyectada al Reino Unido. Su modelo entrega un flujo diario de preguntas desde un banco central, con desbloqueo progresivo a cambio de datos del estudiante y suscripciones de bajo costo.

Este repositorio contiene el **módulo de preguntas**: el ciclo funcional donde el estudiante recibe una pregunta, la responde contra reloj, conoce su resultado comparado con la cohorte, accede a la explicación paso a paso y a la habilidad evaluada, gestiona su cuota diaria y puede solicitar la recorrección de una pregunta que considere errónea.

Es el núcleo de valor del producto: sin él, la aplicación no cumple su propósito.

---

## Estado del proyecto

| | |
|---|---|
| Fase actual | Fase 2 — Desarrollo (semanas 5 a 15) |
| Iteración en curso | Iteración 1 · Habilitación del entorno |
| Próxima entrega funcional | Entrega 1, al cierre de la iteración 3 |

La Fase 1 cerró con la definición del proyecto documentada: enunciado de alcance, visión del producto, lista de historias de usuario, estructura de desglose del trabajo, carta Gantt, acta de constitución y documento de requerimientos.

---

## Alcance

### Incluido

**Seis pantallas** en el cliente móvil:

| Pantalla | Función |
|---|---|
| Pregunta | Enunciado y alternativas, cronómetro, descuento de cuota |
| Resultado | Acierto o error, alternativa correcta, explicación breve, percentil de velocidad, medalla obtenida |
| Explicación detallada | Planteamiento, pasos intermedios, verificación y concepto clave |
| Habilidad | Habilidad evaluada, nivel de dominio, árbol de prerrequisitos y recursos recomendados |
| Recorrección | Solicitud de revisión con motivo tipificado y comentario |
| Desbloqueo de cuota | Ampliación de la cuota diaria declarando colegio o región |

**Trece servicios** en el backend:

```
Práctica       GET  /practice/next
               GET  /questions/{id}
               POST /questions/{id}/answer
               GET  /questions/{id}/explanation
               GET  /questions/{id}/skill

Cuota          GET  /me/quota
               POST /me/quota/unlock

Recorrección   POST /corrections
               GET  /corrections

Configuración  GET  /tests
               GET  /me/preferences
               PUT  /me/preferences
               GET  /me/progress
```

Además: la lógica de cuota diaria escalonada y la economía de recompensas, el modelo de datos del módulo con su caché local sin conexión, la integración del manejo de sesión y renovación de credenciales, las pruebas unitarias y de integración, y el empaquetado en contenedores con despliegue reproducible.

### No incluido

Registro y autenticación, onboarding, pantalla de inicio, sistema de medallas y canjes, regalos y beneficios, grupos de estudio, comunidad, marketplace de tutores, muro de pago y suscripciones, ajustes y notificaciones.

Tampoco la consola de administración, el generador de preguntas, el sitio web del alumno ni la landing de marketing.

La pasarela de pago, el inicio de sesión social y las notificaciones push se consumen pero no se intervienen. La generación y curaduría del banco de preguntas es insumo de la empresa contraparte.

---

## Tecnologías

### Cliente móvil

| Componente | Tecnología | Rol |
|---|---|---|
| Framework | Flutter 3.22+ / Dart 3.3+ | Un solo código para iOS y Android |
| Estado | Riverpod | Gestión de estado reactiva y testeable |
| Navegación | GoRouter | Enrutamiento declarativo |
| Red | Dio | Cliente HTTP con interceptor de renovación de tokens |
| Persistencia local | Drift (SQLite) | Caché para repaso sin conexión |
| Almacenamiento seguro | flutter_secure_storage | Custodia del refresh token |

### Backend

| Componente | Tecnología | Rol |
|---|---|---|
| Runtime | Node.js | Entorno de ejecución |
| Framework | Express | API REST bajo `/api/v1` |
| Base de datos | Firestore (Firebase Admin SDK) | Persistencia, con emulador para desarrollo local |
| Autenticación | JWT | Access token de 15 min + refresh de 30 días con rotación |
| Empaquetado | Docker y archivo de composición | Levantamiento reproducible del entorno |

### Justificación de las decisiones principales

- **Flutter** permite mantener un único código fuente para ambas plataformas móviles, algo determinante para un equipo de tres personas con plazo acotado.
- **Riverpod** ofrece inyección de dependencias y estado testeable sin acoplar la lógica al árbol de widgets, lo que facilita escribir la prueba antes que el código.
- **Drift** habilita el repaso sin conexión, requisito funcional del producto, con consultas tipadas y verificadas en tiempo de compilación.
- **Firestore** es la base de datos ya adoptada por el ecosistema; mantenerla evita divergencias en el modelo de datos.

---

## Arquitectura

```mermaid
flowchart TD
    subgraph client["App Flutter - iOS y Android"]
        UI["Pantallas<br/>features/practice/"]
        PROV["Providers<br/>Riverpod"]
        REPO["Repositorios"]
        CACHE[("Drift · SQLite<br/>caché offline")]
        API["ApiClient · Dio"]
    end

    subgraph server["Backend"]
        EXPRESS["Node.js + Express<br/>/api/v1"]
        SDK["Firebase Admin SDK"]
    end

    DB[("Firestore")]

    UI --> PROV
    PROV --> REPO
    REPO --> API
    REPO <--> CACHE
    API -->|"HTTPS · JWT Bearer"| EXPRESS
    EXPRESS --> SDK
    SDK --> DB
```

### Flujo de datos

La interfaz nunca conversa directamente con la red. Toda petición atraviesa la cadena `UI → provider → repositorio → ApiClient`. En cada lectura el repositorio escribe el resultado en la caché local y, ante un fallo de red, intenta servir la última copia disponible.

### Manejo de sesión

El access token viaja en la cabecera `Authorization`. Ante una respuesta 401, el interceptor de Dio renueva las credenciales con el refresh token, que rota en cada uso, y reintenta la petición original de forma transparente. Si la renovación falla, la sesión se marca como cerrada y el router redirige al inicio.

### Contrato de respuestas

Todas las respuestas de la API comparten una estructura común:

```json
{
  "data":  { },
  "error": null,
  "meta":  { "requestId": "...", "timestamp": "..." }
}
```

En caso de error, `data` es `null` y `error` contiene `code` (identificador estable, legible por máquina), `message` (texto listo para mostrar), `details` y `field`.

Dos decisiones de diseño atraviesan todo el módulo:

- **Los códigos de error de negocio se traducen a estados de interfaz, no a mensajes genéricos.** Alcanzar la cuota base conduce a la pantalla de desbloqueo; no produce un error.
- **La respuesta correcta no viaja al cliente antes de que el estudiante responda.** Ni `GET /practice/next` ni `GET /questions/{id}` la incluyen. Es una regla de integridad del producto.

---

## Instalación y ejecución

### Requisitos previos

- Flutter 3.22 o superior (Dart 3.3+)
- Android Studio con el SDK de Android, o Xcode para iOS
- Node.js y npm
- Docker, o bien Java si se prefiere levantar el emulador de Firestore de forma directa

### Cliente móvil

```bash
cd aprueba_app

# Genera las carpetas nativas android/ e ios/
flutter create --org cl.aprueba --project-name aprueba_app .

flutter pub get

# Obligatorio: genera database.g.dart, requerido por Drift
dart run build_runner build --delete-conflicting-outputs

flutter run --dart-define=API_BASE_URL=http://127.0.0.1:4000/api/v1
```

> Sin ejecutar `build_runner` el proyecto no compila, porque la capa de persistencia local depende de código generado.

### Backend

```bash
cd backend
npm install
cp .env.example .env
npm run emulator   # emulador de Firestore, en otra terminal
npm run seed       # datos de prueba
npm start          # API en http://127.0.0.1:4000/api/v1
```

### Configuración

No hay credenciales en el código. Todos los valores sensibles se inyectan mediante `--dart-define` en el cliente y variables de entorno en el backend.

### Verificación

```bash
flutter analyze                    # análisis estático
flutter test                       # pruebas unitarias
python3 tool/check_static.py .     # verificación auxiliar sin SDK de Flutter
python3 tool/check_models.py       # contrasta los modelos con respuestas reales
```

---

## Estructura del repositorio

```
.
├── aprueba_app/                 Cliente móvil Flutter
│   ├── lib/
│   │   ├── core/                tema, i18n, router, red, storage, configuración
│   │   ├── data/
│   │   │   ├── local/           persistencia Drift
│   │   │   ├── models/          modelo lógico de la API
│   │   │   └── repositories/    un repositorio por dominio
│   │   ├── features/
│   │   │   └── practice/        ← módulo de preguntas
│   │   └── providers/           wiring de Riverpod
│   ├── test/                    pruebas unitarias
│   └── tool/                    verificadores auxiliares
│
├── backend/                     API REST
│   └── src/
│       ├── routes/              definición de endpoints
│       ├── services/            lógica de negocio
│       └── seed/                datos de prueba
│
└── docs/                        documentación del proyecto
```

---

## Metodología de trabajo

El proyecto aplica **Extreme Programming** como metodología única.

La elección responde a un diagnóstico y no a una preferencia. El problema del equipo no era de organización sino técnico: ninguno de los tres había escrito Dart antes de comenzar. Extreme Programming prescribe las prácticas de ingeniería que ese problema exige, mientras que un marco de gestión por sí solo las deja a criterio del equipo. La coordinación de tres personas de la misma sección, con horario común, cabe en un tablero.

### Iteraciones

Diez iteraciones semanales, agrupadas en tres entregas funcionales.

| Entrega | Iteraciones | Semanas | Contenido |
|---|---|---|---|
| 1 | 1 a 3 | 5 a 7 | Entorno operativo, modelo de datos y la primera pantalla funcionando de punta a punta |
| 2 | 4 a 6 | 8 a 10 | El ciclo completo de aprendizaje: responder, entender por qué y conocer la habilidad evaluada |
| 3 | 7 a 10 | 11 a 14 | Cuota diaria, recorrección, funcionamiento sin conexión, pruebas de integración y contenedores |

Al inicio de cada iteración se eligen las historias, se estiman y se dividen en tareas. Al cierre se mide el trabajo completado, y ese dato sirve para planificar la iteración siguiente.

### Prácticas adoptadas

| Práctica | Aplicación |
|---|---|
| Programación en pares | Las tres primeras iteraciones se trabajan íntegramente en pares; después se mantiene para la lógica de cuota, el interceptor de sesión y la caché |
| Prueba antes que el código | Las reglas de cuota, recompensas y recorrección son verificables, así que se fijan como pruebas antes de implementarse |
| Integración continua | Se incorpora a la rama principal a diario, no al final de la semana |
| Refactorización | Mejora permanente del diseño existente, sin etapa separada |
| Diseño simple | La solución más sencilla que funcione y pase las pruebas |
| Propiedad colectiva | Cualquiera puede modificar cualquier archivo |
| Estándares de código | Analizador estático del proyecto y convenciones de Dart y de Node |
| Entregas pequeñas | Tres entregas funcionales durante el semestre, no una sola al final |
| Ritmo sostenible | El proyecto convive con el resto de las asignaturas |
| Reunión de pie | Corta, para sincronizar y detectar bloqueos |
| Juego de planificación | Al inicio de cada iteración |
| Metáfora | El módulo se entiende como el ciclo de práctica del estudiante: recibe una pregunta, responde, aprende del resultado y, si detecta un error, lo reclama |

### Desviaciones declaradas

**Cliente en sitio.** Extreme Programming contempla la presencia permanente del cliente durante el desarrollo. La contraparte no participa del trabajo diario, de modo que la práctica no se cumple. La mitigación tiene tres partes: un integrante asume el rol de cliente para las decisiones diarias, interpretando el contrato de servicios y las reglas documentadas sin inventar; cada decisión tomada sin confirmación queda registrada con su fuente y su impacto potencial; y las entregas funcionales se validan con la contraparte, aunque de forma espaciada.

**Reunión de pie diaria.** Se adapta a una sincronización semanal breve, porque los tres integrantes no comparten jornada completa.

Ambas se declaran por transparencia metodológica. Declarar la desviación vale más que sostener que se aplica la metodología completa.

---

## Convenciones

### Criterios de terminado

Una historia está terminada cuando sus pruebas de aceptación pasan. En concreto:

- Las pruebas unitarias están escritas y en verde
- `flutter analyze` no arroja advertencias
- El código fue revisado por al menos otro integrante
- Se respeta el contrato de respuestas y el catálogo de errores
- Los errores de negocio se traducen a estados de interfaz comprensibles
- La respuesta correcta no se expone antes de que el estudiante responda
- No hay credenciales ni secretos versionados

### Control de versiones

- Rama principal: `main`
- Ramas de trabajo: `feature/<descripción-breve>`
- Integración a `main` a diario, con revisión previa de al menos un integrante

---

## Equipo

| Integrante | Rol en XP | Responsabilidad principal |
|---|---|---|
| Martin Alonso Espinoza Morales | Coach y Tracker | Guía las prácticas, mide la velocidad, coordina con la empresa y ejerce como cliente delegado |
| Jeremías Danilo Fernández Millacura | Programador | Servicios de backend y modelo de datos |
| Sebastián Roberto Acevedo Araya | Programador | Cliente móvil y capa de presentación |

Los tres escriben sus propias pruebas: con un equipo de este tamaño, Extreme Programming no separa el rol de tester. La propiedad colectiva del código implica que estos focos indican responsabilidad principal, no exclusividad.

**Profesora guía:** Eliana Mallen González
**Empresa contraparte:** Alloxentric, vinculada mediante la Incubadora Duoc UC San Bernardo

---

## Contexto académico

Este repositorio corresponde al Proyecto de Título de la asignatura **PTY4614 Capstone**, sección CAPSTONE_002D, de la carrera de Ingeniería en Informática de **Duoc UC, sede San Bernardo**, durante el segundo semestre de 2026.

El desarrollo se organiza en tres fases: definición del proyecto, desarrollo del producto y presentación ante la comisión evaluadora.
