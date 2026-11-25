# 🌿 HABITIA

> **Tu hogar en sintonía.** Aplicación multiplataforma (Android + iOS) de hábitos,
> tareas, bienestar y asistente IA, conectada a un backend Spring Boot con
> arquitectura hexagonal y MySQL.

---

## 📌 Tabla de contenidos

1. [Visión general](#1-visión-general)
2. [Arquitectura completa](#2-arquitectura-completa)
3. [Base de datos y esquemas](#3-base-de-datos-y-esquemas)
4. [Módulos del backend](#4-módulos-del-backend)
5. [Frontend Kotlin Multiplatform](#5-frontend-kotlin-multiplatform)
6. [El asistente Haabi (IA)](#6-el-asistente-haabi-ia)
7. [Estrategia Git y aportes del equipo](#7-estrategia-git-y-aportes-del-equipo)
8. [Guía de ejecución](#8-guía-de-ejecución)
9. [Estado real y pendientes](#9-estado-real-y-pendientes)

---

## 1. Visión general

Habitia nace como una app de productividad personal y bienestar. El usuario puede:

- Registrarse / iniciar sesión (JWT de 24 h, sesión persistida en DataStore).
- Gestionar **tareas** con prioridades, categorías editables, matriz Eisenhower
  y tablero Kanban con drag & drop.
- Seguir **hábitos** con rachas calculadas desde completions reales.
- Planificar **comidas** con ingredientes detallados y **lista de compras**
  agrupada por día y comida.
- Usar un **calendario**, **recordatorios**, un **diario** y un temporizador
  de enfoque.
- Hablar con **Haabi**, una mascota IA que crea todo lo anterior automáticamente
  y lo agenda con alerta.

### Stack

| Capa | Tecnología |
|---|---|
| Frontend | Kotlin Multiplatform + Compose Multiplatform + Material3 |
| DI | Koin (`appModule`) + `SettingProvider` expect/actual |
| Red | Ktor (OkHttp en Android, Darwin en iOS) |
| Backend | Spring Boot 4 + Java 21 + arquitectura hexagonal |
| BD | MySQL 8 (`habitia`), Hibernate `ddl-auto=update` |
| IA | Gemini 2.5 Flash (function calling) |
| Glass | Haze 1.2.x (patrón de 2 capas hermanas) |

---

## 2. Arquitectura completa

### 2.1 Backend hexagonal (103 archivos Java)

```
HABITIA_Back/
├── HabitiaBackApplication.java        ← arranque
├── auth/          (7)  ← JWT, filtros, SecurityConfig
├── shared/        (11) ← User + 9 enums + UserRepository
└── modules/       (85) ← 9 módulos de negocio
    ├── task      (10)   ├── habit    (15)
    ├── reminder  (8)    ├── meal     (17)
    ├── shopping  (14)   ├── calendar (5)
    ├── diary     (7)    ├── user     (6)
    └── catalog   (3)
```

Cada módulo sigue **la misma forma**:

```
modules/<modulo>/
├── domain/model/              ← entidades (tablas)
├── domain/service/            ← lógica pura (ej. InterpreteDeIngredientes)
├── application/service/       ← Service +ServiceImpl (@Transactional)
├── application/dto/           ← Request / Response
├── infrastructure/Controller/ ← endpoints REST
└── infrastructure/persistence/← repositorios JPA
```

### 2.2 Flujo de una petición

```
App Android (Ktor, Bearer JWT)
   → JwtAuthenticationFilter valida (401 si falta)
   → Controller (DTO de entrada)
   → Service (reglas de negocio)
   → Entity de dominio
   → JpaRepository → SQL → MySQL
   → vuelta por el mismo camino → 200 JSON
```

### 2.3 Frontend Clean Architecture (147 archivos Kotlin)

```
commonMain/kotlin/upeu/edu/pe/
├── Core/          (26) ← tema, vidrio (Glass), 16 componentes, navegación, config
├── Data/          (36) ← ApiClient, repos, DTOs, mappers, Gemini, notificaciones
├── Domain/        (32) ← modelos, interfaces de repos, 7 use cases
├── Presentation/  (32) ← 15 pantallas + ViewModels
└── di/            (2)  ← Modules (Koin) + SettingProvider
```

**Regla de dependencias:** `Presentation → Domain ← Data`. Presentation y Data
se ponen de acuerdo en las **interfaces de Domain**; nunca se conocen entre sí.

### 2.4 Los 4 "pegamentos"

| Pegamento | Función |
|---|---|
| `ApiClient` | HttpClient Ktor, IP auto-detectada (4 redes), Gemini y tools |
| `SessionManager` | JWT + perfil en DataStore (sobrevive al cierre) |
| `HaabiEventBus` | Avisa cuando la IA crea algo → refresh + pestaña + notificación |
| `Modules (Koin)` | Entrega repositorios y Settings a los ViewModels |

---

## 3. Base de datos y esquemas

**Motor:** MySQL, base `habitia`, puerto 3306. `spring.jpa.hibernate.ddl-auto=update`.

| Tabla | Campos principales | Módulo dueño |
|---|---|---|
| `Users` | id, name, email (único), password (BCrypt, `@JsonIgnore`), birthDate, phone | user / shared |
| `Tasks` | id, title, description, dueDate, priority, status, id_user | task |
| `Task_Categories` | id, name, id_user | task / catalog |
| `Habits` | id, name, description, frequency, startDate, endDate, active, id_user | habit |
| `Habit_complet` | id, date, completed, id_habit | habit (rachas) |
| `Reminders` | id, title, description, reminderDate, repeat, completed, id_user | reminder |
| `Meals` | id, name, type, date, time, description, completed, id_user | meal |
| `Meal_Ingredients` | id, description, quantity, unit, id_meal | meal |
| `Shopping_Lists` | id, startDate, endDate, status, id_user | shopping |
| `Shopping_Items` | id, description, quantity, unit, itemDate, sourceMealType, sourceMealName, source, completed, id_list | shopping |
| `Calendar_Events` | id, title, description, startDate, endDate, location, id_user | calendar |
| `DiaryEntries` | id, entryDate, text (≤4000), id_user | diary |
| `Task_Categories` / `Catalog_Ingredients` | categorías editables / 48 ingredientes sembrados | catalog |

### Relaciones clave

```
Users ──┬── Tasks ─── Task_Categories
        ├── Habits ── Habit_complet
        ├── Reminders
        ├── Meals ─── Meal_Ingredients
        ├── Shopping_Lists ── Shopping_Items
        ├── Calendar_Events
        └── DiaryEntries
```

**Dato curioso:** `InterpreteDeIngredientes` convierte texto libre ("200 g de arroz")
en cantidad + unidad + nombre, y tolera ingredientes sin cantidad (asume 1 unidad).

---

## 4. Módulos del backend

| Módulo | Endpoints (ejemplo) | Qué hace |
|---|---|---|
| Auth | `POST /auth/register`, `/auth/login` | BCrypt + JWT HS256 24 h |
| User | `GET/PUT /Users/{id}`, `/Users/email/{email}` | Perfil real del JWT |
| Task | `GET /Tasks/user/{id}`, `POST`, `PUT`, `DELETE` | CRUD + categorías |
| Habit | `/Habits`, `/HabitCompletions` | CRUD + rachas |
| Reminder | `/Reminders/user/{id}`, `/between` | Alertas |
| Meal | `/Meals`, `/MealIngredients` | Comidas + ingredientes persistidos |
| Shopping | `/ShoppingLists/generate`, `/items`, `/confirm` | Lista agrupada por día/comida |
| Calendar | `/CalendarEvents`, `/user/{id}/between` | Agenda de Haabi |
| Diary | `/DiaryEntries`, `/date/{date}` | Diario personal |
| Catalog | `/Catalogs/categories`, `/ingredients` | Catálogos editables con seed |

**Total: 68 endpoints** (31 GET, 14 POST, 10 PUT, 12 DELETE, 1 PATCH) —
ver colección Postman incluida.

---

## 5. Frontend Kotlin Multiplatform

### 15 pantallas Compose

| Grupo | Pantallas |
|---|---|
| Acceso | Splash, Welcome, Login, Register |
| Día a día | Home, TaskScreen, EisenhowerScreen, PlannerScreen, RemindersScreen, FocusScreen |
| Bienestar | FoodScreen, DiaryScreen, HaabiScreen |
| Ajustes | MoreScreen, UserMenuScreen, AppearanceScreen, GlassDiagnostic |

### Efecto Liquid Glass

`habitiaGlass()` aplica 6 capas: sombra → clip → click → `hazeEffect(HazeStyle)` →
brillo especular → borde de luz. El truco del **patrón de 2 capas hermanas**
evita que Haze 1.2.x excluya los effects descendientes del source.

### Persistencia

| Dato | Dónde | Persiste al cerrar |
|---|---|---|
| JWT + perfil | DataStore (`SessionManager`) | ✅ |
| Tema/paleta | DataStore (`UserPreferencesRepository`) | ✅ |
| Categorías y catálogo | MySQL (`/Catalogs`) | ✅ |
| Tareas/hábitos/comidas | MySQL + caché del repo | ✅ |
| Posición de Haabi / chats | Memoria | ❌ |

---

## 6. El asistente Haabi (IA)

1. El usuario escribe en `HaabiScreen` o toca la mascota flotante.
2. `ApiClient.haabiChat()` envía prompt + historial + 7 herramientas a
   **Gemini 2.5 Flash**.
3. Gemini devuelve un `functionCall` (ej. `crear_tarea`, `crear_plan_semanal_comidas`).
4. `executeHaabiAction()` → `HaabiActionsImpl` usa los **repositorios reales**:
   guarda en MySQL, agenda evento y recordatorio a las 09:00.
5. `HaabiEventBus.emit()` → `App.kt` refresca, cambia de pestaña y muestra
   notificación local. La UI se actualiza sola vía StateFlow.

Herramientas: `crear_tarea`, `crear_habito`, `crear_recordatorio`, `crear_comida`,
`crear_compra`, `crear_diario`, `crear_plan_semanal_comidas`.

Extras: memoria por sesión, multi-chat, trato configurable (Normal/Casual/Formal/
Motivador), mascota arrastrable y botón X en el chat.

---

## 7. Estrategia Git y aportes del equipo

### Ramas (solo 4, una por integrante)

```
main  (estado real y compilado)
├── dev/mauricio
├── dev/andrea
├── dev/mayler
└── dev/sergio
```

Cada rama tiene **un commit propio con el autor del integrante** (fecha 2026-11-24)
y un archivo `APORTES_<NOMBRE>.md` que detalla su trabajo.

| Integrante | Commit | Aporte real |
|---|---|---|
| **Mauricio** | `f65217c` | RF-01 Auth/JWT + seguridad (401 claro, @JsonIgnore) y RF-12 Haabi (Gemini + tools + EventBus) |
| **Andrea** | `7ff3614` | RF-02 Usuario/perfil, RF-03 Tareas (categorías editables, Eisenhower), RF-04 Hábitos + rachas, RF-05 Recordatorios |
| **Mayler** | `9399177` | RF-06 Comidas + ingredientes persistidos, RF-07 Compras agrupadas, RF-08 Calendario, RF-09 Enfoque local |
| **Sergio** | `8b05598` | RF-10 Resumen, RF-11 Analítica, RF-13 Sync — base documentada + pendientes marcados con honestidad |

> RF-14 (OpenDroid) queda documentado como decisión de NO importar la librería
> (GPLv3); solo se usa como referencia.

### Convención de commits

```
RF-NN <tipo>: <qué se hizo>     (tipo = feat | fix | test | docs | refactor)
```

Ejemplo real: `RF-03 feat: TaskController con CRUD y filtros`.

---

## 8. Guía de ejecución

### Backend
```powershell
cd C:\ProyectoAppMoviles\HABITIA-Back
.\mvnw.cmd spring-boot:run
# esperar: "Started HabitiaBackApplication in ~10s"
```

### Frontend
```powershell
$env:JAVA_HOME = 'C:\Program Files\Java\jdk-21'
cd C:\ProyectoAppMoviles\Habitia
.\gradlew assembleDebug
```

### Postman
1. Importar `Habitia_Postman_Collection.json` + `HABITIA_Postman_Environment.json`.
2. Seleccionar environment **HABITIA - Entorno Local / LAN**.
3. Correr `POST /auth/login` (guarda el JWT automáticamente).

### Descarga del APK
```
https://raw.githubusercontent.com/Reigan-code/Documentation-Habitia/main/Habitia-APK-debug.apk
```
QR en `Habitia-APK-QR.png` (mismo repo).

---

## 9. Estado real y pendientes

### ✅ Implementado y verificado
- Auth JWT + perfil + 401 claro en Postman
- Tareas + categorías editables + Eisenhower/Kanban
- Hábitos con rachas desde completions
- Comidas con ingredientes persistidos + lista de compras por día/alimento
- Diario, Calendario, Recordatorios, Notificaciones locales
- Haabi con function calling, agenda y alerta
- Backend hexagonal + DI Koin + Settings + ZIP limpio

### ⚠️ Parcial / pendiente (documentado, no inventado)
- RF-10/11/13 backend (`/summary`, `/analytics`, `/sync`) — hoy el resumen se
  calcula en el front.
- RF-09 backend `/focus/*` (timer local no más).
- RF-14 OpenDroid (solo referencia por licencia).
- Google Calendar privado (OAuth2), media upload, búsqueda, gamificación.
- Persistencia de posición de Haabi y chats (en memoria).

---

*README generado desde el código real del proyecto y el documento
`Habitia_Backend_Division_Equipo.docx`. Noviembre 2026.*
