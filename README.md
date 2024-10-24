# sed (S)tream (ed)itor for Unix/Linux

## Introducci{o}n

### Descripci{o}n general
Normalmente `sed` se invoca as{i}:

    SCRIPT SCRIPT INPUTFILE...

Por ejemplo, para reemplazar todas las ocurrencias de `hello` a `world` en el archivo `input.txt`:

    sed 's/hello/world' input.txt > input.txt
