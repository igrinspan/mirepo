---
layout: page
title: Teórica 9
permalink: /materias/orga2/teo9/
---
# Teo 9 - Orga 2

# Tareas

El tema que vamos a ver hoy "suena más" a una tarea del S.O. que del procesador.

## Definiciones generales

- **Tarea:** unidad de trabajo que un procesador puede despachar (colocarla en una lista para buscarla después y ejecutarla), ejecutar y detener a voluntad.
    
    La tarea puede tener la forma de:
    
    - La instancia de un programa o proceso (o *thread*).
    - Un handler de interrupción o excepción. Podemos asociar a una interrupción determinada a una tarea.
    - Un servicio del S.O., por ejemplo Malloc. No suele ocurrir.
- **Espacio de ejecución:** es el conjunto de secciones de código, datos y pila que componen la tarea.
- **Contexto de ejecución:** es el conjunto de valores de los registros internos del procesador en cada momento.
- **Espacio de Contexto de ejecución:** un bloque de memoria en el que el S.O. almacenará el contexto completo de ejecución del procesador.

## Context Switch

Es lo que vamos a hacer para pasar de una tarea a la otra. Consiste en salvar y restaurar el contexto de dos procesos, o *threads* cuando se suspende la ejecución del primero para pasar a ejecutar el segundo.

Puede incluir, o no, guardar y restaurar el espacio de memoria o información de control del proceso y otras cosas.

Por esto podemos decir que las tareas de un procesador no operan simultáneamente, sino serialmente pero cambiando de una a otra en gran velocidad. Se producen miles de context switch por segundo y por eso se crea la sensación de simultaneidad.

### Rol del Sistema Operativo - Scheduler

El S.O. tiene un módulo de software llamado **scheduler**, que trabaja con una lista de tareas a ejecutar. Define un intervalo de tiempo llamado **time frame,** que es dividido en muchos intervalos más pequeños, que van a ser las unidades de tiempo.

El manejo de prioridades consiste en asignarle a cada tarea un porcentaje del time frame, medido en cantidad de intervalos unitarios.

Entonces cada tarea tiene una cantidad de tiempo para progresar, y si se pasa se suspende y despacha para ejecutar la siguiente.

Hay un porcentaje del time frame que no es para las tareas, sino para el context switch (porque se necesita un poco de tiempo para guardarse los valores de los registros del procesador y luego pisarlos). Ese tiempo que usa el context switch es siempre el mismo, independientemente de la tarea.

Video 1: todo el proceso explicado.

![teo9-1][1]

## Modelos de Programación

Hasta ahora estábamos trabajando con la vista de una arquitectura basada en el **desarrollo de aplicaciones.** El conjunto de recursos con el que trabajamos es el 
"**Modelo de programación de Aplicaciones**".

Este modelo describe: los registros de propósito general, las instrucciones, los tipos de datos, los modos de direccionamiento. Pero hay más.

Si queremos desarrollar un sistema con tareas, que las almacene, administre, regular accesos y demás, la visión del programador de aplicaciones no es suficiente.

Entonces hay que desarrollar un conjunto de tareas que contribuyan a implementar el sistema que queremos, y un software que administre esas tareas.

### Modelo de Programación de S.O.

Surge el "**Modelo de programación de Sistemas**", ****que tiene recursos nuevos de hardware que amplían la capacidad de la CPU. Esos recursos le permiten al Sistema Operativo proveer servicios y cosas de manera mucho más veloz y eficiente.

Las interrupciones y excepciones, por ejemplo, están en el terreno del Sistema Operativo. Hay muchas funciones del S.O. que no van a ser necesarias para las aplicaciones.

### **Scheduling de Tareas**

Cualquier S.O. posee un módulo llamado scheduler, que administra la ejecución de tareas.

El scheduler se divide en 2 capas:

1. Una de alto nivel en la que se implementan políticas para manejar el cambio de tareas, que son algoritmos de scheduling. **(C)**
2. Una de bajo nivel en la que se realiza el context switch. **(ASM)**

Para la **capa de bajo nivel**, el context switch debe llevar a cabo las siguientes acciones: 

1. Resguardar los registros core y cualquier otro del modelo de programacion de aplicaciones, en un área de memoria **sólo accesible para el kernel.** Así los protege del resto de las tareas.
2. Determinar la ubicación del TCB (o PCB), **Task Control Block,** de la siguiente tarea en la lista de tareas en ejecución. 
    
    El contexto forma parte del TCB, que es una estructura.
    
3. Extraer el contexto de la nueva tarea y copiar registro a registro en la CPU.

Por otro lado, **la capa de alto nivel** es la que llama al context switch, en base a los siguientes criterios:

- Prioridad
- Preemption
- Atomicidad de operaciones
- Concurrencia
- Limitaciones de tiempo (latency)
- Paralelismo, si hay más de una CPU

Vamos a tomas un approach bottom-up. Primero nos vamos a ocupar del Context Switch, que usa el HW. Luego de la capa superior, pero no aún.

## Task Switch en procesadores Intel

¿Con qué recursos contamos para manejar tareas?

- Segmento de estado de tarea (TSS)
- Descriptor de TSS
- Descriptor de Puerta de Tarea
- Registro de tarea
- Flag NT (bit 14 de EFLAGS)

### TSS - Task State Segment

Es una estructura que vamos a usar como espacio de contexto de cada tarea. Está linkeado a una tarea, pero esa tarea no puede ver qué hay adentro.

![teo9-2][2]

<aside>
⚠️ Guardamos el CR3. Eso es porque cada tarea tiene su CR3. 
Y esto es porque **cada tarea tiene una estructura de paginación propia.**

</aside>

Cada vez que se crea una tarea, al parecer hay 3 pilas, de distintos niveles. Este segmento, típicamente, se crea en la GDT. Hay que ponerle como límite mínimo 103 = 0x67.

¿Por qué nos guardamos el mapa de Entrada/Salida?

En Modo Protegido, una tarea de tipo user (menor privilegio) no puede ejecutar instrucciones de acceso a Entrada/Salida. A determinadas tareas se les habilita el acceso a determinados ports de E/S.

**Descriptor de TSS**

Tiene un descriptor porque es un segmento. Es un descriptor de sistema.

![teo9-3][3]


- El bit **B (busy)** sirve para indicar que la tarea está siendo ejecutada.

El selector del TSS de la tarea que está siendo ejecutada está almacenado en el registro TR. **Task Register.**

Otra forma de acceder a una tarea es invocar un **Task Gate**. Están los Task Gate Descriptors, que se almacenan en la IDT o GDT, no entendí. En este caso, cada vez que ocurre la interrupción asociada a este descriptor, se va a ejecutar la tarea correspondiente. Por ejemplo, Linux hace esto con las doble faltas.

### Despacho de Tareas

Despachar una tarea es lanzar la ejecución, o sea detener la tarea en curso, salvar su contexto y agarrar el contexto de la nueva tarea.

El procesador puede despachar una tarea de las siguientes formas posibles: 

1. Por medio de un CALL (far)
2. Por medio de un JMP (far)
3. Mediante una llamada implícita del procesador al handler de una **interrupción** que sea manejado por una tarea. 
4. Mediante una llamada implícita del procesador al handler de una **excepción** que sea manejado por una tarea.
5. Mediante la instrucción IRET en una tarea cuando el flag NT es 1 para la tarea actual.

Para cualquier método, lo primero que hay que hacer es identificar la tarea a la que hay que saltar. Por eso en la GDT hay que tener un selector que apunte a una Task Gate o a un TSS. 

Este selector debe estar en la correspondiente posición dentro de la instrucción CALL o JMP. O sea cuando hacemos un JMP, en lugar de poner un selector de segmento en la dirección, ponemos un selector de TSS o de puerta de tarea. Es más fácil llamar directo al TSS antes que todo lo otro. Cuando hacemos esto, como offset podemos poner cualquier cosa, ni la va a tener en cuenta.

### **Proceso de conmutación de tarea**

¿Qué pasa cuando hacemos un JMP o CALL o IRET o cuando ocurre una INT?

1) El procesador analiza el valor que debe colocar en el registro CS, y busca el descriptor en la GDT. Se fija el tipo y los atributos del descriptor, para ver si es de código, datos, si es una task gate, un TSS, etc.

Si es un Task Gate, vuelve a buscar en la GDT para encontrar un descriptor de TSS. 

Por otro lado, si lo que se carga en CS es un selector de TSS, se busca directamente en la GDT el descriptor de TSS.

2) En TR está el selector de TSS actual, que debemos guardarnos porque vamos a pisarlos. Entonces nos guardamos el descriptor nuevo de TSS y el selector nuevo en registros temporarios.

3) Almacena en el TSS actual el estado del procesador.

4) Se pone en 1 el bit CR0.TS (Task Switch).

5) Como ya guardamos el contexto en el TSS actual, cargamos el nuevo TR y descriptor.

6) Copia a los registros de la CPU el contenido almacenado en el nuevo TSS.

### Anidamiento de tareas

Consiste en que una tarea llame a otra. No saltar, llamar. No JMP, CALL. Porque es JMP es un viaje de ida, pero cuando hacés CALL tenés que retornar y por ende guardar la dirección de retorno y todo el contexto.

EGFLAGS.NT = Nested Task.

Las tareas que están anidadas dentro de otra, tienen ese bit encendido. Además, en el TSS de las tareas anidadas, hay un puntero al descriptor de la tarea que las llama, abajo a la derecha.

No es muy recomendado hacerlo 😕.

### Contexto completo

Falta guardar cosas en el TSS. Sólo guardamos los registros generales y eso, pero ¿dónde guardamos los registros XMM, los de la FPU, los MM? 

Son registros mucho más grandes que los generales y se usan mucho menos, pero igual, si los estamos usando, deberíamos guardarlos.

¿Cómo hacemos?

Cada vez que el procesador conmuta una tarea, setea el bit CR0.TS.

Por otro lado, cada vez que se va a ejecutar una función que utilice registros de la FPU, MMX o XMM, el procesador chequea CR0.TS y si el bit está en 1, se genera una excepción #NM (tipo 7). 

En el handler de esa excepción, se hace el switch de la siguiente manera: 

1. Limpiar el bit CR0.TS
2. Instrucción FXSAVE para salvar todos los registros FPU/XMM en un bloque de memoria de 512 bytes.
3. Instrucción FXRSTR recupera el bloque guardado oportunamente y lo mete en los registros.

La interrupción se genera **justo cuando los vas a usar.** También se guarda, no sé dónde, 

Quedan los últimos 2 videos.


[1]: https://lh3.googleusercontent.com/xmLFoRlruqaZrj6Y123BnOJEhwYWQjcA1PAP-kUKPUuBQvlZ2Tym41fYDhOTwffD0npluN_1Sxm9LVKju-X7XlIj1V_Ar-m73ihcB0Dy1zI_EjT9WMtAv4aTzGkU0kmGzDdeDoM9wcS5mSAI_xvvL3U5WMGiB5zNJHab55zEQ3kAEHQZ8qxTL4ujieFT_Sg3Gi6hRvXDr8R54Qc0eNItpO5mOyUTNr3WWY2NK1obMgDrDIglZ9VmVz0B_DOkfz1L51K2EdH-RovO9q56XnVx4p64p-j_WdoaERUPqdX_l1PtTLSIEkov4HY-CG6uTzJU4gw3z3mUyYvrJsQtg2mt9b2YBbmrXVfEqleFb8iErmjr7zjbDNgpCoxn46hRY4cUhJeZtVjg-dPGJasZno7kn8TvXXX_690N2CJ47NwQiPb6kGIqnAr1o4ArlmxYw2M2gGgtjC73zcjXfrePoaBCOMSe3iICyn4Llrrc1OCXdBteTuJ6d1VuzH0GU4z8EUmPfByU3u9mCuko0Lp_UXvhHQeiJEY-UEElFSq6jmjZVQ-u9nIfNVuKufq-tflBOL7-OppqqMqJevvhH8O4_euTSMoWbqa7PeQoX__1pTBjBb1mwgXK7iqAAjO1ov3l3alHn445clzASuQHN0LbdVfuV0Hrc4Sbe6IVp5N_5Tr_fgDWBQuq01fBiX4DyXC5lEiVUiBY4s_cWM7WLYc6I6JkWDCNXpEM8fLhwy81_X6dipYd66l-mlYJ7KiK3GHTgMPl_RRimFkuQP6qbPtO-HPkjfMyj52Y3usYl6C6iYmXdB08LNUV_LKCXFqFmYJkOtWiG5Evv3-m7kST037WqrhpJWLKUDZ2Xqbo5JEeFEUX7w=w1098-h198-no?authuser=3

[2]: https://lh3.googleusercontent.com/d_ZsmRAgBGSMYn2MLXJUPgqXnFbK_tak5h538d8w3ehWDQKOiahS2vBcTkiQzU3lRldfp0dPhFDUzhKPX6biUfMiG3W0RBvbzDkRM6hZ5I-oCTUJMalT4LKrny4C7dMIdcIxKuLZpROZyLNzfTxB5rJRQxzlC16Pok7DTah1yXkhgmCtLWHG5Iaor0L2gdArkbD87Hb9B0dSgvpAcTYmnUEF4iVsuLESS77NQL7PG9HRPUW9E7fs0rtZTVg-fBFGWt6N1LPSp7zrYbL-GMG5sylgq_kxltn3i_CRhajmuwq7jo30JB43v8zIv4QQ6iJ1Skz_xeb_9azhP-Zd-BLe6UvVXJCGCXMjyWWq4vS76m9UYlN2c21Di6HhvNQAYhOrQDSX5g7lbcfToxih_ZBwn9FoAtVifNA7ZXyifQ2Fae4PqtY4ydMBddsCqNiiNyQuxXnbe_PzWzPOg8NB6tnvGLjq6MG2BtiFNjINMzqLtVIfLvZUhdcu0Gg7Tb7SmQMU-FA1kkvV0VdwvLYN0ZPrQ4dARMLAhMPzxJteDbb1eVRse1RLfv2SusARvcXhvjZQFmQ3n25TET4DIM6-rmjxB6kSSjDwZiIqN5EC5rsNpDVraVncOSGcgJjBYXBOa6zd7i4Hi_fwGAftbUDwNunsWsRmYIpGWBb3BeKX9ReBJ4nTbxtetHZdffPDmSjFj8gPIhcYZK7RaCzeH-cHQax_hFds7q9vohM_hyRpV9uiyXOvfV6gG_SoKolcQiy9I3hgTm_6mNovYC_GNALzncGGROlumqrgkE_n0IwcGP789uKxpe1gCWXqJbvXh9HIFinBD1NbFuxYwKWAGLtPOP8IfYtVunTu8_oPjG9Vj4OCfQ=w1110-h702-no?authuser=3

[3]: https://lh3.googleusercontent.com/iTfEYIfwQ7WZFA4gXhMlCCfs3bLPLnChYzZfvonJgnK1X8H6SMxPGNweF9aeYZRmO07Ck0pbXZZI4t3ms8wpx8QURQvM7d4kCez6ILnFROfUOX1Re5sosvxvdp01OtLJRNwo_Ne9gndqsnA9QFUR2aLvXlC_zp0jSf4YJy4zz-nJ4uGa1QDkaD8GIk_FOK841TBLJtq8F4R9_WFFCKbDpjsc1NTA2t2xDzromJnmaTM6Yl4Hjfl8tgkssjk2RztJTkMBKzLdeCWg5_lxwZSWilPMGJgXPbF8lv5K3IogAOoOkRPBAVvi6iwSBYGe6Hr19odE4eDYEbWiDj6nzDj6jM_F4j8X6tQ3jKF2SxjvZ3xJ4GdsOHWaiRUBCZiDYzRCOPrCUbEVhYK3Ai1vYU9R8WPQfMO664Und-arD4VfMJTVfO5WVw2uEmG5jyO9uGA1ldJMpUtqDOWHb_wQ_ymxuSTyyAANWAWCGf2918dlWEWh25RGqB1_zBiHtV9BPVsQOoM5vygFH10GnbAahFoftdoRcQqvpVtYRGRwMQZ5LQGm7WSUxWC8oBtakN3yBvH4zAh3nyYHSD93S6s9UVcw11K4z4LF3kZ2nWBoy0lsmNCwYsqUmAQc7FbbCfyCE9mZeEx0D8VMRTtlGi6AJspT8pZgyLW-atob16O0CtXZOBYv2c-_xHz27kfsmBkB8xbGCWqE6mUuPJpnlyiHv7M780YYJ5RKHv2JEu00osn6iOpPUqSb5y70yW5FGYhbapNCKA2pozio1eurS9PYBRE-8CkxvVTgG-cdMcqUlzDKSYv4sINkM31TOXboctOpldmzbihAZEMNPKjqlacaE3BWat9-f3SfwyFe4TlzEVGZ0A=w1126-h646-no?authuser=3
