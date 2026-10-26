# PantechGympro

**GymPro** es el sistema de gestión híbrido (web + móvil) desarrollado por **PanTech**
para automatizar la operación de gimnasios pequeños y medianos. Nace como respuesta a
la gestión manual (hojas de cálculo, registros en papel) del gimnasio piloto **Hyper
Panchito Gym** (Lerma, Estado de México).

Este README vive en el repo especial `.github` de la organización y es lo que GitHub
muestra en [github.com/PantechGympro](https://github.com/PantechGympro) — es el punto
de entrada para cualquiera que se integre al equipo.

---

## Repositorios

GymPro se divide en 4 repos, migrados en octubre de 2026 desde un monorepo original a
esta estructura por servicio:

| Repo | Contenido | Stack |
|---|---|---|
| [`gympro-api`](https://github.com/PantechGympro/gympro-api) | Backend REST | Flask, PostgreSQL, MongoDB, Redis, scikit-learn |
| [`gympro-web`](https://github.com/PantechGympro/gympro-web) | Portal web | React (CRA), nginx |
| [`gympro-mobile`](https://github.com/PantechGympro/gympro-mobile) | App Android | React Native + Expo |
| [`gympro-infra`](https://github.com/PantechGympro/gympro-infra) | Orquestación y despliegue | Docker Compose, nginx (ingreso público) |

`gympro-mobile` **no participa en el despliegue del VPS** — no hay servicio `mobile` en
ningún `docker-compose*.yml`; consume la misma API que el portal web, pero se compila y
distribuye por separado (EAS / Play Store).

`gympro-infra` se clona como hermano de `gympro-api` y `gympro-web` (sus `docker-compose*.yml`
referencian `../gympro-api` y `../gympro-web` como `context:` de build) — ver el README
de ese repo para el detalle completo, incluyendo el punto crítico de no cambiar el
`name:` del proyecto de Compose (así se conservan los volúmenes de datos al migrar).

Cada repo tiene su propio README con instrucciones de instalación, variables de entorno
y estructura de carpetas. La documentación funcional/académica del proyecto (manuales,
entregables, encuestas) vive en Confluence/Jira del equipo, no en estos repos — el
monorepo original (`SistemaGYM`) queda archivado como referencia histórica del código
previo a esta migración.

---

## Modelo de ramas

Las 4 repos usan el mismo modelo:

```
dev-miguel ──┐
dev-francisco ─┼──► main ──► staging ──► prod
dev-maximiliano │      (PR+review)  (subdominio   (VPS real /
dev-carlos ───┘                      de pruebas)   release)
```

- **`dev-<nombre>`** (`dev-miguel`, `dev-francisco`, `dev-maximiliano`, `dev-carlos`):
  rama personal de trabajo diario. Cada quien comitea y hace push libremente en la suya;
  cuando algo está listo, abre un **Pull Request hacia `main`**.
- **`main`**: rama de integración, protegida — requiere Pull Request + al menos 1
  review antes de mergear (configurado en `Settings → Branches` de cada repo). Es el
  estado "lo que ya pasó revisión", pero todavía no es lo que corre en producción.
- **`staging`**: refleja el entorno de pruebas (el subdominio habilitado para validar la
  nueva arquitectura antes de tocar el VPS real). Se actualiza mergeando `main → staging`
  cuando se quiere probar un conjunto de cambios en ese entorno.
- **`prod`**: lo que realmente corre en el VPS de producción (donde opera el gimnasio
  piloto). Solo se actualiza mergeando `staging → prod` después de validar ahí — nunca
  directo desde `dev-*` o `main`, precisamente para no arriesgar una caída mientras el
  sistema está en operación real.

  En `gympro-mobile`, `staging`/`prod` se interpretan como canales de build (EAS
  `preview`/`production`) en vez de un despliegue de servidor, pero siguen el mismo
  flujo de promoción.

### Ramas por funcionalidad (`feature/*`)

No existe una rama `feature` fija. Cuando alguien empieza algo nuevo, crea su propia
rama descriptiva desde su `dev-<nombre>` (ejemplo: `feature/reportes-pdf`,
`feature/notificaciones-push`), trabaja ahí, y la integra de vuelta a su `dev-<nombre>`
o directo a un PR hacia `main` cuando esté lista. Son temporales — se borran después de
mergear.

### Convención de commits

Los repos ya vienen usando [Conventional Commits](https://www.conventionalcommits.org/)
desde antes de la migración — se mantiene:

| Prefijo | Para qué |
|---|---|
| `feat:` | Funcionalidad nueva |
| `fix:` | Corrección de un bug |
| `chore:` | Mantenimiento, configuración, dependencias |
| `docs:` | Solo documentación |
| `refactor:` | Cambio de estructura sin cambiar comportamiento |
| `perf:` | Mejora de rendimiento |

Con alcance opcional entre paréntesis: `feat(api): ...`, `fix(mobile): ...`.

---

## Buenas prácticas de GitHub (vigentes en los 4 repos)

- **CI** (`.github/workflows/ci.yml`): valida el código en cada push/PR a `main`
  (compila y lintea en `gympro-api`, build + lint en `gympro-web`, chequeo de tipos en
  `gympro-mobile`, validación de sintaxis de los compose en `gympro-infra`).
- **`CODEOWNERS`**: revisores por defecto asignados automáticamente en cada PR.
- **Plantillas** de Issue (bug/feature) y de Pull Request, para reportes y cambios
  consistentes.
- **Protección de `main`**: Pull Request + review obligatorios antes de mergear.

---

## Despliegue

El VPS real (`prod`) corre con `docker-compose.server.yml` desde dentro de
`gympro-infra/`, con `gympro-api/` y `gympro-web/` clonados como hermanos. El cambio del
monorepo original a esta estructura de 4 repos se valida primero en un subdominio de
pruebas (rama `staging`) antes de aplicarse al VPS real donde opera el gimnasio piloto,
precisamente para no arriesgar una caída del servicio mientras está en uso.

---

*Última actualización: octubre 2026, durante la migración del monorepo `SistemaGYM` a
esta estructura de 4 repos.*
