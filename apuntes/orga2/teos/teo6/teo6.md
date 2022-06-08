---
layout: page
title: Teórica 6
permalink: /apuntes/orga2/teo6/
---

# Teo 6 - Orga 2

# Modo protegido

### Programación Bare Metal

Bare metal quiere decir **sin Sistema Operativo**.

**Arranque del procesador**

El procesador, cuando se enciende (o cuando se activa su #RESET), busca las instrucciones necesarias para arrancar.

- Comprueba algunos tests y, si todo está bien, **inicializa eax en 0.** Si ese registro, cuando se enciende la máquina, no está en 0, el procesador se apaga
- El resto de los registros toman valores **bien determinados.**
- Se invalidan las caché internas
- Se invalidan las TLB (paginación de memoria)
- Se limpian los Branch Target Buffers.

Otra forma es enviar la señal #INIT, que mantiene los registros de la FPU, algunos registros de memoria y la caché interna. Reinicia el procesador pero NO todo.

- Se inicializa el resto de los procesadores. (En arquitecturas de más de un procesador.)

**VALORES INICIALES DE INTERÉS**

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled.png)

El procesador busca instrucciones con el registro CS (code segment) y el EIP(32)/IP(16). 

Vamos a parar al **Reset Vector.** 

- Los valores iniciales de CS:IP son 0xF000:0xFFF0.
- A la vez, el procesador está en modo real. Entonces, su rango de direccionamiento físico es desde 0x00000 hasta 0xFFFFF, 1Mbyte.
- La dirección física que genera un procesador 8086 viene dada por cs << 4 + ip, que es 0xFFFF0.

**Un problema**: los nuevos procesadores tienen un espacio de direccionamiento mucho más amplio, y por algún motivo arrancan en Modo Real accediendo al espacio de 1Mbyte del que hablábamos antes. Hay que intentar que no se acceda a ese Mbyte de alguna manera, sin interrumpir la continuidad de la RAM.

**Una solución:** al llegar al Reset Vector, que el procesador use algunos recursos de Modo Protegido, aunque esté en Modo Real.

Intel agrega "registros caché ocultos", que están asociados al registro de segmento, y para el arranque se pueden usar en Modo Real.

Los registros de la derecha son exclusivos del procesador. No se pueden tocar. Los de la izquierda son los que vemos.

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%201.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%201.png)

**Limit:** el 0xFFFF es el máximo valor de offset que podés tener, porque en Modo Real, los segmentos tienen por default 64k (direccionable de 0x0000 a 0xFFFF).

**Base Address:** El code segment ¿por qué arranca de ahí?. En Modo Protegido, la dirección se forma con la dirección base + el offset. CS.BaseAddress + IP = 0xFFFFFFF0.

Esto quiere decir que el procesador va a buscar la primera instrucción al fondo del espacio de direccionamiento de 4GB. Ahí metemos la ROM de 16 bytes. En arquitectura 32 bits está al fondo.

Entonces: el procesador inicia en Modo Real con su restricción de espacio de direccionamiento a 1Mbyte de memoria, pero realiza opcode fetch muy por encima de esa dirección. Al estar en Modo Real, no debería acceder más encima de 1Mb.

El valor de CS **no puede cambiar mientras el procesador trabaje en modo real.** En ese caso, el Base Address se pone en 0x00000000 y queda restringido a continuar buscando instrucciones en el primer Mb de memoria. 

**¿Cuándo cambia el CS?** 

Cuando hay jmps y calls far (inter segmentos). Y cuando hay interrupciones.  (NO HAY QUE HACER CALLS Y JMPS FAR Y HAY QUE MANTENER LAS INTERRUPCIONES DESHABILITADAS).

Ahora hay que seleccionar el modo en el que el procesador debe funcionar. 

- **Modo Real**
- **Modo Protegido Flat**
- **Modo Protegido Segmentado** (cambia el modelo de segmentación).

Nuestra ROM se mapea físicamente en el fondo de los 4Gb.

En primer lugar, en vez de poner código, debemos ponerle **directivas al ensamblador.**

```c
ORG 0xFF000  //el origen es 0xFF0000, a 4k del fondo del mega
USE16 //indica al ensamblador generar código de 16bits
```

**Único código para iniciar**: loop infinito

```c
init16:
	cli //deshabilita interrupciones, por las dudas, aunque ya vienen deshabilitadas
	jmp init16 //loop infinito

align 16 // completa hasta la siguiente dirección múltiplo de 16. Esta directiva genera código.
//el align rellena con NOPs, (not operate)
```

En este caso, necesitamos que el init16 sea mapeado por el ensamblado en la dirección 0xFFFF0, entonces hay que rellenar desde 0xFF000 hasta 0xFFFF0. Hay que rellenar los primeros 4080 bytes de los 4096 totales.

Primer impulso: 

```c
TIMES 4080 db 0x90 // generamos 4080 NOPs

init16:
	cli 
	jmp init16 
align 16

```

Resuelve el problema pero no es escalable. Porque si vamos agregando código hay que volver a calcular cuánto espacio debemos rellenar instrucción por instrucción y es un garrón.

Metodo más eficiente: (**utilizando EQU**)

```c
ORG 0xFF000  //el origen es 0xFF0000, a 4k del fondo del mega
USE16 //indica al ensamblador generar código de 16bits

code_size EQU (end - init16) //es una constante, no genera   código.
times (4096-code_size) db 0x90

init16:
	cli 
	jmp init16 
aqui:
	hlt //si por algún motivo sale del loop: HALT
	jmp aqui

align 16

end:
```

—Video ejemplo - BOCHS, 22min.— 

okteta, arichivo list, archivo bin.

Podemos debuggear nuestra ROM con bochs 😎, basado en gdb.

---

QUEDAN 3 VIDEOS (02:30hs).

## Modo Protegido - Gestión de Memoria.

Ya vimos cómo inicializar el sistema. Ahora hay que ver cómo organizamos la memoria. Hay muchas formas posibles de expresar una dirección de memoria. Pero en esa dirección siempre hay un único dato.

**Dirección física:**

Número que representa a la celda física de memoria. Cuando un procesador quiere acceder a memoria, debe escribir en el buffer de direcciones el valor corresponediente a esta dirección de memoria. **Siempre el procesador usa la Dirección Física en los pines de address,** cuando su U.C. habilita la salida del bus de address.

Esta dirección física no es la que manejamos en los programas.

```c
buffer resb 1024 db // define 1024 bytes en memoria en 0
 
mov al, 00 //valor inicial
mov ecx, 1024 //inicializa el contador en 1024
mov esi, buffer //guarda en esi la dirección de buffer
repnz stosb //accede al buffer y lo inicializa
```

Buffer es una etiqueta, representa un desplazamiento dentro de un segmento. Entonces, para **entender su dirección física**, hay que entender que está **asociada a un registro de segmentos**. En particular, en un segmento de datos. (DS = data segment). 

Entonces la dirección es **DS:buffer.**  DS representa el segmento en el que está y buffer es el "offset" dentro de ese segmento.

A este tipo de direcciones que manejamos en nuestros programas, como buffer, se las conoce como **Direcciones Lógicas.** No tienen nada que ver de cómo está mapeada la memoria en el hardware. 

**¿Qué es la dirección virtual?**

Suele coincidir con la dirección lógica. En algunos procesadores no. Se llama también dirección lineal.

### Memory Management Unit (MMU)

- Cambia el mapeo entre una Dirección Lógica y la Dirección Física.
- Visión de la memoria física dividida en fragmentos.
- Permite al S.O. separar los procesos. RTOS (Sistema Operativo de Tiempo Real).
- El MMU le da a cada proceso su propio espacio de memoria físico exclusivo y lo deja protegido del resto de las tareas.

### Administración de Memoria con MMU

2 criterios diferentes de dividir la memoria en bloques:

- **Paginación**
- **Segmentación**

Intel hace una combinación de ambos 🤨.

**SEGMENTACIÓN vs. PAGINACIÓN**

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%202.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%202.png)

Los segmentos son tan grandes o tan chicos como el HW lo permita. Se pueden solapar y son muy flexibles.

Las páginas, en cambio, son todas de igual tamaño y no se solapan. Cada página comienza en el byte siguiente al último de la página anterior. Esta bueno si tenemos memoria de sobra, pero mmm.

**Espacio Físico**

Los procesadores IA-32 organizan la memoria como una secuencia de bytes direccionable a través de su Bus de Address. La memoria conectada a este bus se llama **memoria física.** Las direcciones son **direcciones físicas.**

**Espacio Lógico**

Intel organizó el espacio de direccionamiento en segmentos. 

- 4 registros de segmento para almacenar hasta 4 selectores de segmento.
- Registros de 16 bits porque los segmentos tienen a lo sumo 64k de tamaño.
- Las direcciones se expresaban mediante dos valores:
    1. Identificador de segmento.
    2. Offset o **dirección efectiva** a partir del inicio de ese segmento.

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%203.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%203.png)

Con estas direcciones, el procesador realiza algunas operaciones para pasarlas a dirección física.

### **Traducción de direcciones**

Los procesadores en Modo Protegido general una dirección física mediante un proceso de traducción de 2 niveles:

1. Traslación de dirección lógica (a lineal)
2. Paginación del espacio lineal (a física)

El espacio lineal es un rango de direcciones contiguas, como el físico. Inicia en 0 y llega al máximo valor que puede ocupar un segmento de acuerdo al modo de trabajo. En **Modo Protegido de 32 bits**, el rango llega hasta     2^32 - 1. 0x00000000 a 0xFFFFFFFF.

1. Traslación de la dirección lógica:
    - El procesador le especifica a cada proceso o tarea, dónde comienza cada uno de sus segmentos y cuál es su tamaño.
    - La dirección de comienzo de un segmento, en Modo Protegido, es un valor de 32 bits.
    - El tamaño máximo de un segmento es el de su máximo valor de desplazamiento. Es de 32 bits en Modo Protegido y 48 bits en modo IA-32e.
    - Se necesitan algunos bits extra de control para administrar el acceso a los diferentes segmentos. Además de indicar si es read only, etc., etc.
    
    Vemos una estructura nueva llamada **Descriptor de Segmento,** en el que van a estar la **Dirección Base,** el **Límite** y los **Atributos** del segmento seleccionado. Residen en la memoria RAM y se los agrupa en tablas.
    
    El selector de segmento te manda al descriptor de segmento.
    

## Unidad de Segmentación

### Selector/Registro de segmento

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%204.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%204.png)

- **Índice**: indica el n-ésimo elemento de la Tabla de descriptores. Como tiene 13 bits, cada tabla tiene 2^13 descriptores.
- **TI (Table Indicator)**: selecciona en qué tabla debemos buscar el segmento seleccionado. **GDT (Global Descriptor Table = 0) o LDT (Local Descriptor Table = 1)**. Hay segmentos compartidos o accesibles por todas las aplicaciones del S.O., por eso existe el GDT. Cada tarea tiene su propia tabla de descriptores (LDT).
- **RPL (Requested Privilege Level)**: Nivel de privilegio requerido por la aplicación. 00 mayor privilegio, 11 menor.
Vamos a verlo más adelante.

→ Memoria → Interrupciones → Tareas → Protección →

### Descriptores de Segmento de 32 bits

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%205.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%205.png)

Estructuras en memoria agrupadas en tablas. Una Global y muchas Locales, una para cada aplicación.

- **Dirección Base:** es la dirección a partir de la cual se despliega en forma continua el segmento. (32 bits). El segmento comienza en esta dirección.
- **Límite:** especifica el máximo offset que puede tener un byte direccionable dentro del segmento. Suele confundirse con el tamaño del segmento. Pero es el tamaño del segmento menos 1, porque el offset del primer byte del segmento es 0. (20 bits).

<aside>
‼️ Ojo que los datos no están contiguos. Por ejemplo: **Dirección Base** se divide en 3 y **Límite** en 2 partes.

</aside>

El procesador lee este descriptor y medio que lo ordena.

32 bits de address, 20 de límite y 12 de atributos.

**¿Por qué 20 bits de límite?**

Si hay que usar **menos de 1Mb, los 20 bits alcanzan** para especificar el offset en nivel de byte**.** 

Si hay que usar **más de 1Mb**, no alcanza. En ese caso, **el campo límite está expresado como el offset de bloques de memoria de 4Kb.** Sirve si necesitamos segmentos muy grandes. El offset no es tan específico como antes pero bue, cumple su función. 

**¿Cómo sabemos cuándo leerlo como byte y cuando como Kb?**

Vemos los bits de atributos.

**Bits de Atributos**

Atributos agregados para 32 bits.

- **G (granularity):** 
1 → el límite tiene granularidad de 4kb
0 → el límite no tiene granularidad. Expresa el máximo offset de 1byte.
- **D/B (default/big):** indica con qué modelo de segmentación trabajamos. (0 = 16 bits - Modo Legacy, 1 = 32 bits - Modo Big).
- **L (long):** se habilita para especificar que los segmentos son de 64 bits. Si L = 1 → B/D = 0. Sino el procesador genera un fallo.
- **AVL (available):** "usalo para lo que vos quieras".

Atributos originales.

- **P (present):** te dice si el segmento al que querés acceder está presente en memoria o no. Si no está presente, se genera un aviso al S.O. para que se ocupe de traerlo desde la memoria virtual a la memoria RAM. Ese aviso al S.O. es una interrupción de SW. Si la memoria está llena, hay un problema. Tenemos que llevar segmentos "que no estamos usando" de la física a la virtual (desalojarlos) para hacer lugar a los nuevos. (Criterio LRU).
- **A (accesed):**  cada vez que el procesador realiza un acceso al segmento, el bit se setea en 1. Esto nos va a ayudar a desalojar segmentos de memoria, usando el criterio LRU. Es un poco más complejo lo de LRU igual.
- **DPL (descriptor privilege level):** le indicás al descriptor el nivel de privilegio que tiene. 00 a 11. El kernel es un segmento que siempre corre en más privilegiado.
- **S (system):** define si es un descriptor de código o datos o un descriptor de sistemas. Los de sistemas son descriptores especiales para cosas raras del S.O. 
S = 0 → es de sistema. S = 1 → No. (Sí, posta. Está al revés).
- Cuando S = 0, los 4 bits siguientes tienen un código de 4 bits donde se definen los 16 tipos de descriptores de sistema soportados.
- Cuando S = 1. El siguiente bit (el 11) dice si el segmento es de código o de datos. Si es 0 es de datos, si es 1 es de código.
Cuando es de código, C (conforming) y R (readable), cuando R está en 0 no se puede leer el segmento.
Cuando es de datos, ED (expand down), indica que el segmento decrece hacia las direcciones numéricas menores, como la pila. Y W (writable) indica si el segmento puede escribirse o no.

Segmento de datos obviamente se puede leer y a veces escribir, segmento de código no se puede escribir y a veces leer.

Ya vimos la forma de los descriptores y también sabemos que se almacenan en tablas (Tables). Pero, ¿cómo hacemos para encontrarlos?

## GDT y LDT - Descriptor Tables

### Global Descriptor Table (TI = 0).

El selector de segmento tiene un índice, y un atributo que indica a qué tabla ir. Si el bit TI del selector de segmento está en 0, vamos a la GDT.

La GDT tiene un registro que indica la dirección de memoria comienza la GDT y el tamaño de la tabla menos 1. Ese registro se divide en **Dirección Base (**32 bits**)** y **Límite (**16 bits**).** Este registro se llama **GDTR.** 

En la tabla, los descriptores están acomodados uno después del otro. Según el índice que extraemos del selector, agarramos el descriptor correspondiente dentro de la tabla.

<aside>
⚠️ El primer descriptor de la GDT no se usa.

</aside>

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%206.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%206.png)

La tabla GDT y su registro deben estar armados antes de iniciar el modo protegido en el CR0, porque vemos que la vamos a necesitar.

### Local Descriptor Table (TI = 1).

Ahora la cosa es más rebuscada. Porque cada tarea tiene su propia LDT. Se usa un solo registro LDTR que va a ayudar a encontrar la LDT que estamos buscando. Hay un descriptor de la LDT dentro de la GDT. Es un segmento de sistema. Por eso el LDTR tiene el tipo de un selector de segmentl bit TI siempre en 0.

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%207.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%207.png)

La GDT y GDTR y LDT y LDTR y eso lo inicializamos nosotros.

Dirección lineal: tipo dirección física.

Entonces es un garrón hacer esto todas las veces que tenemos que cambiar de segmento. En la vida real, no cambiamos de segmento tan seguido.

Los registros caché ocultos son registros internos del procesardor. Cada registro de segmento tiene unos de esos registros. Además la LDTR y TR (task register).

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%208.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%208.png)

![Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%209.png](../teos/teo6/Teo%206%20-%20Orga%202%205a3c4fb4431245809831255b99a2556b/Untitled%209.png)