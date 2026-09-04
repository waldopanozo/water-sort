# 06 — Setup local (apuntes de sesión)

Fecha: 2026-09-04. Notas de cómo se corrió el juego en esta máquina.

## Entorno instalado

| Componente | Ubicación / detalle |
|------------|---------------------|
| Flutter | `~/flutter` (stable **3.47.2**, Dart **3.13.2**) |
| Android SDK | `~/Android/Sdk` (`ANDROID_HOME`) |
| JDK | OpenJDK 17 |
| AVD usado | `tradersworld` (muy chico: ~320×640) |
| Emulador | `emulator-5554` |

Linux desktop: doctor marca falta `clang`, `ninja-build`, `libgtk-3-dev` (no usado).
Web: no hay carpeta `web/` en el repo.
`linux/` está en `.gitignore` y no venía en el árbol.

## Comandos para relanzar

```bash
export PATH="$HOME/flutter/bin:$PATH"
export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_SDK_ROOT=$HOME/Android/Sdk

# Emulador (si no está)
$ANDROID_HOME/emulator/emulator -avd tradersworld -gpu auto &

cd ~/work/opensource/water-sort
flutter pub get
flutter run -d emulator-5554 --debug
```

## Fix aplicado para poder buildear

`android/app/build.gradle.kts` exigía `key.properties` al crear el `signingConfig` release aunque fuera debug → crash:

`null cannot be cast to non-null type kotlin.String`

Cambio local: solo crear firma release si existe `key.properties`; si no, release usa debug keys.

**Estado:** cambio en el working tree del repo (sí se puede versionar si se quiere; no es secreto).

## Observaciones al probar

- App arrancó en debug; overflow amarillo en home por el AVD minúsculo.
- Primer frame lento / frames skipped en emulador (normal en x86 frío).
- Sin keystore de release no se puede firmar como Play Store; debug alcanza para jugar.

## CI vs local

Workflows GitHub solo hacen build/release con secrets de keystore; no hay `flutter test` en CI. Localmente tampoco había Flutter hasta esta sesión.
