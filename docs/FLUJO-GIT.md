# Flujo de Git

Cómo trabajamos con ramas, commits, PRs, versiones y hotfixes en este repo. Vale igual para las personas y para
Claude Code. La decisión es ADR-072, en BitWebDocs. El recorrido con un caso de tres semanas, paso a paso, está en
el artefacto «GitFlow en BitWeb» (<https://claude.ai/artifact/VbktKfaeF8GEi61cDuTfBQ>); donde difiera, vale este
documento.

## Ramas

| Rama | Para qué | Sale de | Entra a | Método |
|---|---|---|---|---|
| `main` | Producción. Sólo recibe releases y hotfixes, cada uno con su etiqueta `vX.Y.Z`. | — | — | — |
| `develop` | Lo elegido para la próxima release. No tiene servidor: nadie prueba en `develop`. | `main`, una vez | — (de acá se corta la release) | — |
| `feature/<tema>` | Funcionalidad nueva, en una app que existe o en una nueva. | `develop` | `develop` | PR; espera al corte |
| `release/X.Y.Z` | La versión en testing. Sólo recibe ajustes, reverts y hotfixes. Hay una por vez. | `develop`, en el corte | `develop` siempre, y `main` si el equipo lo decide | PR, al cerrar |
| `cierre/X.Y.Z` | Resolver el conflicto del PR de la release a `develop`: trae `develop` a una rama que sale de la release. Existe sólo si ese PR da conflicto. | La release | `develop` | PR, al cerrar |
| `bugfix/<tema>` | Un ajuste que pidió testing, o un error de producción que puede esperar. | La release (un ajuste) o `develop` (lo que puede esperar) | La rama de la que salió | PR; a `develop`, espera al corte |
| `hotfix/<versión>` | Un error de producción que no puede esperar. | `main` | `main`, y la release en testing o, si ese repo no tiene, `develop` | PR, cuando testing lo aprobó |
| `revert/<tema>` | Sacar una feature de la release, o volver a traerla a `develop`. | La release, o `develop` | La rama de la que salió | PR; a `develop`, espera al corte |
| `chore/*` | Mantenimiento, build, configuración. | `develop` | `develop` | PR; espera al corte |
| `docs/*` | Documentación. | `develop` | `develop` | PR; espera al corte |

«Espera al corte» vale en los repos que cortan release: los que no, están en «Varios repos».

## Reglas

- **Todo entra por PR**: a `develop`, a `release/*` y a `main`. Nadie pushea a esas ramas: las reglas lo
  rechazan.
- **Todo se mergea con merge commit** (`gh pr merge --merge`). Squash y rebase están apagados: así cada rama
  conserva de dónde vino cada cambio, y una feature se puede sacar entera con un revert.
- **Una sola release en testing por vez.** Mientras una release está en testing, y hasta el corte siguiente,
  los PRs a `develop` quedan abiertos: revisados y aprobados, pero sin mergear. GitHub no lo puede hacer cumplir:
  lo cumplimos nosotros. El PR `chore: sincronizar los archivos del flujo de ramas`, que abre el script
  `scripts/sincronizar-flujo-git.ps1` de BitWebDocs, no espera al corte: lleva sólo archivos del flujo, desde la
  fuente revisada, y se mergea apenas está aprobado. Tampoco esperan el cierre de una release ni el segundo PR de
  un hotfix (ver «Cierre» y «Hotfix»).
- **Sin force-push** sobre ramas compartidas, y no se borran `main` ni `develop`.
- Cada PR necesita **1 aprobación de otra persona** y **las conversaciones resueltas**. En `release/*` y en
  `main`, un push nuevo al PR descarta la aprobación: hay que volver a aprobar.
- `.github/CODEOWNERS` pide la revisión de quienes nombra en cada PR. **Sólo asigna revisores**: la aprobación la puede
  dar cualquiera con permiso de escritura.
- El único check obligatorio es **`pr-title`**: valida el título del PR (ver abajo). En `release/*` todavía no se
  exige: la `release/1.0.0` se cortó con el workflow viejo, que no corre ahí. Hasta que cierre la 1.0.0, el título
  de un PR a la release lo controla quien aprueba.
- **Dos PRs desde la misma rama** (el cierre de una release, un hotfix): abrí los dos antes de mergear el primero,
  porque al mergear GitHub borra la rama. Si la borró y el otro PR quedó cerrado, «Restore branch» en el PR
  mergeado la vuelve a subir y `gh pr reopen <número>` lo reabre.

## Commits y títulos de PR

Conventional Commits, en español:

```
tipo(alcance): descripción
```

- **Tipos**: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`, `perf`, `build`, `revert`.
- **Alcance**: opcional, en minúsculas (`feat(odapi): …`).
- **Cambio incompatible**: un `!` antes de los dos puntos (`feat(auth)!: …`).
- **El título del PR es el mensaje del merge commit** que queda en la rama destino: cuidalo tanto como un commit.

Ejemplos:

```
feat(odapi): cifrar la query hacia la fachada
fix: devolver 502 cuando la fachada no contesta
docs: explicar el flujo de ramas
feat(auth)!: exigir la clave interna en el login
```

## El ciclo de un cambio

```powershell
git fetch
git switch -c feature/cifrar-query origin/develop     # o bugfix/…, chore/…, docs/…

# … cambios, commits …
dotnet build
dotnet test                            # los tests se corren en local: todavía no hay CI de tests

git push -u origin feature/cifrar-query
gh pr create --base develop --title "feat(odapi): cifrar la query hacia la fachada"
```

`gh pr create` toma el template del PR (`.github/pull_request_template.md`). Completá «Impacto en deploy» aunque
sea para decir «ninguno».

Revisado y aprobado, en un repo que corta release el PR **queda abierto hasta el corte**: lo mergea el responsable
de la release cuando el equipo lo elige (ver «El corte»). Los que no esperan, como el del sync de BitWebDocs, están
en «Reglas». Si mientras espera queda en conflicto con `develop`, traé `develop` a tu rama
(`git merge origin/develop`), pusheá y pedí que lo vuelvan a mirar.

## El ciclo de una release

Una release es la versión que va a salir: `release/X.Y.Z`, cortada de `develop`. **El servidor de testing tiene la
release en testing**, y testing prueba siempre una etiqueta de la release, nunca `develop`. Cada semana: el viernes
se corta la release, testing la prueba y pide ajustes, y cuando la aprueba se cierra; recién entonces se corta la
siguiente.

Lo lleva el **responsable de la release**, un rol: con el equipo elige qué entra, corta, etiqueta, escribe la nota
de entrega, abre los PRs de cierre y lleva la ficha (ver «Varios repos»). Es mucho trabajo repetido en muchos
repos, y para no cometer errores tiene la skill `release-semanal` de Claude Code, en el paraguas `BitWebProject`:
junta el estado de la semana y ejecuta lo que decide el equipo, confirmando cada acción en GitHub antes de hacerla.
Claude no decide.

### Números y etiquetas

- **SemVer** (`MAJOR.MINOR.PATCH`), con un número para toda la plataforma (ver «Varios repos»). Cada release sube
  MINOR (`1.1.0`, `1.2.0`…) y un hotfix sube PATCH (`1.1.1`). Un `!` en algún título de la release es un cambio
  incompatible: sube MAJOR.
- Lo que va a testing lleva la etiqueta **`X.Y.Z-rc.N`**, sin `v`, una por ronda: `1.1.0-rc.1`, `1.1.0-rc.2`… Es una
  etiqueta anotada sobre la release, no una rama, y no genera release en GitHub.
- Lo que va a producción lleva **`vX.Y.Z`**, sobre `main`, con su release en GitHub.
- Testing aprueba una etiqueta concreta: si después entra cualquier cambio, es un rc nuevo y se vuelve a aprobar.

**Convención de tags de este repo:** `vX.Y.Z` para producción y `X.Y.Z-rc.N` para testing.

### El corte

Es el viernes, con la release anterior cerrada. **Una semana sin release cerrada, o sin nada nuevo, no tiene
corte.**

1. El responsable junta los PRs abiertos a `develop` de todos los repos, con lo que hace falta para decidir: si
   están aprobados y revisados, si se pueden mergear, y si tocan varios repos y tienen que entrar juntos.
2. **El equipo elige** cuáles entran, y el responsable los mergea:

   ```powershell
   gh pr merge 42 --merge --match-head-commit <cabeza que pasó la guarda>
   ```

   La `<cabeza que pasó la guarda>` es la cabeza del PR en GitHub (`git rev-parse origin/<rama del PR>`), del mismo `fetch` sobre el que corrió la guarda de los revisores (ADR-059; el procedimiento está en el `CLAUDE.md` del paraguas): GitHub rechaza el merge si al PR se le sumó algo después.

   Si uno queda en conflicto con otro que entró antes, no se mergea: su autor trae `develop` a su rama, y entra
   si lo vuelven a revisar a tiempo o, si no, en el corte siguiente.
3. **En cada repo que va a cortar, build y tests en local sobre el `develop` que quedó** (un worktree de
   `origin/develop` después de un `fetch`, donde dice el `CLAUDE.md` del paraguas; el de una sub-app, en `BitWebSubApps/_worktrees/`)
   y se anota `git rev-parse origin/develop`, el `<sha probado>`: dos PRs que pasan solos pueden romperse juntos. Si falla, ese repo no se
   corta hasta arreglarlo: un PR a `develop` con el arreglo, o el revert del PR que lo rompió; lo decide el equipo.
4. **Se corta la release** en cada repo que tiene en `develop` algo que producción no tiene (lo que entró en este
   corte, o una release que espera producción), y se le pone el primer rc. Se recomienda no cortar un repo cuya
   diferencia es sólo archivos del flujo, los que trae el PR del sync: decide el equipo, y esos archivos viajan
   con la próxima release de ese repo que tenga un cambio real.

   ```powershell
   git fetch
   git rev-parse origin/develop   # tiene que dar el <sha probado>; si entró algo (un hotfix), se vuelve a probar
   git switch -c release/1.1.0 <sha probado>
   git push -u origin release/1.1.0
   git tag -a 1.1.0-rc.1 -m "1.1.0-rc.1"
   git push origin 1.1.0-rc.1
   ```

5. Se abre la ficha de la release, con lo que entró y la nota de entrega del rc.1, y `1.1.0-rc.1` va al servidor de
   testing.

Mientras testing prueba, el desarrollo sigue en sus ramas desde `develop`, con los PRs abiertos hasta el corte
siguiente. Nada de eso llega a testing.

### Ajustes y rc

Testing manda los ajustes en Jira, todos juntos por ronda, contra el número de la etiqueta. Cada ajuste es una rama
`bugfix/<tema>` que **sale de la release y vuelve a ella, nunca de `develop`**: el ajuste es para lo que se está
probando, y llega a `develop` cuando la release cierra.

```powershell
git fetch
git switch -c bugfix/filtro-orden origin/release/1.1.0
# … el ajuste, con sus tests; build y tests en local …
git push -u origin bugfix/filtro-orden
gh pr create --base release/1.1.0 --title "fix(deudores): respetar el orden del filtro por RUT"
```

Aprobado, se mergea con `gh pr merge <número> --merge --match-head-commit <cabeza que pasó la guarda>`: un PR a la release no espera al corte.

Con todos los ajustes de la ronda adentro, el responsable pone el rc siguiente **en todos los repos de la release**,
aunque alguno no haya cambiado desde el anterior:

```powershell
git fetch
git tag -a 1.1.0-rc.2 -m "1.1.0-rc.2" origin/release/1.1.0
git push origin 1.1.0-rc.2
```

y escribe en la ficha la nota de entrega: qué cambió desde el rc anterior y qué se sacó. `1.1.0-rc.2` va al
servidor de testing.

### Sacar una feature

Si algo no se puede arreglar a tiempo, **lo decide el equipo en cada caso**: se espera, o se saca de la release
para no seguir atrasando el resto. Se saca con un revert del merge que la trajo a `develop`, en una rama
`revert/<tema>` que sale de la release, en todos los repos que la feature toca:

```powershell
gh pr view 42 --json mergeCommit --jq .mergeCommit.oid   # 42: el PR que trajo la feature a develop
git fetch
git switch -c revert/exportar-excel origin/release/1.1.0
git revert -m 1 <merge del ajuste 57>   # primero los ajustes de la feature, del más nuevo al más viejo
git revert -m 1 <merge de la feature>
git push -u origin revert/exportar-excel
gh pr create --base release/1.1.0 --title "revert: sacar exportar-excel de la 1.1.0"
```

Si la feature ya tuvo ajustes en la release (sus PRs `bugfix/*` a la release), en la misma rama se revierten
primero esos PRs de ajuste, del más nuevo al más viejo, y después el merge de la feature; cada uno con su
`mergeCommit` (`gh pr view <n> --json mergeCommit --jq .mergeCommit.oid`). Si no se revierten,
el revert de la feature choca con lo que el ajuste cambió encima, o el ajuste queda suelto en la release.

Si la feature (o un ajuste) entró con squash (toda la 1.0.0, antes de ADR-072), el commit no es un merge:
`git revert <commit>`, sin `-m 1`.

Aprobado y mergeado, sale en el rc siguiente, y la ficha lo anota como decisión del equipo.

El trabajo no se pierde, pero **no vuelve solo**: al cerrar, el revert llega a `develop` con la release, y Git da la
feature por incluida (mergear su rama otra vez no la trae). Para traerla, después del cierre se abre el «revert del
revert»: un PR a `develop` desde una rama `revert/<tema>` que sale de `develop`, con el arreglo de la feature en la
misma rama, en todos los repos que la feature toca. **Se decide en el corte, como cualquier otro PR.** La ficha
lista lo que se sacó: son los «revert del revert» pendientes.

Se revierte **el merge del PR `revert/<tema>` a la release**, que después del cierre ya está en `develop`: así
vuelven juntos la feature y sus ajustes. Si ese PR entró con squash, `git revert <commit>` sin `-m 1`.

```powershell
git fetch
git switch -c revert/exportar-excel origin/develop    # después del cierre: el revert ya está en develop
gh pr view 61 --json mergeCommit --jq .mergeCommit.oid   # 61: el PR revert/exportar-excel → release/1.1.0
git revert -m 1 <merge del PR del revert>
# … el arreglo de la feature, con sus tests …
git push -u origin revert/exportar-excel
gh pr create --base develop --title "feat(excel): volver a traer exportar-excel, corregido"
```

### Cierre

Testing avisa que la release está cerrada, en general por teléfono. El responsable lo anota en la ficha: el rc
aprobado, quién avisó y cuándo. La release **se mergea a `develop` siempre**, y **ahí el equipo decide si también
va a producción**:

- **Va a producción:** dos PRs desde `release/1.1.0`, a `main` y a `develop`.
- **Espera:** sólo el PR a `develop`. La release queda aprobada y esperando: la siguiente sale de `develop` y la trae
  adentro, y cuando esa vaya a producción lleva las dos. Producción nunca tiene la `v1.1.0`, y la ficha de la 1.1.0
  queda abierta hasta entonces.

```powershell
git fetch
gh pr create --base develop --head release/1.1.0 --title "chore: release 1.1.0 a develop" --body "Cierre de la release 1.1.0. Ficha: <enlace>"
# si va a producción, el segundo, antes de mergear el primero:
gh pr create --base main --head release/1.1.0 --title "chore: release v1.1.0" --body-file .github/PULL_REQUEST_TEMPLATE/release.md
```

`--body-file` toma la plantilla de tu copia del repo. La `release/1.0.0` se cortó antes de que existiera la del
ciclo semanal: su cierre usa la de BitWebDocs, `flujo-git/.github/PULL_REQUEST_TEMPLATE/release.md`.

Cada PR necesita su aprobación. Se mergea primero el de producción, se le pone la etiqueta, y después el de
`develop`:

```powershell
gh pr merge <número del PR a main> --merge --match-head-commit <rc aprobado>
gh pr view <número del PR a main> --json mergeCommit --jq .mergeCommit.oid   # el merge commit de ese PR
gh release create v1.1.0 --target <merge commit de ese PR> --generate-notes
gh pr merge <número del PR a develop> --merge --match-head-commit <cabeza que pasó la guarda>
```

`<rc aprobado>` es el SHA de la etiqueta que aprobó testing (`git rev-parse 1.1.0-rc.3^{commit}`).
La etiqueta `v1.1.0` va sobre el merge commit del PR, no sobre la punta de `main`, que puede haberse movido entre el merge
y la etiqueta. Después se despliega `v1.1.0` en producción y se actualiza la tabla de versiones por entorno. La
ficha se cierra cuando la release llega a producción, y con ella la de una release que esperó y viajó adentro.

**Si el PR a `develop` da conflicto** —pasa si entró algo a `develop` después del corte—, no se resuelve en
GitHub: eso commitea en `release/1.1.0`, y las reglas lo rechazan. Se resuelve en una rama de cierre, que sale de la
release y trae `develop`:

```powershell
git fetch
git switch -c cierre/1.1.0 origin/release/1.1.0
git merge origin/develop                 # los conflictos se resuelven acá
git push -u origin cierre/1.1.0
gh pr create --base develop --head cierre/1.1.0 --title "chore: release 1.1.0 a develop" --body "Cierre de la release 1.1.0. Ficha: <enlace>"
gh pr close <número del PR de release/1.1.0 a develop>
```

El PR a producción sigue saliendo de `release/1.1.0`, no de la rama de cierre: a producción va lo que aprobó
testing, sin nada de `develop`. Si la release espera producción, `release/1.1.0` queda sin PR que la borre: se borra
a mano (`git push origin --delete release/1.1.0`), y sus etiquetas quedan.

**Si el que da conflicto es el PR a producción**, `main` tiene un cambio que la release no trae igual: un hotfix
que no pasó por ella, o uno de antes de este flujo, que entró a `develop` con squash. Tampoco se resuelve en
GitHub: un `bugfix/<tema>` que sale de la release hace `git merge origin/main` y vuelve a la release por PR, y
testing aprueba un rc nuevo antes de cerrar.

### El corte siguiente

Recién con la release cerrada se corta la siguiente, el viernes, como en «El corte». Entre los PRs abiertos a
`develop` están los «revert del revert» de lo que se sacó; y si la release cerrada espera producción, la nueva la
trae adentro.

## Hotfix

Un error de producción que no puede esperar sale de `main`, con el número de la plataforma: con producción en
`v1.1.0`, el hotfix es `1.1.1`.

```powershell
git fetch
git switch -c hotfix/1.1.1 origin/main
# … el arreglo, build y tests en local …
git push -u origin hotfix/1.1.1
git tag -a 1.1.1-rc.1 -m "1.1.1-rc.1"
git push origin 1.1.1-rc.1
```

1. **Testing lo prueba como `1.1.1-rc.1`, con todas las demás apps en la versión de producción.** El servidor de
   testing es uno solo: mientras tanto, la release en testing espera.
2. Aprobado, **dos PRs desde `hotfix/1.1.1`**: a `main`, y a la release en testing o, si ese repo no tiene
   release en testing (no hay ninguna, o ese repo no cambió en ella), a `develop`.

   ```powershell
   gh pr create --base main --title "fix(auth): aceptar el login sin clave vieja"
   gh pr create --base release/1.2.0 --title "fix(auth): aceptar el login sin clave vieja"   # sin release/1.2.0 en el repo: --base develop
   ```

3. **Primero el de `main`**, con su etiqueta de parche:

   ```powershell
   gh pr merge <número del PR a main> --merge --match-head-commit <rc aprobado>
   gh pr view <número del PR a main> --json mergeCommit --jq .mergeCommit.oid   # el merge commit de ese PR
   gh release create v1.1.1 --target <merge commit de ese PR> --generate-notes
   ```

4. **Después el otro.** A la release: vuelve a testing con un rc nuevo en todos sus repos (`1.2.0-rc.2`: lo mismo
   que el anterior, más el arreglo). A `develop`: **entra en el momento, aunque no haya corte**, porque ya está en
   producción y, si esperara, la próxima release saldría sin él. Sin este segundo PR, el error volvería a producción
   con la próxima release.

Si el segundo PR da conflicto, se resuelve en la rama del hotfix (`git merge origin/release/1.2.0`, u
`origin/develop`, o con el editor de GitHub): producción ya tiene lo suyo, y la rama del hotfix no tiene reglas.
Por eso el de `main` va primero. El responsable anota el hotfix en la ficha de la release en testing.

## Varios repos

- **Un número para toda la plataforma.** Cada repo que cambia abre su `release/X.Y.Z` con el mismo número; los que
  no cambian no abren release. Cada rc se etiqueta en todos los repos de la release, aunque alguno no haya cambiado
  desde el rc anterior: testing ve un solo número. Un hotfix lleva el número de la plataforma (`v1.1.1`, aunque ese
  repo haya quedado en `v1.0.0`) y se etiqueta sólo en los repos que toca.
- **Una feature que toca varios repos** usa el mismo nombre de rama en todos, y entra, se ajusta o se saca en todos a
  la vez.
- Los cambios entre repos tienen que ser **retrocompatibles**: lo nuevo convive con lo viejo hasta que todos los
  repos se actualizaron.
- **El backend se despliega antes que el frontend.**
- El PR lo dice en **«Impacto en deploy»**: el orden, la configuración y los PRs de otros repos que tienen que entrar
  juntos.

### La ficha de la release

Cada release tiene una ficha: un issue de `bitsistemas/BitWebDocs` titulado «Release X.Y.Z», con la etiqueta
`release`, que se abre en el corte y se cierra cuando la release llega a producción. Jira no sirve de fuente: testing
lo usa poco, y el cierre llega por teléfono. La ficha asienta:

- qué entró en el corte: los PRs, por repo;
- cada rc, con su fecha y la nota de entrega (qué cambió desde el anterior y qué se sacó);
- los ajustes que pidió testing, con su clave de Jira si la tienen, el PR `bugfix/*` de cada uno y su estado;
- las decisiones del equipo: qué se sacó con un revert, y si la release va a producción o espera;
- lo que avisa testing por teléfono, con quién y cuándo.

La lleva Claude con la skill `release-semanal`; lo que llega por teléfono se le dicta, o se anota a mano. A mano,
`gh issue create --repo bitsistemas/BitWebDocs --title "Release 1.1.0" --label release` la abre, y
`gh issue list --repo bitsistemas/BitWebDocs --label release` muestra las fichas.

### Los repos que no cortan release

Cortan release los repos que se despliegan: BitErpWeb, las sub-apps y los tres ApiCore (FacadeWrapper,
Notifications y ServiceAgent). **No cortan release** el SDK (`BitFwk.SubApp`, un paquete NuGet con su propio
número), BitWebDocs y BitWebEjercicios. En ellos **los PRs a `develop` no esperan al corte**: se mergean apenas están
aprobados. Cuando hace falta, `develop` pasa a producción con un PR a `main`, con merge commit, y lleva la etiqueta
de su convención de tags.

## Tests

Todavía no hay CI de tests: **build y tests se corren en local antes de abrir el PR**, y el checklist del template
lo pide.

## En una urgencia

Si las reglas de la rama traban algo que no puede esperar, un admin puede **desactivar el ruleset temporalmente**.
Se vuelve a activar apenas pasó la urgencia, y queda registrado en el PR: qué se desactivó, por qué y cuándo se
reactivó.
