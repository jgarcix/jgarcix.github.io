---
title: ¿Cómo realizar un tratamiento a la tty?
published: true
---


# [](#header-1)Introducción


En la mayoría de las máquinas, cuando accedemos al sistema mediante una reverse shell, por ejemplo, nos encontramos frente a una TTY en la que no podemos manejarnos cómodamente. Esto se lleva a cabo con la finalidad de poder desplazarnos con las flechas, usar el tab para autocompletar...

```bash
root@overpass-prod:~# tty
tty                                                                                        
not a tty                                                                                  
root@overpass-prod:~#
```

# [](#header-1)Desarrollo 

Primero deberemos estbalecer una tty por defecto mediante:

```bash
root@overpass-prod:~# script /dev/null -c bash                                             
script /dev/null -c bash                                                                   
Script started, file is /dev/null                                                          
root@overpass-prod:~# tty                                                                  
tty                                                                                        
/dev/pts/1 
```

A continuación cancelamos el netcat con Ctrl+Z y ejecutamos: 

> stty raw -echo; fg
> reset xterm

Esto es para resetear la xterm accediendo a la consola en la que podemos hacer Ctrl+C y no cargarnos la reverse shell

Hay ocasiones en las que TERM no equivale a xterm. Para que así sea ejecutamos:

> export TERM=xterm

```bash
root@overpass-prod:~# echo $TERM
dumb
root@overpass-prod:~# export TERM=xterm
root@overpass-prod:~# echo $TERM
xterm
```

La consola en la que estamos ya es interactiva pero proporcionalmente no esta adecuada al tamaño de nuestra terminal. Podemos ver las proporciones con: 

> stty size

Para establecer las proporciones adecuadas primero deberemos saber cuantas filas y columnas partimos desde nuestra terminal. En otra ventana consultamos el tamaño con el mismo comando y lo restablecemos con:

> stty rows (X) columns (X)

Y ya estaría!

# [](#header-1)Conclusión

A modo de resumen los comandos que habría que ejecutar serían los siguientes:

> script /dev/null -c bash

> Ctrl+Z

> stty raw echo; fg

> reset xterm

> export TERM=xterm

> stty rows 44 columns 140

Espero que te haya gustado!
