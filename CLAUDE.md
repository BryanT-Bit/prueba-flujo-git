# CLAUDE.md

<!-- flujo-git:start -->
## Git: reglas de trabajo

El detalle está en [`docs/FLUJO-GIT.md`](docs/FLUJO-GIT.md). Lo que no se negocia:

- **Nunca commit ni push directo a `main`, a `develop` ni a una `release/*`.** Todo entra por PR.
- **Las ramas salen de `develop` actualizada** (`git fetch; git switch -c … origin/develop`). Un hotfix sale de
  `main`. De la `release/X.Y.Z` en testing salen los ajustes (`bugfix/*`) y los reverts (`revert/*`) que sacan
  una feature, no de `develop`, y sus PRs van a la release.
- **Commits y títulos de PR en Conventional Commits, en español**: `tipo(alcance): descripción`.
- **PR a `develop`**, o a la release si es un ajuste o un revert; a `main` solo un hotfix o una release. Siempre
  con el template del PR.
- **En un repo que corta release, los PRs a `develop` quedan abiertos hasta el corte**: se piden revisar y aprobar,
  pero no se mergean; los mergea el responsable de la release cuando el equipo los elige. Salvo los que llegan desde
  la release o desde un hotfix, y el que abre `sincronizar-flujo-git.ps1` de BitWebDocs (sólo archivos del flujo).
- **Todo se mergea con merge commit** (`gh pr merge --merge`): squash y rebase están apagados.
- **Build y tests en local antes del PR**: todavía no hay CI de tests.
- **No crear tags ni releases** salvo pedido explícito.
- **Sin force-push** sobre ramas compartidas.
<!-- flujo-git:end -->

## Arquitectura

- La plataforma es un contenedor: **Gateway (YARP) + IdentityServer + N sub-apps**. Cada sub-app se abre en un iframe
  desde el menú, y tiene un frontend (Razor) y un backend que se despliegan por separado.
- **Todo el tráfico pasa por el Gateway**, también el de frontend a backend. Nunca hay llamadas directas entre
  componentes.
- **Los datos de los clientes se acceden por una fachada**: es la única vista de esas bases. No asumir cómo está
  implementada. Algunas sub-apps usan APIs externas, también a través del Gateway.
- **La fachada no controla el acceso: autoriza el Gateway.**
- La cookie vive en el frontend e identifica al usuario y su empresa activa. **Los permisos NO están en la cookie**: se
  consultan al IdentityServer al cargar el menú.
- **El Gateway valida que el recurso pertenezca a la empresa activa** e inyecta las cabeceras `X-Bit-*` hacia el backend.
- **Rutas conocidas**: `/api/app/{appId}/*` (frontend → Gateway → backend), `/api/facade/*` (backend → Gateway →
  fachada) y `GET /_bit/manifest`.
- **El backend va en tres capas**:
  - **Controller**: distribuye, sin lógica de negocio.
  - **Service**: le pide los datos al Repository y aplica toda la lógica.
  - **Repository**: obtiene los datos de la fachada o de la API externa y los devuelve tal cual.

  Si una solución rompe esta separación, decirlo y justificarlo antes de hacerla.
- **No inventar nombres de rutas, servicios ni tipos de la plataforma.**
- **Stack**: C#/.NET 8, ASP.NET Core, Razor, IIS en Windows (Hosting Bundle).

## Completar en este repo

- **Rol del repo:**
- **Solución, proyectos y namespaces:**
- **Convenciones de nombres:**
- **Registro de dependencias:**
- **Manejo de errores y logging:**
- **Testing:**
- **Cliente HTTP:**
- **Repos relacionados y orden de despliegue:**
