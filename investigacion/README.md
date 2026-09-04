# Investigación — Water Sort

Apuntes personales sobre el proyecto. **Esta carpeta está en `.gitignore`** y no se versiona.

## Contenido

| Documento | Qué cubre |
|-----------|-----------|
| [01-vision-del-proyecto.md](./01-vision-del-proyecto.md) | Qué es, propuesta de valor, distribución, licencia |
| [02-arquitectura.md](./02-arquitectura.md) | Capas, stack, módulos y dependencias |
| [03-flujo-de-juego.md](./03-flujo-de-juego.md) | Generación, jugabilidad, progreso y persistencia |
| [04-puntos-de-mejora.md](./04-puntos-de-mejora.md) | Deuda técnica, riesgos y oportunidades |
| [05-escalado-dificultad.md](./05-escalado-dificultad.md) | Cómo sube la dificultad por nivel (fórmulas + tablas) |
| [06-setup-local.md](./06-setup-local.md) | Flutter, emulador, fix Gradle, cómo relanzar |
| [07-diseno-tips-limitados.md](./07-diseno-tips-limitados.md) | Diseño tips: 3 desde niv. 10 / random (**implementado**) |
| [08-pr-tips-upstream.md](./08-pr-tips-upstream.md) | Cómo commitear + texto del PR a sidhant947 |
| [solucion-nivel-30.md](./solucion-nivel-30.md) | Solución 67 moves nivel 30 |
| [patch-gradle-signing-local.md](./patch-gradle-signing-local.md) | Fix firma local (no va en el PR) |

## Snapshot rápido

- **Tipo:** puzzle de ordenar colores en tubos (Water Sort)
- **Stack:** Flutter + Riverpod + Hive + Flame
- **Versión actual (pubspec):** `1.0.15+16`
- **Licencia:** GPL v3
- **Distribución:** Google Play + F-Droid
- **Código de dominio útil:** ~7k LOC en `lib/` (~24 archivos Dart)
- **Fork/remoto local:** `waldopanozo/water-sort` (upstream original del autor Sidhant)
- **Dificultad campaña:** +1 color cada 3 niveles (máx. 16); capacity 5/6 en múltiplos de 5/10; scramble más fragmentado desde niv. 7; timer desde niv. 4
- **Local:** Flutter en `~/flutter`; se probó en AVD `tradersworld`

Última revisión de estos apuntes: 2026-09-04

## Rama del fork
Esta carpeta vive en la rama `notes/investigacion-local` del fork `waldopanozo/water-sort`. No forma parte del PR upstream.
