# sed (S)tream (ed)itor for Unix/Linux

## Introducción
`sed` es un editor de línea. Un edito de stream se utiliza para realizar tranformación básica de texto
en un flujo de entrada (un archivo o una entrada desde una tubería). Aunque en algunos aspectos es similar a un
editor, `sed` funciona haciendo un solo paso sobre el input, y por consiguiente es má eficiente. Pero es la capacidad de `sed`
para filtrar texto en una canalización lo que lo distingue de otros tipos de editores.

### Descripción general
Normalmente `sed` se invoca así:

    SCRIPT SCRIPT INPUTFILE...

Por ejemplo, para reemplazar todas las ocurrencias de `hello` a `world` en el archivo `input.txt`:

    sed 's/hello/world/' input.txt > output.txt

Si no se especifica INPUTFILE, o si INPUTFILE es `-`, `sed` filtrara el contenido de la entrada estándar.
Los siguientes comandos son equivalentes:
```bash
sed 's/hello/world/' input.txt > ouyput.txt
sed 's/hello/world/' < input.txt > output.txt
cat input.txt | sed 's/hello/world/' - > output.txt
```

`sed` escribe sobre la salida estándar. Use `-i` para editar archivos en lugar de imprimir la salida estándar. Ver
tambi{en} `W` `s///w` para escribir la salida en otros archivos. El siguiente comando modifica `file.txt` y no produce
ninguna salida:

    sed -i 's/hello/world/' file.txt

Por defecto, `sed` imprime todas las entradas procesadas (excepto las entradas que han sido
modificadas/eliminadas por comandos como `d`). Use `-n` para suprimir la salida. y el comando
`-p` para imprimir líneas especificas. El siguiente comando imprime solo lq línea 45 del archivo de entrada:

    sed -n '45p' file.txt
