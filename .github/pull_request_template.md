## Qué cambia

<!-- En una o dos frases: qué hace este PR y por qué. -->

## Tipo

<!-- Marcá uno. Tiene que coincidir con el tipo del título (Conventional Commits). -->
- [ ] feat · funcionalidad nueva
- [ ] fix · corrección
- [ ] refactor · reestructura sin cambio de comportamiento
- [ ] perf · rendimiento
- [ ] test · tests
- [ ] docs · documentación
- [ ] chore / build / ci · mantenimiento, build o integración
- [ ] revert · revierte un cambio anterior

## Ticket / contexto

<!-- Link al ticket o a la conversación, y lo que haga falta para entender el cambio. -->

## Cómo probarlo

<!-- Pasos concretos para verificarlo: comandos, pantallas, datos de prueba. -->

## Impacto en deploy

- **Orden de despliegue** (backend antes que frontend): <!-- qué se despliega primero, o «ninguno» -->
- **Configuración** (appsettings, claves, variables, IIS): <!-- qué hay que cambiar en cada entorno, o «ninguna» -->
- **Repos relacionados** (PRs que tienen que entrar junto con este): <!-- links, o «ninguno» -->

## Checklist

- [ ] El título sigue Conventional Commits en español: `tipo(alcance): descripción`.
- [ ] Controller sin lógica de negocio / Service con las reglas / Repository sin transformaciones.
- [ ] Todo pasa por el Gateway: no hay llamadas directas entre componentes.
- [ ] No se leen permisos de la cookie (se consultan al IdentityServer).
- [ ] Si cambia el contrato de la sub-app, se actualizó `/_bit/manifest`.
- [ ] Tests agregados o actualizados.
- [ ] Build y tests corridos en local.

## Notas para quien revisa

<!-- Dónde mirar con más cuidado, decisiones discutibles, lo que quedó afuera a propósito. -->
