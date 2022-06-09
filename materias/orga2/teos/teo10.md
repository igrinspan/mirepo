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

![teo10-1][1]


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

![teo10-2][2]

Se parece a un descriptor e interrupciones. Tiene un campo "extra" que indica la cantidad de parámetros con los que se hace la llamada. Son 5 bits.

En el DPL va el privilegio mínimo con el que se puede llamar esta puerta. Si ponemos DPL = 3, la pueden llamar todos. Si ponemos DPL = 1, sólo pueden llamarla código con CPL = 0 ó  1.

### Pila en un cambio de nivel de privilegio

El cambio de nivel de privilegio va a ser "para arriba", por ejemplo de nivel 3 a nivel 0. En la pila de nivel 3 pusheamos los 2 parámetros y luego hacemos el CALL.

Una vez que hacemos el CALL, en la pila del procedimiento al que llamamos se guarda la dirección de retorno (CS:EIP), los parámetros, y la referencia a la pila anterior (SS:ESP). 
¿De dónde se saca la pila a la que vamos? Del TSS. (SS0:ESP0).

Si usamos segmentos conforming, no se produce cambio de pila, porque el código se ejecuta desde el nivel que lo llamamos.

![teo10-3][3]

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

![teo10-4][4]

<aside>
⚠️ Una buena práctica puede ser setear el PDE en User y después decidir para cada página si querés que sea User o Supervisor. 
Porque si ponés el PDE en Supervisor, ya todas las páginas van a ser Supervisor.

</aside>

---

---

Faltan los últimos 2 videos (20 minutos).


[1]: https://lh3.googleusercontent.com/bgiTQpqu9OMK4pl0G7p1R8KI8YeKAbUSarpS1zKEA5sDg61sG5u5V1HL41pLuOmTznETVuUMnb2Q3XrFQPOMUAH9hIQaGE4iddnJo-KMydMtcc7SpQRx4p4aP8LPzqFo8IXHnhf5HM37jwsieBASs2LRF2zfOrU6rcsNFeKHeYGp0f0K9l9TCJWPE1yTmpr-SLSMnrjsPIJhIhc45aQSuSDDkoMbxAjMCrUPYqPymAkDuulmHj_dGMFZMiucnhVeE5v1MDnSO_g8HXTtU7pLzNz5Te_dRJpAmP2TOc5FCaAJcTZpP6oAogn6S-2oPWv9JM0dRAfBgujvLLZtcbMu7wyF1h-z2j_zUh8cKxq3YtMcnOCces1qUT-NWCckseNbWcxt8cJ4XJn7Eq-_Zouljkag4rjME20tOAjZaNA1B6PaN085FXgXF1LItvORKAztd56do4GzGwLGUFQ22NL-hWMtfQI35t3VbwcxOV_Cq9wggyMOBIL0I3-lBSYt43C4tPcRrdHrvDVHl1IdDGONyb0E-BJXCDRi7U_Y78B2-KxfYjvN0wGn1h4HOI4oSPqNzWaF69KRFYjd0MDbNoOfp833Mgk42DVmqu6zT3VOq8IZVAVc8xJu4gE4dK1D5B-2EgG1h--ey5CN0IumfAkodYPYM0yfhFzHonmRa6UtSlG_esPd68neYaLUiWZMYEEg5DK7Qh-VoL8idFKptosAxRUpdFRi9nz9OtsfIRgxVNMtgiMvP_O--NK1DCGNTZzzU7yI2ZJ_YJHKAdtjCy97IRtRmFD4oJbgm2RN0204gIFnw1nYfY5-j561FjhmgntUVzb9SG30hWP_5w7Mlmxxc7pTQ0S4UuAuBAQH-gDApw=w714-h664-no?authuser=3
 
[2]: https://lh3.googleusercontent.com/MMLHU1L9EmfNUyh5n8m7aWLoKb1qZzgTie1Jb2rWF5GIGur4TbI2UU-dyfthjgiOyiY8iuItOXSahYJ10-TUPDk6UBXPOyTZ22Rb9CFE4JWd_kztb1O_FR9rvQ2UBysNvnGLVnMSy0ju1jZbQnhBNd8gIybWotMa98eXV2hvTbIEg6dvWOQ84H4B6H2o2wBQ_zbUrVSWs559T9vOaXbP3P1-1TMBN6sUfvbRa-IO_umbbMnyg8OtFDqHCnoLpUyECVM0z-SlCQmAmfPbo1ynhhvLBw7aZMOiGdifW1SzyLeMuJ3YLdihpSsZyTbT6OL-XiOwJv8gQHtYMxuEqOSc0tKmNB2wfmPK4bWXQFqYZ8YHDDqWTGtsvAJ2qDro2D3NQi-nYoXm9WtLxh3hxKL1xkb6vNLTv09jNGKCW0NVbVvF49d_U9VloiOFBy_34jafhMHgEHRt5CNFwCC6KPWJmuyUoAzXW_WxfHPPZ88zsloeSBBPu4pGGlCWtCspFVXyVX3Ftv6Ab6f2qwcacX1Inrxiip4Z9gdringIGQZh6NiJLQ0rdZYU13W1kKugfaZ9jWI9TdADbXWMmeifXVuRGuvg90Hr1mTd0tG0uGfzdtddVrWhvYkTpNjl6APmQ24g8oikyJrResckM_vpLr80rvjelykzrx5bFocoDTiuOYgM9hXY9wT4RyVfhEPpidiaI7QGvLcOHTlBVyWjhNNa9nxSXECYkLozk7_enEocS12cL7ZN_xTzYmV8JRQkqNkA5kp3VdOvAJA-DlEF71jvfj5zFcJL0OvkOsLUtZn0bXyjeJE0pZeWtVoz418OcVRZpPgLPwU-OhdaR2dyM7g1pyew-JCr6HzfO7yDQsNb4g=w820-h620-no?authuser=3

[3]: https://lh3.googleusercontent.com/hxK-bFjG1mFvbxjQwWjLtRgAYlVC7-UPnjzukAbs4w5PjRveMnlTmkVmnaIgDm8WoZoTlIOqh5ktoSOuk0IqN3ysbLA_GS07FJIN1XWRPz37QdGR3hVcjuSgBxTDzOYZwU8GsUdc2TLtli9gU9lksI70Cd8vkdLkuwCDOKPIhVTVbk60_p-C8oCCM-JFBweCodK53tF_rCgStb8-WXg77x8MUVP6G8oliOpI44OPCIyHW50GMcj79i8PBznsW7_bj7pqqsWFV51tdqgkGwKhPA1Edh18gPgRY02TuMiJBzCB3ZHGFSbXciPfIzWJqjs8KUBziXSMTtWdgeDgNOnWko00VsBhqCX3WypmIfQpdKdzhU3_wqrSh4ERKJzAZCBIKvgBSy0Xwkms0cqzxN65K1YB3spWQLP1d7jCg8XiQvIx2Vo_8c-nAX64hxLDMKuYvoP_pk8vDj8JKBRY1uBs0MahKIfBkY-DY3DzdIAphhf6PfHUwA0WzICFjU_VQlXEPs7O2yyjVAGfhxU62SVL2aL0-Yze4Erp9d3LLn3CcT8r1QkYNszM3x4LNqq7t16wY_Nx7IqanRgCPOv_mxnRG64vHaqMeXKuOX8sbmBc7QESMc1rj9UmYQTEg0dTvd2osCspHL3vVjAWY0s-U5qU3BctPq7ue7cFudJFXpxfNh87pvj74qrzQMy5yp5tLgjlev6LPD59d2uDUmI_uYqCM4I-qjt9OS1dQG7VfZYFjbIj-l4WuAdQpE7qxi6jJq63OsJW-AI9GnWbZ3eIAtvy5YorKFDhhQD3iSidY7LF5aTOEgbGWltmj6hdabMMM4vz0HnZBfbufwaa6c3-20gnTxmPmwixu5C1dXG3ECWT-g=w1012-h650-no?authuser=3

[4]: https://lh3.googleusercontent.com/BsNk6CqPNLMmhl4zqi2nfDTJt4cAa2Wk8Csy3SU6msSJ4d2pjR6bYO0a8y2ZGu6avHb0xJUz5MJMunJBnjg5VBYIYtHFrITF3UqdT6onVxGBLsRfEM8Ypix4HmP4nnWPaZK_7HVghtlFUboFgHaeKbjJ0UdA3xhVo011ZpKpdjQehAKfUi3sxr0oB0TDWXDJhWGImZ89xf2pt5XJS4fQ286wA4q5hsrzwMJT85nTo5RQWlhNe642ratKmjK6sXK74ecV8XXmsP0hT0JcI2865NY20E-U8Oc-ZiobrgG61WGCVi2CULq4bAbmwxHAqSx9TB6_OBcJBvIHTegfRvAKTO9YGOnp0MNETiwrSHRYMEZnpN8NMvVdGgfwb6YpyPXlqurPYQ0gTWbORTBcUS8zrSCBeE13o3A6Zvo6VslXKcCTZGJyMblK--w3yl2Yguz3rKR9iV6vGYiAdSmQJ1Vzq3tgc0HhcNf5JSy6BGWBIE3CXwgXDMMwgf8LsG7oNdcCwWdq_9Kb-mEibzbzsGEaLgcychvwO0QExHE-vpZr4z8-Tk9uUtzNtvfnO_u-zShwE29VMy_Xt0XvXTl8kp6A8PiMz3jQXAwkUnsljteongEhO4SKnpxWPEvb_exF4eMWdl--HU8DJQSOE_mSRF7FeDHxRileO9UFNZwm8nXuFS4yX7O6p8cc8ScNxj79v5NoywZoANyNpbHH-z0XtdLHsZ6bNA2dH-wnXGIRITD3VUWaGHGGk1xsvSXtGaacQcv8US0n1YaXe-poaBwlGXzZDoZerUj-UQsUMYw0uApFCs63IJ68HlIqfcpxV13XBhY0mIdwqidCbW6OX632Qfif8r5EIwgz0_2aQ_zsHjkZPw=w996-h650-no?authuser=3

