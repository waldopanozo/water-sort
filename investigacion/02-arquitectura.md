# 02 — Arquitectura

## Vista general

Arquitectura en capas estilo **clean-ish / feature-first**, con estado vía **Riverpod** (`StateNotifier`).

```
lib/
├── main.dart                 # bootstrap Hive + ProviderScope
├── domain/                   # modelos + use cases (sin I/O)
│   ├── models/               # Tube, GameLevel, UserProgress, UserProfile
│   └── use_cases/            # LevelGenerator, LevelSolver
├── data/                     # persistencia
│   ├── services/             # HiveService + adapters TypeAdapter
│   └── repositories/         # ProgressRepository (fachada)
└── ui/
    ├── providers.dart        # wiring Riverpod
    ├── core/                 # theme, widgets compartidos
    └── features/
        ├── home/             # menú, settings, perfiles
        ├── game/             # partida (ViewModel + Flame + overlay Flutter)
        ├── level_select/
        └── how_to_play/
```

## Stack

| Capa | Tecnología | Rol |
|------|------------|-----|
| UI widgets | Flutter Material | pantallas, diálogos, navegación `Navigator` |
| Motor visual del tablero | **Flame** (`FlameGame`) | tubos, pour animado, partículas |
| Audio | **flame_audio** | SFX |
| Estado | **flutter_riverpod** | providers + StateNotifiers |
| Persistencia | **Hive / hive_flutter** | progreso, settings, estado de partida |
| Tipografía | BebasNeue | branding UI |

## Patrones

1. **Repository:** `ProgressRepository` encapsula Hive y cachea progreso/perfil activo.
2. **Use cases:** generación y solver de niveles separados de la UI.
3. **ViewModel:** `GameViewModel` / `HomeViewModel` concentran reglas de sesión.
4. **Override en bootstrap:** `hiveServiceProvider` se inyecta desde `main()` tras `init()`.

## Modelo de dominio clave

### `Tube`

- `List<Color> colors` (índice 0 = fondo, last = tope)
- `capacity` (4 por defecto; 5/6 en niveles especiales)
- Helpers: `isFull`, `isEmpty`, `isSolved`, `canReceive`

### `GameLevel`

- Número de nivel, tubos, `optimalMoves`, `totalMoves`
- `isComplete` si todos los tubos están resueltos o vacíos

### Progreso

- `UserProgress`: nivel actual, máximo completado, total de movimientos
- Estrellas por nivel en settings Hive (`*_level_stars`)
- Estado de partida activa (`*_saved_level_state`) para reanudar

## Navegación

Sin router tipado: `Navigator.push` / `pop` / `pushReplacement` entre views. Simple, pero sin deep links ni rutas nombradas.

## Persistencia (Hive boxes)

| Box | Contenido |
|-----|-----------|
| `user_progress` | `UserProgress` por perfil (`progress_$id`) |
| `user_profiles` | perfiles |
| `game_settings` | flags, tema, estrellas, partida guardada, perfil activo |

Settings se namespacian por `profileId` (ej. `${profileId}_timer_enabled`).

## Acoplamientos notables

- **Domain usa `Color` de Flutter** → el dominio no es puro Dart; dificulta tests headless y reutilización.
- **`AppColors` es estado global estático** (`setTheme`) → temas no fluyen solo por Riverpod/InheritedWidget.
- **Lógica de pour duplicada** en generator, solver, view model y Flame game.
- **`LevelGenerator` importa `app_colors.dart` (UI)** → domain depende de UI.

## Plataformas

- Código Android presente; carpeta `ios/` no visible en el repo (foco Android / F-Droid).
- `.gitignore` ignora `/test/` y `/linux/` → tests y desktop no forman parte del flujo habitual.
- Cómo correr en local (Flutter + emulador): [06-setup-local.md](./06-setup-local.md).
- Curva de dificultad por nivel: [05-escalado-dificultad.md](./05-escalado-dificultad.md).
