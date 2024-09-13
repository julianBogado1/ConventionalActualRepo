# GitHub Actions + Conventional Commit

### Requisitos previos

Contar con una instalacion de node global en la ultima version disponible

### Configuracion

En el root del repositorio, en un directorio .github/workflows/ creamos un release.yml

```yaml
name: Releases
on:
  push:            
    branches:
      - main

jobs:
  changelog:
    permissions:
      contents: write
      issues: write
      pull-requests: write
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: conventional Changelog Action
        id: changelog
        uses: TriPSs/conventional-changelog-action@v3.7.1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: create release
        uses: actions/create-release@v1
        if: ${{ steps.changelog.outputs.skipped == 'false' }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ steps.changelog.outputs.tag }}
          release_name: ${{ steps.changelog.outputs.tag }}
          body: ${{ steps.changelog.outputs.clean_changelog }}
```

Tags:

- name: nombre del workflow
- jobs: son las tareas a realizar, cada se puede seguir en la consola de github detelladamente
- permissions: los permisos para la action    
- steps: los pasos que se van a ejecutar al correr el workflow, donde se especifica que acciones utiliza, que variables toma y se pueden especificar condicionales

### Workflow Triggers

Un workflow se puede activar automaticamente con acciones de git

Para especificar cuando queremos que se triggeree y en que rama:

```yaml
on:
  push:            
    branches:
      - main
```

Aca vamos a trackear solo pushes hechos en la branch main

Para especificar mas acciones podemos agruparlas como un array: 

```yaml
on: [push, fork]
```

O simplemente listarlas

```yaml
on:
  label:
    types:
      - created
  push:
    branches:
      - main
  page_build:

```

<aside>
💡

Los nombres de las branches soportan expresiones regulares (con algunas modificaciones)

</aside>

Para mas info aca esta la [sintaxis permitida](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#filter-pattern-cheat-sheet)

### Configuracion en Github

para permitir realizar acciones desde un workflow debemos g

---

# Proceso de commit

Existen tres niveles posibles de versionado: 

<aside>
✅

v1.2.3

</aside>

donde 1 seria nivel MAJOR, 2 seria MINOR y 3 seria PATCH

cada nivel se modifica cuando se usa una keyword en los commits. 

Las keywords son las siguientes:

- fix: aumenta el numero PATCH
- feat: aumenta el numero MINOR
- BREAKING CHANGE: aumenta el MAJOR

Si bien en la [documentacion de Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) se especifican algunas mas, estas son las que me funcionaron

### Sintaxis del commit message

En linux cuando realizamos un commit sin mensaje se abre un editor nano, esto es util porque la sintaxis de los commits es bastante estricta y cualquier mismatch no va a triggerear el bot de Conventional Changelog Action en github

```yaml
git commit
```

lLuego en nano

```yaml
feat: <descripcion>

[optional body]

[optional footer]
```

Dentro de lo que es footer iria incluido el tag “BREAKING CHANGE”

Son muy importantes los espacios. Por las dudas agrego templates para cada nivel de release:

V0.0.1

```yaml
fix: the fix
```

V0.1.0

```yaml
feat: the feature
```

V1.0.0

```yaml
feat: the feature

BREAKING CHANGE: the breaking change
```

La convencion establece que los Breaking changes son cambios que modifican el uso de la api para los usuarios

---

# Desventajas de usar conventional commits

- No se puede actualizar la version del marketplace automaticamente ya que va linkeado al 2fA, esto se debe realizar manualmente
- Si no se respeta la sintaxis del commit, los cambios no figuran en el changelog. Esto se puede mitigar con herramientas que limitan los commits, como podria ser Husky o algun otro hook
