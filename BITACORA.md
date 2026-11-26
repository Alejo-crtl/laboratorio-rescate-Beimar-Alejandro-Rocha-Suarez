# Bitácora del laboratorio de rescate

# 1. Análisis de los comandos `git log`

## a. `git log`

Muestra el historial de commits **alcanzables desde la rama actual (`HEAD`)**, en orden del más reciente al más antiguo.

---

## b. `git log --oneline`

Muestra el historial en formato compacto: **una línea por commit**.

---

## c. `git log --oneline --graph --all`

Muestra el historial de forma compacta, con un **gráfico ASCII** y incluyendo **todas las ramas y referencias** del repositorio.

---

## ¿Qué es el código que aparece al inicio de cada commit?

El código que aparece al inicio, por ejemplo:

```text
47bd0f931479254850a1536529f33b8f28d4a5e1
```

o su forma abreviada:

```text
47bd0f9
```

es el **hash del commit**.

Es un identificador único calculado por Git a partir del contenido del commit y de sus metadatos. En este repositorio se muestra como un hash SHA-1 de 40 caracteres hexadecimales. La forma corta, normalmente de 7 caracteres, es una abreviatura del hash completo.

Ese hash:

- Identifica de forma única el commit.
- No es un número de versión.
- No es un código de error.
- Permite referirse al commit en comandos como `git show`, `git checkout`, `git diff`, etc.

---


# 2. Análisis: `git diff` vs `git diff --staged`

## ¿Por qué el segundo `git diff` no mostró nada?

`git diff` **sin opciones** compara:

```text
working directory  <-->  staging area (index)
```

Es decir, muestra **solo los cambios que aún NO han sido agregados con `git add`**.

### Paso a paso

**1️ Primer `git diff` (antes de `git add`)**

El archivo `notas.txt` fue modificado en el working directory, pero esos cambios **todavía no estaban en el staging area**. Por eso `git diff` mostró:

**2️ `git add notas.txt`**

Este comando **mueve los cambios del working directory al staging area**. A partir de este momento, el working directory y el staging area contienen exactamente lo mismo para ese archivo.

**3️ Segundo `git diff` (después de `git add`)**

Como el working directory y el staging area **son idénticos**, no hay diferencias que mostrar. Por eso la salida fue **vacía**.

---

## ¿Qué muestra `git diff --staged`?

`git diff --staged` (o su equivalente `git diff --cached`) compara:

```text
staging area (index)  <-->  último commit (HEAD)
```

Es decir, muestra **los cambios que ya están preparados (`staged`) y que se incluirán en el próximo commit**, comparándolos contra la última versión confirmada.

En este caso, después de `git add notas.txt`, esos cambios ya están en el staging area, por lo que `--staged` los vuelve a mostrar:

---

# 3. Análisis: `git status` + `git restore` sobre `notas.txt`


## ¿Qué hizo `git restore notas.txt`?

Sin `--staged`, `git restore` **solo actúa sobre el working directory**: recupera el archivo desde el **staging area**.

Es decir:

- **Staging area** → se queda igual (con la versión modificada de 3 líneas).
- **Working directory** → se restaura el archivo `notas.txt` con el contenido que está en el staging area.

Por eso, después de ejecutarlo, el archivo **vuelve a existir en disco** con las tres líneas, y el estado `deleted` desaparece.

# ¿Funcionaría `git restore notas.txt` si nunca se hubiera hecho commit del archivo?

- **Sí funcionaría** en el escenario del laboratorio, porque el archivo **sí había sido agregado con `git add`** (estaba en el staging area), aunque nunca se hubiera commiteado.
- **No funcionaría** si el archivo nunca hubiera pasado por `git add`: en ese caso Git no tiene ninguna copia y responde con `error: pathspec 'notas.txt' did not match any file(s) known to git`.
- La confusión típica viene de creer que `git restore` recupera desde el último commit. **No es así**: recupera desde el **índice**. Por eso puede "revivir" archivos que nunca fueron commiteados, siempre que hayan sido staged.

---

# 4. Diferencia entre `git restore` y `git restore --staged`

| Comando | Qué hace | Cuándo usarlo |
|---|---|---|
| `git restore <archivo>` | Restaura el archivo en disco desde el staging area | Cuando te arrepientes de cambios **no preparados** |
| `git restore --staged <archivo>` | Saca el archivo del staging (vuelve a `HEAD`) sin tocar el disco | Cuando hiciste `git add` por error o quieres "des-preparar" |

En una frase:

- **`git restore`** recupera **contenido** desde el índice hacia el disco.
- **`git restore --staged`** recupera **el estado del índice** desde `HEAD`, sin tocar el disco.

# 5. `git revert`: ¿borra el commit o crea uno nuevo?


| Pregunta | Respuesta |
|---|---|
| ¿`git revert` borró el commit equivocado? | **No.** El commit original sigue en el historial. |
| ¿`git revert` creó uno nuevo? | **Sí.** Crea un commit nuevo con los cambios inversos. |
| ¿`git revert` "restaura" el archivo? | Indirectamente: el commit nuevo **revierte el contenido**, por eso ves el archivo "restaurado", pero eso lo logra **añadiendo un commit**, no eliminando uno. |
| ¿Por qué es seguro tras `push`? | Porque **no reescribe la historia**: solo añade un commit al final, así todos los demás pueden hacer `git pull` sin conflictos ni `push --force`. |

----

# 6. Diferencia entre `git reset --soft` y `git revert`

git reset --soft
----------------
BORRA los commits pero deja los cambios guardados.
Los cambios quedan listos para volver a commitear.

Solo usarlo si los commits NO los has subido (push).
Porque cambia la historia y daña a los demás.


git revert
----------
NO borra nada. Crea un commit NUEVO que deshace el anterior.
El commit malo sigue en la historia, pero sus efectos se anulan.

Usarlo cuando el commit YA lo subiste (push).
Es seguro porque no cambia la historia.

---

# 7. ¿Qué hace `git stash` y `git stash -- pop`?

git stash
---------
Guarda temporalmente tus cambios sin hacer commit.
Deja tu carpeta limpia como si no hubieras tocado nada.

¿Para qué sirve?
- Tienes cambios a medias y necesitas cambiar de rama.
- Llegó un bug urgente y no quieres commitear lo que llevas.
- Quieres probar algo y luego volver a lo que estabas.

Es como una "cajonera" donde metes tus cambios
y los sacas después cuando los necesites.


git stash pop
-------------
Saca los cambios guardados y los pone de vuelta en tu carpeta.
Y borra esa entrada de la cajonera (ya no queda guardada).

Es como abrir el cajón, sacar lo que guardaste y cerrarlo vacío.


