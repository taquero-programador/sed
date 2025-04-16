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
```sh
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
```sh
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
```sh
echo 1 | sed '\%1%s21232'
# 3
```

```sh
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

**`4`**: Un error I/O de procesamiento grave durante el tiempo de ejecución GNU `sed`
abortado inmediatamente.

Además, los comandos `q` y `Q` puden ser utilizados para terminar `sed` con un código de salida
personalizado:

```sh
echo | sed 'Q42' ; echo $?
# 42
```

## `sed` scripts

### `sed` descripción general del script
Un programa `sed` consisten en uno o más comandos `sed`, pasados uno a uno o más opciones
`-e`, `-f`, `--expression` and `--file`, o la primera no opción si el argumento es cero
esa opciones serán usadas. Este documento se refiere a al script `sed`; se entiende que esto
significa la concatenación en orden de todos los scripts y archivos de script.

Los comandos `sed` siguen esta sintaxis:

    [addr]X[options]

X es una sola letra de comando `sed`. `[addr]` es una dirección de línea opcional. Si `[addr]`   
se especifica, el comando X se ejecutará solo en las líneas coincidentes. `[addr]` puede ser
un único número de línea, una expresión regular o un rango de líneas. Adicional `[addr]` se
utiliza para algunos comandos `sed`.

El siguiente ejemplo elimina las líneas 30 y 35 en la entrada. `30,35` es un rango de
direcciones. `d` es el comando eliminar:

    sed '30,35d' input.txt > output.txt

El siguiente ejemplo imprime todas las entradas hasta una línea comenzando con la palabra `foo`.
Si se encuentra dicha línea, `sed` terminará con el estado de salida 42. Si no se encontró dicha
línea (y no se produjo ningún otro error), `sed` saldrá con el estado 0. `/^foo/` es una
dirección de expresión regular. `q` termina el comando. `42` es la opción del comando.

    sed '/^foo/q42' input > output.txt

Los comandos dentro de un script se pueden separar por punpt y coma (`;`) o nuevas líneas
(ASCII 10). Se pueden especficiar mútiples scripts con `-e` o ``-f.

Los siguientes ejemplos son todos equivalentes. Realizar dos operaciones `sed`: eliminar
cualquier línea que coincida con la expresión regular `/^foo/`, y reemplazando todas las
ocurrencas de la cadena `hello` con `world`:

```sh
sed '/^foo/d/ ; s/hello/world/' input.txt > output.txt

sed -e '/^foo/d' -e 's/hello/world/' input.txt > output.txt

echo '/^foo/d' > script.sed
echo 's/hello/world/' >> script.sed
sed -f script.sed input.txt > output.txt

echo 's/hello/world/' > script2.sed
sed -e '/^foo/d' -f script2.sed input.txt > output.txt
```

Comandos como `a`, `c` e `i` debido a su sintaxis, no pueden ser seguidos por un punto y coma,
por lo tanto, debe ser terminado con nuevas líneas o ser colocado al final de un script. Los
comandos también pueden ir precedidos de caracteres en blanco.

### Resumen de comandos
Los siguientes comandos son compatibles con GNU `sed`. Algunos son comandos POSIX, mientras que
otros son extensiones GNU. Los detalles y ejemplos para cada comando se encuentra en las
siguientes secciones.

```sh
a\
text
```
Apendice texto después de una línea.

**`a text`**: Apendice texto después de una línea (sintaxis alternativa).

**`b label`**: Subdivisión incondicionalmente a etiqueta. La etiqueta puede omitirse en cuyo
caso se inicia el siguiente ciclo.

```sh
c\
text
```
Reemplace (cambie) líneas con texto.

**`c text`**: sintaxis alternativa.

**`d`**: Eliminar el espacio del patrón; comience inmediatamente el siguiente ciclo.

**`D`**: Si el espacio del patrón contiene nuevas líneas, elimine el texto en el patrón hasta
la nueva línea y reinicie el ciclo con el resultado espacio del patraón, sin leer una nueva
línea de entrada.

Si el espacio del patrón no contiene nueva línea, inicie un nuevo ciclo normal como si `d`
emitio el comando.

**`e`**: Ejecuta el comando que se encuentra en el espacio del patrón y reemplaza el espacio
del patrón con la salida; una nueva línea posterior es suprimido.

**`e command`**: Ejecuta el comando y se envía su salida al flujo de salida. El comando puede
ejecutarse a través de mútiples líneas, todas menos el último final de una barra.

**`F`**: (nombre de archivo) imprime el nombre del archivo de entrada actual (como resultado
final de nueva línea).

**`g`**: Reemplace el contenido del espacio de patrón con el contenido del espacio de retención.

**`G`**: Añada una nueva línea al contenido del espacio del patrón, y luego el contenido del
espacio de retención al del espacio del patrón.

**`h`**: (hold) Reemplace el contenido del espacio de retención con el contenido del espacio
de patrón.

**`H`**: Añadir una nueva línea al contenido del espacio de espera, y luego el contenido del
espacio del patrón al del espacio de retención.

```sh
i\
text
```
Insertar texto antes de una línea.

**`i text`**: sintaxis alternativa.

**`l`**: Imprima el espacio del patrón en una forma inequívoca.

**`n`**: (siguiente) Si la impresión automática no estpa deshabilitada, imprima el espacio del
patrón luego, independientemente, reemplace el espacio del patrón con la siguiente línea de
entrada. Si no hay más entradas, entonces `sed` saldrá sin procesar más comandos.

**`N`**: Agregue una nueva línea al pesacio del patrón, luego agregue la siguiente línea al
espacio del patrón. Si no hay más entradas, entonces, `sed` sale sin procesar más comandos.

**`p`**: Imprima el espacio del patrón.

**`P`**: Imprima el espacio del patrón, hasta la primera <newline>.

**`@[exit-code]`**: Este comando es el mismo que `q`, pero no imprimirá el contenido del
espacio del patrón. Como `q`, proporciona la capacidad para devolver un código de salida.

**`R filename`**: Una línea cola de nombre de archivo para ser leído e insertado en el flujo
de salida del ciclo actual, o cuando se lee la siguiente línea de entrada.

**`s/regexp/replacement/[flag]`**: Haga coincidir la expresión regular con el contenido del
espacio del patrón Si se encuentra, reemplace la cadena coincidente con `replacement`.

**`t label`**: (prueba) Rama a etiqueta solo si ha habido éxito de substitución desde la
última línea de entrada leída o condicional como fue tomada. La etiqueta puede omitirse, en
cuyo caso se inicia el siguiente ciclo.

**`T label`**: (casi similar).

**`v [version]`**: Este comando no hace nada, pero hace fallar a `sed` si las extensiones no
son compatibles, o si la versión solicitada no está disponible.

**`w filename`**: Escribe el espacio del patrón para el nombre del archivo.

**`W filename`**: escriba el nombre del archivo dado la parte del espacio del patrón hasta la
primera línea nueva.

**`x`**: Intercambia el contenido de los espacio de retención y patrón.

**`y/src/dst/`**: Traduce cualquier carácter en el espacio de patrones que coincida con
cualquier _source-chars_ con el carácter correspondiente _dest-chars_.

**`z`**: (zap) Este comando vacía el contenido del espacio del patrón.

**`#`**: Comentario.

**`{ cmd; cmd ...}`**: Agrupe varios comandos juntos.

**`=`**: Imprima el número de línea de entrada actual.

**`: label`**: Especifique la ubicación de la etiqueta para comandos de rama (`b`, `t`, `T`).

### El comando `s`
El comando `s` (sustituto) es probalemente el más importante en `sed` y tiene muchas opciones
diferentes. La sintaxis del comando `s` es `s/regexp/replacement/flags`.

Su concepto básico es simple: el comando `s` intenta coincidir el espacio de patrón contra la
expresión regular suministrada _regexp_; si es exitoso, entonces esa parte del espacio de
patrón será reemplazada con `replacement`.

El reemplazo puede contener (_n_ ser un númer del 1 al 9) referencia, que se refieren a la
porción de la coincidencia que está contenida entre _n_ y su concidencia. Además, el reemplazo
puede contener caracteres sin espacio que hacen referncia a toda la porción coincidente del
espacio de patrón. `\n\(\)&`

Los caracteres `/` pueden ser sustituidos uniformemente por cualquier otro único carácter de
cualquier comando dado a `s`. El carácter `/` (o cualquier otro carácter se utiliza en su
lugar) puede aparecer en _regexp_ o _replacement_ solo si está precedido por un carácter `\`.

Finalmente, como extensión GNU `sed`, puede incluir una secuencia especial hecha de una rección
inversa y una de las letras `L`, `l`, `U`, `u` o `E`. El significado es el siguiente:

**`\L`**: Gire el reemplazo para minúsculas hasta que `\U` o `\E` se encuentren.

**`\l`**: Gire el siguiente carácter a minúscula.

**`\U`**: Gire el reemplazo a mayúsculas hasta que `\L` o `\E` se encuentren.

**`\u`**: Gire el siguiente carácter en mayúscula.

**`\E`**: Detener la conversión de casos iniciada por `\L` o `\U`.

Cuando se está utilizando la bandera `g`, la conversión de casos no propaga desde una expresión
de la expresión regular a otra. Por ejemplo, cuando se ejecuta el siguiente comando con
`a-b-` en el espacio de patrón:

    s/\(b\?\)-/x\u\1/g

La salida es `axxB`. Al reemplazar la primera `-`, la secuencia `\u` solo afecta el reemplazo
vacío de `\1`. No afecta al carácter `x` que es agreagado al espacio del patrón al reemplazar
`b-` con `xB`.

Por otro lado, `\l` y `\u` afecta al resto del texto de sustitución se van seguidos de una
sustitución vacía. Con `a-b` en el espacio del patrón:

    s/\(b\?\)-/\u\1x/g

reemplazará `-` con `X` y `b-` con `Bx`. Si este comportamiento es indeseable, puede evitarlo
agragando una secuencia `\E` después de `\1` en este caso.

Para incluir un literal `\`, `&`, o nueva línea final de reemplazo, asegúrese de preceder lo
deseado con un `\`.

El comando `s` puede ser seguido por cero o más de las siguientes _flags_:

**`g`**: Aplique el reemplazo a todo con _regexp_, no solo al primero.

**`num`**: Solo reemplaza el número de coincidencia de la _regexp_. Interacción con el comando
`s`.

**`p`**: Si se realizó la sustitución, imprima el nuevo espacio de patrón. Nota: cuando se
especifican ambas opciones `p` y `e`, el orden relativo de los dos produce resultados muy
diferentes. En general `ep` (evalúa luego imprime) es lo que quiere, pero operar al revés puede
ser útil para depurar. Por esta razón, la versión actual de GNU `sed` interpreta especialmente
la presencia de `p` tanto antes como después `e`, imprimiendo el espacio del patrón antes y
después de la evaluación, mientras que en general, opciones para el comando `s` mostrará su
efecto solo una vez.

**`w filename`**: Si se realizó la sustitución, escriba el resultado en el archivo nombrado.
Como una extesión GNU `sed`, dos valores especiales del nombre de archivos son soportados:
`/dev/stderr`, cuando escribe el resultado del error estándar, y `/dec/stdout` cuando escribe
la salida estándar.

**`e`**: Este comando permite canalizar la entrada desde un comando shell en el espacio de
patrón. Si se realizó una sustitución, el comando que se encuentra en el espacio de patrón se
ejecuta y el espacio de patrón se reemplaza con su salida. Se suprime una nueva línea posterior;
los resultados no están definidos si el comando a ejecutar es un carácter Null.
Esto es una extensión GNU `sed`.

**`I i`**: Modificardor a la coincidencia de expresión regular es una extensión GNU _regexp_
en manera insensible.

**`M m`**: El modificador a la coincidencia de expresión es un GNU `sed` que dirige para que
coincida con la expresión regular en multi-línea. Coincide al principio o al final de la línea
`^` y `$` (`\` y `\`).

### Comandos de uso frecuente

**`#`**: [No permite direcciones] El `#` carácter comienza con un comentario; el comentario
continúa hasta la próxima nueva línea.

Si le preocupa la portabilidad, tenga en cuneta que algunas implementaciones de `sed` solo pueden
admitir un solo comentario de una línea y luego solo cuando el primer carácter del script es un `#`.

Advetencia: si los dos primeros caracteres de un script son `#n`, entonces la opción `-n` es
forzada. Si quieres poner un comentario en la primera línea de un script y ese comentario comienza
con la letra `n` y no se quiere este comportamieto, asegúrese de usar un `N` mayúscula o coloque un
espacio antes de la `n`.

**`q[exit-code]`**: Salir de `sed` sin procesar más comandos o entradas.

Ejemplo: parar después de imprimir la segunda línea:
```sh
seq 3 | sed 2q
1
2
```

Este comando solo acepta una dirección. Tenga en cuenta que el espacio del patrón actual se
imprime si la impresión automática es no deshabilitada con la opción `-n`. La habilidad de
devolver un código de `sed` script es una extensión GNU `sed`.

**`d`**: Eliminar el espacio del patrón; comience inmediatamente el siguiente ciclo.

Ejemplo: eliminar la segunda línea de entrada:
```sh
seq 3 | sed 2d
1
2
```

**`p`**: Imprima el espacio del patrón (a la salida estándar). Este comando generalmente solo
se usa junto con la opción `-n` de la línea de comandos.

Ejemplo: imprima solo la segunda línea de entrada:
```sh
seq 3 | sed -n 2p
2
```

**`n`**: Si la impresión automática no está deshabilitada, imprima el espacio del patrón,
independientemente, reemplace el espacio del patrón con la siguiente línea de entrada. Si no
hay más entradas `sed` saldrá sin procesar más comandos.

Este comando es útil para omitir líneas (por ejemplo, procesar cada línea Nth).

Ejemplo: realizar la sustitución en cada 3 líneas (es decir, cada `n` comandos omitos dos líneas)
```sh
seq 6 | sed 'n;n;s/./x/'
1
2
x
4
5
x
```

GNU `sed` proporciona una sintaxis de dirección de estensión _first-step_ para logar el mismo
resultado:
```sh
seq 6 | sed '0~3s/./x/'
1
2
x
4
5
x
```

**`{ commands }`**: Un grupo de comandos puede estar encerrado entre `{}`. Esto es
particularmente útil cuando desea un grupo de comandos para ser activado por una única
coincidencia de dirección (o rango de dirección).

Ejemplo: realice la sustición y luego imprima la segunda línea de entrada:
```sh
seq 3 | sed -n '2{s/2/X/ ; p}'
x
```

### Comandos menos frecuentes
Aunque quizás se usa con menos frecuencia que los de la sección anterior, algunas muy pequeñas
pero útiles scripts de `sed` se pueden construir con estos comandos.

**`y/source-char/dest-char/`**: Traduce cualquier carácter en el espacio del patro que
coincida con cualquiera de los caracteres de `source-char`

Ejemplo: transliterar `a-j` en `0-9`:
```sh
echo hello world | sed 'y/abcdefghij/0123456789/'
74lo worl3
```

El carácter de `/` puede ser reemplazado por cualquier otro carácter.

Instancia de `/`, `\`, o pueden aparecer nuevas líneas en `source-char` o `dest-char`, siempre
que cada instancia se escape por un `\`. La `source-char` y `dest-char` deben contener el mismo
número de caracteres.

**`a text`**: Añade texto despues de una línea. Esta es una extensión GNU estándar al comando
`a`.

Ejemplo: agregue la palabra `hello` después de la segunda línea:
```sh
seq 3 | sed '2a hello'
1
2
hello
3
```

Los espacios iniciales después del comando `a` se ignoran. El texto a añadir se lee hasta el
final de la línea.

```sh
a\
text
```
Añade texto después de una línea.

Ejemplo: agreagar `hello` después de la segunda línea (`-` indica líneas de salida impresas):
```sh
eq 3 | sed '2a\
hello'
-|1
-|2
-|hello
-|3
```

El comando `a` pone en cola las líneas de texto que siguen este comando para salir al final del
ciclo actual, o cuando se lee la siguiente línea de entrada.

Como un extensión de GNU, este comando acepta dos direcciones.

Secuencias de escape en texto se procesa, por lo que deben usar `\\` para imprimir una sola
barra invertida.

Los comandos se reanudad después de la última línea sin una rección inversa `\`, `world` en
el siguiente ejemplo:
```sh
seq 3 | sed '2a\
hello\
world
3s/./X/'
-|1
-|2
-|hello
-|world
-|X
```

Como extensión de GNU, el comando `a` y texto pueden ser separado en dos parámetros `-e`, lo que
permite un script más fácil:
```sh
seq 3 | sed -e '2a\' -e hello
1
2
hello
3

sed -e '2a\' -e $ "$VAR"
```

**`i text`**: Intertar texto antes de una línea. Esta es una extensión GNU estándar al comando
`i`.

Ejemplo: Inserte la palabra `hello` antes de la segunda línea:
```sh
seq 3 | sed '2i hello'
1
hello
2
3
```
