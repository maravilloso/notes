GIT
===

- Configurar que al hacer push se suban todas las ramas de nombre igual a las existentes en origin (remoto)

```shell
  git config --global push.default matching
```

- Ejemplo de contenido de fichero `~/.gitconfig`:

```shell
[core]
	editor = \"C:/Program Files (x86)/GitExtensions/GitExtensions.exe\" fileeditor
	longpaths = true
	packedGitLimit = 4095m
	packedGitWindowSize = 4095m
	compression = 9
[user]
	name = Marcos Vega
	email = marcos.vega@protom.me
[pack]
	deltaCacheSize = 4095m
	packSizeLimit = 4095m
	windowMemory = 4095m
```
- Ignorar ficheros que ya están manejados por GIT (i.e.: already commited)
```shell
git update-index --skip-worktree **/.classpath **/**/.classpath axpomdm-jar/.settings/org.eclipse.jdt.core.prefs
```

- Clonar solo cierta rama desde remoto:

```shell
  git clone -b rama --single-branch https://...git
```

- Afrontar el error "fatal: early EOF" al clonar repositorios muy grandes o con mala conexión de red, haciéndolo en varios pasos:

```shell
  git clone --depth=1 ssh://git@mi-servidor.com:7999/err/<MIREPO>.git
  cd <MIREPO>/
  git fetch --unshallow
  git pull --all
```

- Incluir en un commit todos los ficheros cambiados de una vez:

```shell
  git commit -a -m " ... comentario ..."
```

- Corregir el comentario del último commit (solo si aún no se ha hecho push)

```shell
  git commit --amend
```

- Incluir otro fichero en el último commit (solo si aun no se hizo push)

```shell
  git add . # or add individual files
  git commit --amend --no-edit
```

- Crear otra rama y conmutar a ella en un solo paso:

```shell
  git checkout -b NUEVARAMA
```

- Resolver conflicto merging:

```shell
  # Usando la versión "nuestra"
  git checkout --ours PATH/FILE
  # Usando la versión "suya"
  git checkout --theirs README.rst
```

- Borrar todos los ficheros que aún no están en staging

```shell
  git clean -n	# verlos/consultarlos
  git clean -f	# Borrarlos definitivamente
  git clean -fd # Incluir directorios
```

- Lista rápida de commits con su ID y comentario para una rama

```shell
  git log --oneline PRO ^master
```

  Si se quiere ver el detalle de lo modificado en uno de dichos commits usando su ID tal como lo mostro el comando log anterior:

```shell
  git show 0586184
```

- Visualizar diferencias existentes entre dos ramas

```shell
  # Diferencias en detalle
  git diff [--stat] master..PRO

  # Solo los nombres de ficheros que han cambiado respecto a la rama principal
  git diff --name-only master...
```

- Deshacer un pull que hiciste entre ramas equivocadas y se mezclaron

```shell
  git reset --hard
```

- Actualizar mi rama con los últimos cambios en remoto sin perder mi trabajo:

```shell
  git stash
  git pull
  git stash pop
```

- Deshacer todos los cambios hechos en mi working copy

```shell
  git stash save --keep-index
  git stash drop
```

- Borrar ramas

```shell
  # Borrar solo rama remota
  git push origin --delete <RAMA-A-BORRAR>
  # Borrar solo rama local
  git checkout <otra-rama-diferente>
  git branch -d <RAMA-A-BORRAR>
```

- Añadir a una rama cualquiera lo más actual desde master

```shell
  git checkout <la-otra-rama>
  git merge origin/master
```

- Descargar a local cierta rama que solamente existe en remoto

```shell
  git fetch origin <RAMA>:<RAMA>
```

- Descargar solo cierta rama remota, combinandola con la actual (master?)

```shell
  git checkout -b <RAMA>
  git pull origin <RAMA>
  ...
  # Specify a reason for the merge
  ...
  git push --set-upstream origin <RAMA>
```

- Deshacer el último merge desde otra rama (cuando dicho merge fué la última operación)
```shell
  git reset --hard HEAD~1
```

- Renombrar rama (tanto en local como en remoto)
```shell
# Names of things - allows you to copy/paste commands
old_name=feature/old
new_name=feature/new
remote=origin

# Rename the local branch to the new name
git branch -m $old_name $new_name

# Delete the old branch on remote
git push $remote --delete $old_name

# Prevent git from using the old name when pushing in the next step.
# Otherwise, git will use the old upstream name instead of $new_name.
git branch --unset-upstream $new_name

# Push the new branch to remote
git push $remote $new_name

# Reset the upstream branch for the new_name local branch
git push $remote -u $new_name
```
