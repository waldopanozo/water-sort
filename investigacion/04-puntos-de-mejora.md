# 04 — Puntos de mejora

Prioridad orientativa: **P0** = riesgo / calidad base, **P1** = impacto alto, **P2** = nice-to-have.

---

## P0 — Calidad y correctitud

### 1. Sin tests automatizados reales

- `.gitignore` ignora `/test/`.
- CI (`test.yml`) **no ejecuta** `flutter test` ni analyze: solo build de APK.
- Dominio (generator/solver/Tube) es ideal para unit tests sin UI.

**Mejora:** carpeta `test/` versionada; tests de `Tube`, solvability, estrellas, serialización Hive; job CI con `flutter analyze` + `flutter test`.

### 2. `optimalMoves` no es óptimo

La métrica de estrellas se basa en una fórmula heurística, no en el largo real de `LevelSolver`.

**Mejora:** calcular (o cachear) longitud de solución al generar; o renombrar UX a “par” / “meta” para no mentir.

### 3. DFS de generación con techo bajo (5000)

`_isSolvable` puede devolver false por límite aunque el puzzle sea solvable → bucles `while (true)` más largos / CPU al abrir niveles altos.

**Mejora:** BFS / IDA*, límite más alto, o generación por “scramble desde estado resuelto” (garantiza solvabilidad).

### 4. Claves de estado con `Color.hashCode`

En el generator: `c.hashCode` para serializar estados. Hash de Color no es un ID estable de diseño (aunque suele ser estable en la práctica).

**Mejora:** usar `color.value` / índice de paleta explícito.

---

## P1 — Arquitectura y mantenibilidad

### 5. Domain acoplado a Flutter UI

`Tube` / `GameLevel` / generator dependen de `package:flutter/material.dart` (`Color`) y de `AppColors`.

**Mejora:** modelo con `int` o enum `WaterColor`; mapear a `Color` solo en UI.

### 6. Duplicación de lógica de pour

Misma lógica en:

- `LevelGenerator._isSolvable`
- `LevelSolver`
- `GameViewModel.completePendingPour` / `isValidPour`
- `WaterSortGame` (animación)

**Mejora:** un solo `PourRules` / `applyPour(tubes, from, to)` en domain.

### 7. ViewModels / archivos muy grandes

| Archivo | ~LOC | Nota |
|---------|------|------|
| `water_sort_game.dart` | 1134 | motor + animación + partículas |
| `settings_view.dart` | 1134 | UI densa |
| `home_view.dart` | 844 | |
| `game_view.dart` | 740 | |
| `game_view_model.dart` | 685 | estado + timer + save + hints |

**Mejora:** extraer componentes (PourAnimator, TubeLayout, SettingsSections); partir state en notifiers más chicos.

### 8. Indentación / estructura irregular en `GameViewModel`

Tras `completePendingPour` hay métodos con indentación inconsistente (ruido al leer / riesgo de merge conflicts).

**Mejora:** formatear con `dart format` y revisar braces.

### 9. Temas como singleton estático

`AppColors.setTheme` muta global; widgets que no rebuild-ean pueden quedar desfasados.

**Mejora:** `ThemeExtension` / provider de tema que dispare rebuild.

### 10. Navegación sin router

`Navigator` imperativo dificulta deep links, tests de flujo y tablet layouts.

**Mejora:** `go_router` o rutas nombradas mínimas.

---

## P1 — Producto / UX

### 11. Solo Android en el árbol visible

Sin `ios/` (y desktop ignorado). Si el objetivo es multiplataforma Flutter, falta scaffolding.

### 12. i18n

Strings hardcodeados en inglés en UI. F-Droid / Play se beneficiarían de al menos ES/EN.

### 13. Accesibilidad

Colores como única señal (más “super hard mode”). Falta contraste/patrones/etiquetas para daltonismo.

### 14. Feedback cuando hint falla

`showHint()` puede devolver false si el solver no encuentra solución a tiempo; conviene mensaje UI explícito.

### 15. Código unlock en claro

`THANKYOU` está en README y en código. Está bien como “gracias por star”, pero no es un sistema de unlock real.

---

## P1 — Build local

### 15b. Firma release rompe debug sin `key.properties`

`android/app/build.gradle.kts` creaba siempre `signingConfigs.release` casteando props a `String`. Sin keystore → `null cannot be cast…` incluso en `assembleDebug`.

**Mitigación aplicada en local (2026-09-04):** crear release solo si existe `key.properties`; si no, release usa debug. Ver [06-setup-local.md](./06-setup-local.md).

### 15c. AVD de prueba demasiado pequeño

AVD `tradersworld` (~320×640) provoca overflow en `HomeView`. Mejor un Pixel / API 35 estándar para QA visual.

---

## P2 — Ops y release

### 16. CI solo manual

No hay pipeline en `push`/`PR` para analyze/test. Regresiones llegan tarde.

### 17. Workflow llamado “test” que no testea

Renombrar o añadir job real de tests evita confusión.

### 18. Dependencias Hive generator

`hive_generator` + `build_runner` en dev, pero adapters parecen escritos a mano (`user_*_adapter.dart`). Clarificar si el codegen se usa.

### 19. Documentación de contribución

README corto (marketing). Falta `CONTRIBUTING`, arquitectura, cómo correr/analizar.

### 20. Graphify / mapa mental

Este repo no estaba en el grafo global `~/.graphify`. Si se itera mucho, conviene `graphify update` local (sin commitear `graphify-out/`).

---

## Ideas de features (backlog personal)

- Daily challenge con seed del día
- Estadísticas (tiempo medio, win rate, racha)
- Tutorial interactivo en el tablero (no solo how-to estático)
- Export/import de progreso (JSON) sin nube
- Modo zen sin estrellas ni timer (más alineado al pitch “relaxing”)
- Web build para demo en portfolio

---

## Qué está bien (no romper)

- Separación domain / data / ui ya es mejor que un monolito único
- Offline + privacy es un diferenciador claro vs clones con ads
- Seed por nivel = puzzles reproducibles (buen debugging)
- Flame da polish (pour, partículas, SFX) por encima del típico puzzle Flutter barato
- Multi-perfil y resume de partida son features “adultas” poco comunes en clones

---

## Primeros pasos sugeridos si se fork-ea para portfolio

1. Tests de dominio + `flutter analyze` en CI.
2. Desacoplar `Color` del domain.
3. Unificar `applyPour`.
4. i18n ES/EN y copy de README en español si el target es LATAM.
5. Dec de arquitectura en el repo (versión pública) distinta de esta carpeta privada.
