# 01 — Visión del proyecto

## Qué es

**Water Sort** es un juego de puzzle casual: el jugador vierte agua de colores entre tubos hasta que cada tubo lleno tenga un solo color (o quede vacío). Es el clásico “sort water / ball sort” en tubos.

Está construido con **Flutter**, pensado para Android, con foco en:

- Juego 100 % offline
- Sin anuncios ni tracking
- Privacidad (sin red, sin analytics)
- Niveles infinitos generados proceduralmente
- UI minimalista + animaciones (Flame)

## Propuesta de valor (según README)

| Promesa | Cómo se refleja en el código |
|---------|------------------------------|
| Niveles infinitos | `LevelGenerator.generate(levelNumber)` con seed = número de nivel |
| Offline | Persistencia local Hive; sin clientes HTTP |
| Sin tracking / ads | Sin SDKs de ads ni analytics en `pubspec.yaml` |
| Temas / skins | `ThemePack` + código de desbloqueo `THANKYOU` |
| Relajante | Temporizador opcional, efectos de sonido, haptic |

## Features de producto detectadas en código

- **Campaña por niveles** numerados (progreso + estrellas)
- **Modo aleatorio** por dificultad (Easy → Super Duper Hard)
- **Multi-perfil** local (nombre + emoji)
- **Undo**, **hints** (solver DFS), **reset**
- **Temporizador** opcional (desde nivel 4 / dificultades > Easy)
- **Modos de accesibilidad/challenge:** super hard (¿ocultar colores?), blur tubos resueltos, pour instantáneo
- **Temas:** 17 packs (`midnight`, `cyberpunk`, `forest`, …)
- **Audio:** `pouring.mp3`, `tube_complete.mp3`, `level_complete.mp3` vía FlameAudio

## Distribución y ops

- Package Android: `com.sidhant.watersort`
- CI: workflows manuales (`workflow_dispatch`) para build APK/AAB y release en GitHub
- Fastlane metadata presente (`fastlane/metadata/android/en-US`)
- Firma con keystore vía secrets de GitHub Actions

## Licencia y implicaciones

- **GPL v3:** cualquier derivado distribuido debe compartir fuentes bajo GPL.
- Si se publica un fork modificado (Play Store, F-Droid, etc.), hay que cumplir copyleft.

## Público objetivo

Jugadores casuales que buscan un puzzle sin fricción (sin energía, sin ads). También atractivo para usuarios F-Droid / privacy-conscious.

## Dónde profundizar en estos apuntes

- Escalado de dificultad: [05-escalado-dificultad.md](./05-escalado-dificultad.md)
- Setup local / emulador: [06-setup-local.md](./06-setup-local.md)
