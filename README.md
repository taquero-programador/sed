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

`sed` trata mutiples archivos de entrada como una cadena larga. El siguiente ejemplo imprime la primera línea
del primer archivo `one.txt` y la última línea del archivo `three.txt`. Use `-s` para invertir el comportamiento:

    sed -n '1p ; $p' one.txt three.txt

Sin las opciones `-e` o `-f`, `sed` usa la primera no opción de parámetro como el _script_ y las siguientes no opciones de parámtro
como archivos de entrada. Si `-e` o `-f` son usados para especificar un _script_, todos los parámetros que no son opcionales son tomados como archivos de entrada.
Las opciones `-e` y `-f` pueden ser combinadas, y pueden aparecer varias veces (en cuyo caso el guión final efectivo será la concatenación de todos los guiones individuales).

Los siguientes ejemplos son equivalentes:
```bash
sed 's/hello/world' input.txt > output.txt

sed -e 's/hello/world' input.txt > output.txt
sed --expression='s/hello/world' input.txt > output.txt

echo 's/hello/world' myscript.sed
sed -f myscript.sed input.txt > output.txt
sed --file=myscript.sed input.txt > output.txt
```

### Opciones de línea de comandos
El formato largo par invocar `sed` es:

    sed OPTIONS... [SCRIPT] [INPUTFILE...]

`sed` puede ser invocado con las siguientes opciones de comando:
**`--version`**: Imprime la versión de `sed` que se está ejecutando y un aviso de derechos de autor, luego sale.

**`--help`**: Imprime un mensaje de uso que resume brevemente estas opciones y la dirección de informe de erroes, luego sale.

**`-n`**
**`--quit`**
**`--silent`**: Por defecto, `sed` imprime el espacio de patrón al final de cada ciclo a través del _script_. Estas opciones deshabilitan la impresión automática, and `sed`
solo produce la salida cuando se le dice explicitamente a través del comando `p`.

**`--debug`**: Imprime la entrada `sed` de forma canónica con una descripción de la ejecución:
```bash
echo 1 | sed '\%1%s21232'
# 3
```

```bash
echo 1 | sed --debug '\%1%s21232'
#SED PROGRAM:
#  /1/ s/1/3/
#INPUT:   'STDIN' line 1
#PATTERN: 1
#COMMAND: /1/ s/1/3/
#MATCHED REGEX REGISTERS
#  regex[0] = 0-1 '1'
#PATTERN: 3
#END-OF-CYCLE:
#3

```

**`-e script`** o **`--expression=script`**: Añade los comandos de _script_ para establecer los comandos
que serán ejecutados mientras procesa la entrada.

**`-f script_file`** o **`--file=script-file`**: Añade comando a un archio de _script-file_ para definir los comando
que seran ejecutados mientras procesa la entrada.

**`-i[suffix]`** o **`--in-place[=suffix]`**: Esta opción especifica los archivos que serán modificados en su lugar.
GNU `sed` hace esto creando un archivo temporal y enviando la salida a este archivo en lugar de la salida estándar.

Esta opción implica usar `-s`.

Cuando el final del archivo es alcanzado, el archivo temporal es renombrado al nombre original del archivo de salida. La extesión, si se suministra,
es usada para modificar el nombre del archivo anterior, haciendo una copia del archivo.
