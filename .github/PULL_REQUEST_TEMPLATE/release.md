## Release vX.Y.Z (release/X.Y.Z → main)

<!-- Este PR sale de release/X.Y.Z (nunca de una rama de cierre ni de develop) y se mergea con MERGE COMMIT, nunca squash. Se abre junto con el PR de la release a develop, antes de mergear el primero. -->

Ficha de la release: <!-- enlace al issue «Release X.Y.Z» de bitsistemas/BitWebDocs -->

## Contenido del release

<!-- Los cambios que entran, agrupados por tipo. Sale de los títulos de los PRs mergeados en develop que entraron en el corte (la ficha los lista). -->
- feat:
- fix:
- otros:

## Validación

<!-- Todo esto queda anotado en la ficha, no sólo acá. -->
- [ ] Testing aprobó el rc `X.Y.Z-rc.N` (la etiqueta concreta, sin cambios después de ella) y avisó que la release está cerrada: quién avisó y cuándo, anotado en la ficha.
- [ ] El equipo decidió que la release va a producción.
- [ ] Build y tests en local sobre `release/X.Y.Z`.

## Impacto en deploy

- **Orden de despliegue** (backend antes que frontend):
- **Configuración** por entorno:
- **Repos relacionados** que se despliegan junto con este:

## Después de mergear

- [ ] Crear la etiqueta y el release sobre el merge commit de este PR (`gh pr view <número> --json mergeCommit --jq .mergeCommit.oid`): `gh release create vX.Y.Z --target <merge commit de este PR> --generate-notes`
- [ ] Mergear el PR de la release a `develop` (ya abierto; si da conflicto, la rama `cierre/X.Y.Z`).
- [ ] Desplegar.
- [ ] Actualizar la tabla de versiones por entorno.
- [ ] Cerrar la ficha de la release.
