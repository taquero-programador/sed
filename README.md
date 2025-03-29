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

Se sigue esta regla: si la extensión no contiene `*`, luego se agrega al final del nombre de
archivo como un sufijo; si la extensión contiene uno o más `*` caracteres, entonces cada uno
(asterisco) se reemplaza con el nombre de archivo actual. Esto permite agregar un prefijo al
archivo de respaldo, en lugar de un sufijo, o incluso para colocar copias de seguridad de los
archivos originales en otro directorio (siempre que el directorio ya exista).

Si no se proporciona ninguna extensión, el archivo original es sobrescrito sin hacer una copia
de seguridad.

`i` toma un argumento opcional, no debería ir ser seguido por otras opciones cortas:

**`sed -Ei '...' FILE`**: Igual como `-E` `-i` sin un sufijo de respalo `-FILE` sera editado
en du lugar sin crear un respaldo.

**`sed -iE '...' FILE`**: Esto es equivalente a `--in-place=E`, creando `FILE` como un respaldo
de `FILE`.

Tenga cuidado de usar `-n` con `-i`: el primero deshailita la impresión automática de líneas y
este último cambia el archivo original sin crear una copia de seguridad. Utilizado
descuidadamente (y sin el comando `p`), el archivo de salida estará vacío.

```sh
# WRONG USAGE: 'FILE' will be truncated.
sed -ni 's/foo/bar' FILE
```

**`-l _N_`** o **`--line-lenght=_N_`**: Especifica la longitud de envoltura de línea
predeterminada para el comando `l`. Una longitud de 0 (cero) significa nunca envolver líneas
largas. Si no es especificado, se considera que es 70.

**`--posix`**: GNU `sed` incluye varias extensiones para POSIX sed. Para simplificar la
escritura de scripts portátiles, está opción deshabilita todas las extensiones que este manual
documenta incluyendo comandos adicionales. Las mayorías de las extensiones aceptan `sed`
programas que están fuera de la sintaxis ordenada por POSIX, pero alguno de ellos (como el
comportamiento de `N`) en realidad viola el estándar. Si desea deshabilitar solo el último tipo
de extensión, puede establecer la variable `POSIXLY_CORRECT` a un valor vacío.

**`-b`** o **`--binary`**: Esta opción está disponible en todas las plataformas, pero solo es
efectiva cuando el sistema operativa hace una distinción entre archivos de texto y archivos
binarios. Cuando se hace tal distinción (como es el caso de MS-DOS Windows). Los archivos
Cygwin están compuestos de líneas separadas por una devolución de carro y un carácter de
alimentación de línea, y `sed` no ve el CR final. Cuando se especifica esta opción, `sed`
abrirá los archivos en modo binario, por lo tanto, no solicitan este procesamiento especial y
considerando las líneas para terminar en una alimentación de línea.

**`--follow-symlinks`**: Esta opción solo está disponible en plataformas que admiten enlaces
simbólicos y tienen efecto solo si la opción `-i` se especifica. En este caso, si el archivo
que se especifica en la línea de comandos hay un enlace simbólico `sed` sigue el enlace y edita
el destino final al enlace. El comportamiento predeterminado es romper el enlace simbólico,
para que el destino del enlace no se modifique.

**`-E, -r`** o **`--regexp-extended`**: Use expresiones regulares extendidas en lugar de
expresiones regulares básicas. Los regex extendidos son aquellas que aceptan `egrep`.
Históricamente, esta era una extensión de GNU pero el `-E`, se ha agregado al estándar POSIX.

**`-s`** o **`--separate`**: Por defecto, `sed` considerará los archivos especificados  en la
línea de comandos como una sola secuencia larga continua. Este GNU `sed` permite al usuario
consideralos como archivos separados: direcciones de rango (como `/abc/,/def/`) no están
permitidas para abarcar varios archivos, los números de línea son relativos al inicio de cada
archivo, `$` se refiere a la última línea de cada archivo, y archivos invocados desde el
comando `R` se rebobinan en el inicio de cada archivo.

**`--sandbox`**: En modo sanbox, `e/w/r` los comando son rechazados (programas que lo contiene
serán abortados sin ser considerados). el modo Sanbox garantiza que `sed` solo opera en los
archivos de entrada en la línea de comandos, y no se pueden ejecutar comandos externos.

**`-u`** o **`--unbuffered`**: Sin bufer tanto de entrada como de salida tan mínimo como
práctico. (Esto es particularmente útil si la entrada proviene de `tail -f`, y deseas ver la
salida tranformada lo antes posible).

**`-z`**, **`--null-data`** o **`--zero-terminated`**: Trata la entrada como un conjunto de
líneas, cada una terminada por un byte cero (el carácter ASCII `NUL`) en lugar de una nueva
línea. Esta opción puede ser utilizada con comandos como `sort -z` and `find -print0` para
procesar nombres de archivos arbitrarios.

Si no se dan las opciones `-e`, `-f`, `--expression` o `--file` en la línea de comandos,
entonces el primer arguemento de no opción en la línea de comandos es tomando como el script
para ser ejecutado.

Si no quedan parámetros de línea de comandos después de procesar lo anterior, estos parámetros
se interpretan como los nombre de los archivos de entrada a ser procesados. Un nombre de
archivo `-` se refiere al flujo de entrada estándar. La entrada estándar se procesará  si no se
especifican nombre de archivo.

### Estado de salida
Un estado de salida de cero indica éxito y un valor distinto a cero indica falla. GNU `sed`
devuelve los siguiente estados de salida:

**`0`**: Finalización exitosa.

**`1`**: Comando no válido, sintaxis no válida, expresión regular no válida o un comando de
extesnión GNU `sed` utilizado con `--posix`.

**`2`**: Uno o más de los archivos de entrada especificado en la línea de comandos no puede ser
abierto (por ejemplo, si no se encuentra un archivo o se deniega el permiso de lectura). El
procesamiento continuó con otros archivos.
