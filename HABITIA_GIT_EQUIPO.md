# HABITIA — Plan de Ramas Git y Commits por Integrante

> Documento derivado de **`Habitia_Backend_Division_Equipo.docx`** (el documento real del profesor).
> Repositorios: `C:\ProyectoAppMoviles\HABITIA-Back` (backend, 14 ramas creadas) y
> `C:\ProyectoAppMoviles\Habitia` (frontend, rama `dev/pruebas` existente — ver §5).

---

## 1. Regla de oro del equipo

```
main  (siempre estable, solo merge de ramas dev revisadas)
 └── dev/<integrante>-rf<NN>-<tarea>   ← UNA rama por requerimiento funcional
        └── commits atómicos, uno por paso (design / development / testing)
```

**Formato del commit:** `RF-NN <tipo>: <qué se hizo>`
- Ejemplo: `RF-03 feat: crear TaskController con CRUD y filtros`
- Ejemplo: `RF-04 test: calcularStreak con días consecutivos`

**Frecuencia:** 1 commit por Task terminada (no mezclar diseño con código con tests en un solo commit).

---

## 2. Mapa de responsables (según el documento real)

| Integrante | Frente | RFs |
|---|---|---|
| **Mauricio** | Autenticación · Seguridad JWT · IA · OpenDroid | RF-01, RF-12, RF-14 |
| **Andrea** | Usuario · Tareas · Hábitos · Recordatorios | RF-02, RF-03, RF-04, RF-05 |
| **Mayler** | Comidas · Lista de compras · Calendario · Enfoque · Media | RF-06, RF-07, RF-08, RF-09 |
| **Sergio** | Resumen diario · Analítica · Sincronización · Búsqueda · Gamificación | RF-10, RF-11, RF-13 |

> Nota: RF-14 (OpenDroid) figura como Mauricio en la tabla de RF del documento;
> el frente "Sergio/Mauricio" también lo menciona. Queda asignado a Mauricio
> para no duplicar dueño (confirmar con el equipo si Sergio colabora).

---

## 3. Las 14 ramas dev (ya creadas en el backend)

| # | Rama | RF | Tarea |
|---|---|---|---|
| 1 | `dev/mauricio-rf01-auth` | RF-01 | Autenticación: registro, login, logout, JWT |
| 2 | `dev/mauricio-rf12-ia` | RF-12 | Recomendaciones IA (Haabi / Gemini) |
| 3 | `dev/mauricio-rf14-opendroid` | RF-14 | Integración OpenDroid (comandos de voz) |
| 4 | `dev/andrea-rf02-usuario` | RF-02 | Gestión de usuarios: perfil y preferencias |
| 5 | `dev/andrea-rf03-tareas` | RF-03 | CRUD de tareas |
| 6 | `dev/andrea-rf04-habitos` | RF-04 | CRUD de hábitos con rachas |
| 7 | `dev/andrea-rf05-recordatorios` | RF-05 | CRUD de recordatorios |
| 8 | `dev/mayler-rf06-comidas` | RF-06 | CRUD de comidas |
| 9 | `dev/mayler-rf07-compras` | RF-07 | CRUD de lista de compras |
| 10 | `dev/mayler-rf08-calendario` | RF-08 | CRUD de eventos de calendario |
| 11 | `dev/mayler-rf09-enfoque` | RF-09 | Sesiones de enfoque |
| 12 | `dev/sergio-rf10-resumen` | RF-10 | Resumen diario |
| 13 | `dev/sergio-rf11-analitica` | RF-11 | Analítica de datos |
| 14 | `dev/sergio-rf13-sync` | RF-13 | Sincronización entre dispositivos |

---

## 4. Plan de commits por integrante (en "tiempos" distintos)

### 🟣 MAURICIO — `dev/mauricio-rf01-auth`
```
Tiempo 1 (Design)
  RF-01 docs: modelo User y esquema de tabla users con email único
Tiempo 2 (Development)
  RF-01 feat: UserEntity, UserRepository.findByEmail y DTOs Register/AuthResponse
  RF-01 feat: SecurityConfig con BCryptPasswordEncoder y JwtService HS256
  RF-01 feat: AuthController con POST /auth/login y /auth/register
Tiempo 3 (Testing)
  RF-01 test: login válido, credenciales inválidas y token expirado
Tiempo 4 (Hardening - ya hecho en el proyecto)
  RF-01 fix: @JsonIgnore en password y entry point 401 para Postman
```
`dev/mauricio-rf12-ia`
```
  RF-12 feat: Gemini 2.5-flash + haabiTools con 7 herramientas
  RF-12 feat: HaabiActionsImpl puente con repos reales
  RF-12 feat: event bus + notificación al crear
```
`dev/mauricio-rf14-opendroid`
```
  RF-14 docs: decisión de NO importar OpenDroid (GPLv3) — solo referencia
  RF-14 feat: contrato HaabiActions listo para comandos de voz (pendiente)
```

### 🟢 ANDREA — `dev/andrea-rf02-usuario`
```
  RF-02 docs: UserPreferences y DashboardConfig
  RF-02 feat: UserController GET/PUT /Users/{id} y /email/{email}
  RF-02 feat: perfil real (nombre, cumpleaños, teléfono) con @JsonIgnore
  RF-02 test: validación de pertenencia al usuario (RN-01)
```
`dev/andrea-rf03-tareas`
```
  RF-03 docs: modelo Task, Priority (LOW/MEDIUM/HIGH) y TaskStatus
  RF-03 feat: TaskController CRUD + /Tasks/today + /status/{status}
  RF-03 feat: GlobalExceptionHandler
  RF-03 test: aislamiento por usuario y fechas
```
`dev/andrea-rf04-habitos`
```
  RF-04 docs: Habit + HabitCompletion (Habit_complet)
  RF-04 feat: HabitController CRUD y HabitCompletionController toggle
  RF-04 feat: computeStreak (rachas consecutivas hasta hoy)
  RF-04 test: rachas diarias/semanales y recálculo
```
`dev/andrea-rf05-recordatorios`
```
  RF-05 docs: modelo Reminder con RepeatType
  RF-05 feat: ReminderController CRUD + /user/{id}/between
  RF-05 feat: getUpcomingReminder ordenado por fecha+hora
  RF-05 test: próximo recordatorio y completado
```

### 🟠 MAYLER — `dev/mayler-rf06-comidas`
```
  RF-06 docs: Meals + MealIngredient
  RF-06 feat: MealsController CRUD + /user/{id}/date + /between
  RF-06 feat: MealIngredientController y persistencia de ingredientes
  RF-06 test: comida con ingredientes por día y tipo
```
`dev/mayler-rf07-compras`
```
  RF-07 docs: ShoppingList + ShoppingItem (item_date, source_meal_type)
  RF-07 feat: /ShoppingLists/generate y CRUD de items
  RF-07 feat: InterpreteDeIngredientes tolerante (sin cantidad = 1 UNIT)
  RF-07 test: rechazo de cantidades <= 0 (RN-04)
```
`dev/mayler-rf08-calendario`
```
  RF-08 docs: CalendarEvent
  RF-08 feat: CalendarEventController CRUD + /between
  RF-08 feat: evento agendado por Haabi (09:00-10:00)
  RF-08 test: rango de fechas
```
`dev/mayler-rf09-enfoque`
```
  RF-09 docs: FocusSession y validación RN-07 (sesión única activa)
  RF-09 feat: pantalla Focus local (pendiente backend /focus/*)
  RF-09 test: pendiente
```

### 🔵 SERGIO — `dev/sergio-rf10-resumen`
```
  RF-10 docs: DailySummaryDTO de agregación
  RF-10 feat: home con progreso derivado (actualmente en HomeViewModel)
  RF-10 feat: /summary/daily (pendiente en backend)
  RF-10 test: agregación multi-dominio (pendiente)
```
`dev/sergio-rf11-analitica`
```
  RF-11 docs: AnalyticsMetrics, TrendData y BalanceScore
  RF-11 feat: consultas SQL/JPQL de agregación (pendiente)
  RF-11 test: tendencias (pendiente)
```
`dev/sergio-rf13-sync`
```
  RF-13 docs: SyncStatus y DTOs push/pull (pendiente)
  RF-13 feat: sincronización entre dispositivos (pendiente)
```

---

## 5. Frontend (KMP) — rama única compartida

Hoy existe `dev/pruebas` en `C:\ProyectoAppMoviles\Habitia`. Si se quiere el mismo
esquema por integrante en el frontend, crear:

```
dev/front-mauricio-auth
dev/front-andrea-task-habit
dev/front-mayler-food-shopping
dev/front-sergio-summary
```

Comandos (en el repo del frontend):
```powershell
cd C:\ProyectoAppMoviles\Habitia
git branch dev/front-mauricio-auth
git branch dev/front-andrea-task-habit
git branch dev/front-mayler-food-shopping
git branch dev/front-sergio-summary
```

---

## 6. Comandos de trabajo diario

```powershell
# Cambiar a tu rama
git checkout dev/andrea-rf03-tareas

# Hacer cambio y commitear (1 commit por Task)
git add .
git commit -m "RF-03 feat: TaskController con CRUD y filtros"

# Subir tu rama
git push -u origin dev/andrea-rf03-tareas

# Al terminar: merge a main (después de revisión)
git checkout main
git merge dev/andrea-rf03-tareas
```

---

## 7. Checklist antes de hacer merge

- [ ] La rama compila (`mvnw clean compile` / `gradlew assembleDebug`)
- [ ] Los endpoints responden en Postman (colección de 68 requests)
- [ ] Sin `FATAL` en `adb logcat` (si toca frontend)
- [ ] Commit con mensaje `RF-NN <tipo>: ...`
- [ ] Deploy del día documentado en `CAMBIOS.md`

---

*Generado desde `Habitia_Backend_Division_Equipo.docx` + el estado real de los repositorios. Noviembre 2026.*
