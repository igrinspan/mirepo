---
layout: page
title: Teórica 7
permalink: /apuntes/orga2/teo7/
---
# Teo 7 - Orga 2

# Interrupciones

### Fuentes de Interrupción

- ***Hardware***
    - Señales eléctricas enviadas desde los dispositivos de *hardware*, concentradas en un controlador externo que las prioriza y envía a la CPU secuencialmente.
    - Son **asincrónicas** al *clock* del procesador. Y nadie puede saber cuándo va a llegar una interrupción, son **no determinísticas.**

- ***Software***
    - Se producen cuando el *software* ejecuta INT n.
    - Son **determinísticas**, porque si ponés INT en el código, sabés que ahí se va a producir una interrucpión.
    
- **Internas (excepciones)**
    - Las genera la propia CPU cuando hay algo que le impide completar la ejecución de la instrucción en curso.
    - Ejemplos: división por cero, *page fault*, violación de protección.

**¿Cómo se identifican las fuentes de interrupción?**

Una interrupción se identifica unívocamente mediante un valor de 8 bits llamado **tipo.** Intel maneja **256 tipos**, todas las combinaciones de 8 bits.

1. ¿Cómo se obtiene el tipo?
    
    Es provisto por el HW en interrupciones que ingresan por el pin **INTR** del procesador.
    
    Acompaña al código de operación en la instrucción **INT type,** para el caso de es SW. Por ejemplo **INT 80h**.
    
    En algunos casos está asociado a una instrucción. Por ejemplo: **INTO** es una instrucción que genera una interrupción tipo 4.
    
    Cada excepción tienen asignado un tipo específico.
    

Una vez que identificamos la fuente de interrupción, hay que vectorizarla. Esto es suspender lo que estamos ejecutando, guardar el estado del procesador y que se pueda atender la interrupción, "saltando" hacia el código de su rutina de interrupción.

### Vectorización de las interrupciones

### Interrupt Descriptor Table (IDT)

Cuando se trabaja en Modo Protefido se define la tabla IDT que almacena descriptores de interrupción. La tabla tiene 256 entradas, una por cada tipo. Los descriptores miden 64 bits → la IDT mide 2k.

La **IDT** puede contener 3 tipos de descriptores:

1. **Interrupt Gate** (de 16, 32 o 64 bits)
2. **Trap Gate** (de 16, 32 o 64 bits)
3. **Task Gate** (sólo en 32 bits)

En los 3 casos se trata de **descriptores de sistema (S=0).** 

<aside>
⚠️ Si metemos descriptores de segmentos de datos o de código u otro de sistema diferente, el procesador generará una **excepción de protección general** al intentar vectorizar la interrupción.

</aside>

**Formato de los descriptores en arquitectura 32**

![Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled.png](../teos/teo7/Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled.png)

El Task Gate lo vemos después.

Abajo están el Interrupt Gate y el Trap Gate. La única diferencia son los 3 bits (8, 9 y 10). 

**¿Cómo se llega a la IDT?**

Tenemos el IDTR, igual al GDTR.

![Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%201.png](../teos/teo7/Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%201.png)

**¿Y cómo vectorizamos la interrupción?**

1. Se genera una INT n.
2. Vamos a la IDT a la puerta para INTs n.
3. Ahora tenemos el descriptor.
4. En el descriptor leemos el selector de segmento al que tenemos que ir. Puede estar en la GDT o LDT. 
5. Ahora lees el nuevo descriptor de segmento. Hay que verificar que este descriptor sea de código, porque debe ser una rutina de atención de interrupciones. Sino, general protection fault.
6. De este descriptor sacamos la dirección base del segmento que contiene el código de la rutina. Y del descriptor de la IDT sacamos el offset a la primera instrucción.

**Manejo de pila**

Nuestro ESP apunta a un lugar determinado. Cuando nos llega la interrupción: guardamos los flags, el CS y el EIP.

 

### Excepciones

Las excepciones se dividen en 3 tipos:

- **Fault:** es una excepción que tiene solución. Puede corregirse y retomar la ejecución de esa instrucción sin perder continuidad. El procesador guarda en la pila la dirección de la instrucción que produjo la falla.
- **Trap:** excepción producida a continuación de una instrucción de trap. Algunas permiten seguir con la ejecución, otras no. El procesador guarda en la pila la dirección de la instrucción siguiente a la instrucción *trapeada.*
- **Abort:** excepción que no permite recuperar la ejecución ni la tarea que la causó. Reporta errores de hardware o inconsistencias en tablas del sistema

Cuando la interrupción es una excepción, el procesador guarda en la pila, además de los flags y eso, el **error code**. Algunas excepciones tienen y otras no.

¿Para qué sirve el **error code**?

Da pistas acerca del origen de la excepción. Tiene 32 bits, pero sólo se leen los 16 menos significativos.

![Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%202.png](../teos/teo7/Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%202.png)

1. **1**  *EXT:* External Event (bit 0): 
Se setea para indicar que la excepción ha sido causada por un evento externo al procesador.

2. **2**  *IDT:* Descriptor Location (bit 1): 
Cuando está seteado indica que el campo Segment Selector Index se refiere a un descriptor de puerta en la IDT: Cuando está en cero indica que dicho campo se refiere a un descriptor en la GDT o en la LDT de la tarea actual.

3. **3**  *TI:* GDT-LDT (bit 2): 
Tiene significado cuando el bit anterior está en cero. Indica a que tabla de descriptores corresponde el selector del campo Indice: 0 GDT, 1 LDT.

---

Las Interrupciones y Excepciones, en un sistema multitarea, las maneja el **kernel.**

### Tipos de Interrupciones

Más importantes:

- Tipo 6: **invalid opcode exception #UD**.
Esta excepción deja al CS:EIP apuntando a la instrucción que generó la excepción.
- Tipo 7: **device not available exception #NM**. Responde a 2 situaciones: Ejecutar SIMD o FPU con el flag CR0.TS = 1. Ejecutar SIMD con CR0.EM = 1. Deja al CS:EIP apuntando a la instrucción que generó la excepción.
- Tipo 8: **double fault exception #DF.**
Ocurre cuando ocurre una excepción mientras se está vectorizando otra excepción. En general, el procesador intenta manejar las excepciones en serie. Pero en ocasiones no se pueden procesar en serie, lo que produce una doble falta.
Podemos saber si se pueden o no procesar serialmente, porque estas excepciones se dividen en 3 categorías: Benign, Contributory y Page Fault.
    
    Si la primera expeción es Benigna, la segunda se va a poder procesar en serie. 
    Si la primera es Contributiva, la segunda generará #DF sólo si se trata de una Contributiva. 
    Si la primera es un Page Fault, sólo se tratará en serie si la segunda es Benigna. 
    
- Tipo 11: **segmento no presente #NP.** 
Se produce cuando se intenta cargar el descriptor seleccionado por CS, DS, ES, FS o GS y su atributo P es 0. También se genera si LLDT catga un selector de LDT con P = 0.
- Tipo 12: **stack fault #SS.** 
Se genera cada vez que una instrucción que utiliza SS para direccionar memoria viola el límite efectivo de la pila (del segmento pila). También se genera si se escribe en SS un selector cuyo P = 0. Pone código de error en la pila.
- Tipo 13: **general protection fault #GP.** 
Engloba todas las violaciones generales al sistema de protección que no estén comprendidas en Stack Fault, TSS inválida o page fault. Tiene código de error.
- Tipo 14: **page fault #PF**
Engloba todas las violaciones de páginas (violaciones al sistema de protección relacionadas con el Sistema de Paginación).
Tiene 5 causas generales.

## Hardware para las interrupciones

Intel usa el PIC (Programable Interrupt Controller).

![Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%203.png](../teos/teo7/Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%203.png)

Puede controlar 8 dispositivos de hardware. Las interrupciones le entran al Interrupt Request Register. Este, junto con el Priority Resolver y el In-Service Register, se comunica con el Control Logic. El Control Logic le envía la señal INT al procesador, para avisarle que tiene una interrupción. El procesador le responde con la señal INTA cuando está listo para atenderla. 

Hay un buffer que nos conecta al bus de datos. Y un buffer y comparador de cascada. Este último es porque el PIC está pensado para conectarse con otros PICs, y dice si este PIC es *máster* o *slave* según el caso.

Actualmente se usa el APIC, que es una versión avanzada del PIC.

### **¿Cómo trabaja el PIC?**

El IRR es un registro que simplemente guarda el estado de esas líneas, y avisa al Priority Resolver cuando alguna de estas líneas se activa. El Priority Resolver resuelve, cuando hay más de una línea prendida, cuál priorizar. ¿Cómo prioriza? En general IR0>IR1>..>IR7. También puede pasar que haya una interrupción en curso y se produzca otra interrupción (**añidamiento**). Eso también lo resuelve.

El priority resolver se fija en el Interrupt Mask Register (IMR). Este registro lo maneja el procesador, y dice qué Interrupción se debe tener en cuenta. (Pueden estar habilitadas IR4 e IR5 y el resto no, por ejemplo). Además de eso están los flags **dentro del procesador**, que si están en 0, el PIC puede hacer toda la bola pero el procesador ni atiende las interrupciones. 

El In-Service Register (ISR) pone en 1 el bit de la interrupción que está en servicio. O sea, si el procesador está atendiendo una IR3, el bit 3 del ISR debería estar seteado.

Una vez que la Control Logic mandó la interrupción, espera pulsos por la entrada INTA. (En particular 2 pulsos).

El procesador está en la suya, con el flag de interrupciones en 1 y recibe una interrupción IR1. En ese caso termina la instrucción en curso (y si es una instrucción atómica, también la siguiente), guarda su estado y envía un pulso por el INTA. 

Hay dos modos de trabajo: **fin de interrupción automático o no.**
En el primero hay un solo pulso de INTA y usa añidamiento full o algo así. 

**Diagrama Cascada**

![Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%204.png](../teos/teo7/Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%204.png)

En cascada, el PIC máster es el único que envía interrupciones al procesador. Se conecta los PICs slave en las líneas más altas (IR7) del PIC máster.

### ¿Cómo se inicializa el PIC?

Vamos a tener que inicializarlo. La PC está programada con **fin de interrupción NO automático.** (Es una de las cosas que vamos a tener en cuenta). **Se activa por flanco, no por nivel.** (otra cosa).

El dispositivo se inicializa mediante una secuencia continua de entre 2 y 4 palabras de inicialización. La secuencia se compone de ICWs (Initialization Control Words). Cada ICW es un byte que setea determinados bits relacionados con el funcionamiento del dispositivo.

- Dependiendo de cómo lo programemos, vamos a tener 2, 3 o 4 ICWs.
- La ICW1 y la ICW2 se envían siempre.
- La ICW3 se envía si tenemos más de un PIC en cascada. En nuestro caso tenemos 2 PICs, entonces hay que usarlo.
- También vamos a usar la ICW4. Esto se indica en la ICW1.

![Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%205.png](../teos/teo7/Teo%207%20-%20Orga%202%2077dc494215664e3f8e5bd3109d09b5fe/Untitled%205.png)

---

Diagramas de las ICWs. (**pag 102 pdf, diapo 84**).

**ICW1:** 

- A0 dice en qué dirección de entrada/salida del PIC se escribe este ICW.
- Bit IC4: indica si la necesitamos (1) ó no (0). **1**
- SINGL: cascada (0) ó single (1). **0**
- ADI: para un procesador viejo  de 8 bits, no se usa.
- LTIM: flanco (1) o nivel (0). **1**

¿Cómo detecta el PIC que lo voy a inicializar? 
El dispositivo detecta el comienzo de la secuencia de inicialización cuando se le escribe en el address 0 un byte cuyo bit D4 está en 1. Obviamente, esto pasa con ICW1, que es la primera palabra en la secuencia de inicialización.

El PIC máster está en la posición de E/S 0x20 y 0x21.
El segundo PIC está en la posición 0xA0 y 0xA1.
0x11 es el valor de nuestro ICW1 para inicializar. (ver bits).

**ICW2:**

- En los bits T vamos a guardar el tipo de la IR0. A las siguientes IRs, el PIC les va a asignar los tipos de interrupción consecutivos.

**ICW3:**

Tiene 2 formatos, uno para el máster y otro para el slave. El ICW3 hace la conexión lógica entre el máster y el slave. 

- Al máster le dice en qué líneas tiene conectado slaves y en qué lineas dispositivos.
- Y le dice al slave a qué línea del máster está conectado. Se lo decimos en los 3 bits menos significativos.

**ICW4:**

Indica en qué tipo de procesador estamos trabajando y el modo de fin de interrupción que usamos: automático (1) o normal (0). **0**

Luego hay 3 bits para poner en 0, que son de buffer y añidamiento.

---

Cuando termina la ICW4, el dispositivo PIC queda establecido y está en modo operacional. Si en algún momento querés modificar algún aspecto del PIC, tenés que mandar esta secuencia de inicialización nuevamente. 

---

**Modo operacional**

"En el modo operacional estamos ahora para trabajar con palabras que ya operen sobre la operación de esto."

OCW = Operational Control Word. Hay 3 OCWs.

**OCW1:** 

Nos permite trabajar sobre el registro de máscaras. Se pone en el puerto 0x21 o 0xA1. De ahí podemos escribir o leer el registro de máscaras.

**OCW2:** 

Importante porque nos permite mandar comandos. End of interrupt, automatic rotation, specific rotation. Vamos a usar los **fin de interrupción.**

Hay 2: específico o no específico.
El no específico resetea todo el ISR, el específico envía sólo el bit que quiere bajar. En L2, L1 y L0 se envía qué bit querés bajar.

**OCW3:**

Sirve para poder leer el IRR y el ISR, pero medio que no lo vamos a usar. Toda la parte de arriba va en 0. Requiere escribir para después leer. O sea escribís qué registro querés leer, y después lo leés. (IRR ó ISR).

---

---

## Hardware de una PC - Manejo de la E/S básica.

¿Por qué las cosas son como son?

Las PCs deben guardar compatibilidad con la primera. Esto nos lleva a tener un primer Mb de memoria fragmentado entre RAM y ROM y direcciones de E/S que debemos respetar siempre.

Hay una tabla de direcciones pre-asignadas para E/S. 

Programable Interval Timer.

Teclado. Break / Make. Scan, make y break code.

Video Monocromo (un solo color).