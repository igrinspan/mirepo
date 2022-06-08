---
layout: page
title: Teórica 10
permalink: /materias/orga2/teo10/
---
# Teo 10 - Orga 2

(02:30hs)

# Protección

En esta clase vemos las cosas que tiene en cuenta el procesador cuando está en modo protegido.

Por cada cosa que hagamos, va a fijarse si está todo bien o si tiene que mandarnos una exepción.

## Chequeo del límite

## Chequeo del tipo

## Niveles de privilegio (Video 2)

Modelo de anillos. Hay 4 niveles de privilegio: 0, 1, 2 y 3. 

![Teo%2010%20-%20Orga%202%20f0d6e64faaee4426a4266f152b7e63d8/Untitled.png](../teos/teo10/Teo%2010%20-%20Orga%202%20f0d6e64faaee4426a4266f152b7e63d8/Untitled.png)

 **¿Por qué pensaron 4 anillos?**

0. Kernel del S.O.

1 y 2. Servicios del S.O 

3. Aplicaciones 

**¿Cómo se usa realmente?**

0. Kernel y servicios del S.O. 

1 y 2. No utilizados

3. Aplicaciones

### Reglas que contemplan los niveles de privilegio

Se suelen chequear cuando se carga un selector en el registro de segmento. 

Vamos a ver 3 tipos:

1. Descriptor Privilege Level (DPL): nivel de privilegio del segmento a ser accedido. Está en las puertas de llamada, de tarea, TSS o descriptores de segmento de código o datos.
2. Current Privilege Level (CPL): nivel de privilegio del código que está intentando acceder a un descriptor de segmento o a una puerta, etc. El procesador lo lee en el cache hidden del selector.
3. Requested Privilege Level (RPL): es el valor que se escribe en los primeros 2 bits de los selectores. Se puede sobreescribir por programa. El procesador lo compara con el CPL y si es menor, lo reemplaza por el CPL al momento de chequear el acceso. 
Si RPL > CPL, el procesador usa el RPL. No sirve d mucho el RPL.
En resumen, el procesador usa el **Effective Privilege Level (EPL).** EPL = máx(CPL, RPL).

Para poder usar algo con DPL (por ejemplo) 2, tu CPL va a tener que ser menor o igual a 2.

### ¿Qué significa el DPL en el descriptor?

- En un segmento de datos / Puerta de llamada / TSS: 
indica el máxima valor numérico que debe tener el código de una tarea para acceder a este segmento. Entonces DPL = 1 → solo se puede acceder desde segmentos de código con CPL = 0 ó 1.
- En un segmento de código no *conforming*:
indica el nivel de privilegio exacto que debe tener el código de una tarea para accederlo.
- En un segmento de código *conforming* o con puerta de llamada:
indica el mínimo valor numérico que debe tener el código actual para poder acceder a este código. Si DPL = 2, los programas con CPL = 0 o 1 **NO** pueden acceder a este segmento. ¿por qué el kernel no puede llamar al código de usuario?

Recuerdo: cuando la interrupción se producía en código de nivel 3, se producía una conmutación de pilas. Esa pila a la que se cambiaba estaba en el TSS de la tarea de nivel 3 que se estaba ejecutando.

Las pilas trabajan sólo con segmentos de código de su mismo nivel de privilegio. 

El código SÍ puede usar datos de menor privilegio, pero no pilas ni código de menor privilegio.

### Segmentos Conforming

Conforming = Ajustable.

Si un segmento de código es conforming, un segmento de código de menor privilegio puede llamar a un segmento de código de nivel mayor. Pero no cambia mucho, porque lo que hace este segmento conforming es ejecutarse desde el nivel de privilegio que lo llamó.

No hay mucha explicación razonable, casi ni se usa.

## Puertas de llamada

Son un mecanismo especial para transferir el control desde un segmento de código a otro de mayor privilegio.

Se basan en un descriptor de sistema, que debe estar en la GDT o LDT, y se acceden efectuando un CALL o JMP far, en el que el selector corresponde a un descriptor de Puerta de Llamada. El offset del JMP se ignora.

![Teo%2010%20-%20Orga%202%20f0d6e64faaee4426a4266f152b7e63d8/Untitled%201.png](../teos/teo10/Teo%2010%20-%20Orga%202%20f0d6e64faaee4426a4266f152b7e63d8/Untitled%201.png)

Se parece a un descriptor e interrupciones. Tiene un campo "extra" que indica la cantidad de parámetros con los que se hace la llamada. Son 5 bits.

En el DPL va el privilegio mínimo con el que se puede llamar esta puerta. Si ponemos DPL = 3, la pueden llamar todos. Si ponemos DPL = 1, sólo pueden llamarla código con CPL = 0 ó  1.

### Pila en un cambio de nivel de privilegio

El cambio de nivel de privilegio va a ser "para arriba", por ejemplo de nivel 3 a nivel 0. En la pila de nivel 3 pusheamos los 2 parámetros y luego hacemos el CALL.

Una vez que hacemos el CALL, en la pila del procedimiento al que llamamos se guarda la dirección de retorno (CS:EIP), los parámetros, y la referencia a la pila anterior (SS:ESP). 
¿De dónde se saca la pila a la que vamos? Del TSS. (SS0:ESP0).

Si usamos segmentos conforming, no se produce cambio de pila, porque el código se ejecuta desde el nivel que lo llamamos.

![Teo%2010%20-%20Orga%202%20f0d6e64faaee4426a4266f152b7e63d8/Untitled%202.png](../teos/teo10/Teo%2010%20-%20Orga%202%20f0d6e64faaee4426a4266f152b7e63d8/Untitled%202.png)

## Protección a Nivel de Páginas

El procesador primero hace toda la comprobación de segmentos. Cuando eso pasa, empieza a fijarse en las páginas.

Los chequeos se realizan en la codificación y ejecución de cada instrucción. Ahora hay 2 niveles: User (bit U/S = 0) y Supervisor (bit U/S = 1). Además, el tipo de página puede ser Lectura o Lectura/Escritura.

Los **CPLs 0, 1, 2** se relacionan con el nivel **Supervisor**.
El **CPL = 3** se corresponde con el nivel **Usuario**.

En el modo Supervisor, se puede acceder a todas las páginas, en modo User se puede acceder solo a las que tienen el bit en 1.

El procesador chequea la protección en el PD y en cada PT.

### Combinando protección de Segmentos y Páginas

El procesador evalúa siempre la protección de segmentos. 

La protección a nivel de página no pisa a la protección a nivel de segmentos. O sea si tenemos un segmento de sólo lectura no podemos poner que nuestra página permita escritura.

Sí podemos hacer un granulado o filtro cuando definimos los permisos en las páginas. Por ejemplo, podemos tener un segmento de lectura y escritura pero para algunas páginas de ese segmento ponemos que sean sólo de lectura, y esto funcionaría bien.

Así se combinan los permisos entre PDE y PTE:

![Teo%2010%20-%20Orga%202%20f0d6e64faaee4426a4266f152b7e63d8/Untitled%203.png](../teos/teo10/Teo%2010%20-%20Orga%202%20f0d6e64faaee4426a4266f152b7e63d8/Untitled%203.png)

<aside>
⚠️ Una buena práctica puede ser setear el PDE en User y después decidir para cada página si querés que sea User o Supervisor. 
Porque si ponés el PDE en Supervisor, ya todas las páginas van a ser Supervisor.

</aside>

---

---

Faltan los últimos 2 videos (20 minutos).