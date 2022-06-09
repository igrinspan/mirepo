---
layout: page
title: Teórica 6
permalink: /materias/orga2/teo6/
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

![teo6-1][1]

El procesador busca instrucciones con el registro CS (code segment) y el EIP(32)/IP(16). 

Vamos a parar al **Reset Vector.** 

- Los valores iniciales de CS:IP son 0xF000:0xFFF0.
- A la vez, el procesador está en modo real. Entonces, su rango de direccionamiento físico es desde 0x00000 hasta 0xFFFFF, 1Mbyte.
- La dirección física que genera un procesador 8086 viene dada por cs << 4 + ip, que es 0xFFFF0.

**Un problema**: los nuevos procesadores tienen un espacio de direccionamiento mucho más amplio, y por algún motivo arrancan en Modo Real accediendo al espacio de 1Mbyte del que hablábamos antes. Hay que intentar que no se acceda a ese Mbyte de alguna manera, sin interrumpir la continuidad de la RAM.

**Una solución:** al llegar al Reset Vector, que el procesador use algunos recursos de Modo Protegido, aunque esté en Modo Real.

Intel agrega "registros caché ocultos", que están asociados al registro de segmento, y para el arranque se pueden usar en Modo Real.

Los registros de la derecha son exclusivos del procesador. No se pueden tocar. Los de la izquierda son los que vemos.

![teo6-1][2]

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

![teo6-1][3]

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

![teo6-4][4]

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

![teo6-5][5]

- **Índice**: indica el n-ésimo elemento de la Tabla de descriptores. Como tiene 13 bits, cada tabla tiene 2^13 descriptores.
- **TI (Table Indicator)**: selecciona en qué tabla debemos buscar el segmento seleccionado. **GDT (Global Descriptor Table = 0) o LDT (Local Descriptor Table = 1)**. Hay segmentos compartidos o accesibles por todas las aplicaciones del S.O., por eso existe el GDT. Cada tarea tiene su propia tabla de descriptores (LDT).
- **RPL (Requested Privilege Level)**: Nivel de privilegio requerido por la aplicación. 00 mayor privilegio, 11 menor.
Vamos a verlo más adelante.

→ Memoria → Interrupciones → Tareas → Protección →

### Descriptores de Segmento de 32 bits

![teo6-6][6]

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

![teo6-7][7]

La tabla GDT y su registro deben estar armados antes de iniciar el modo protegido en el CR0, porque vemos que la vamos a necesitar.

### Local Descriptor Table (TI = 1).

Ahora la cosa es más rebuscada. Porque cada tarea tiene su propia LDT. Se usa un solo registro LDTR que va a ayudar a encontrar la LDT que estamos buscando. Hay un descriptor de la LDT dentro de la GDT. Es un segmento de sistema. Por eso el LDTR tiene el tipo de un selector de segmentl bit TI siempre en 0.

![teo6-8][8]

La GDT y GDTR y LDT y LDTR y eso lo inicializamos nosotros.

Dirección lineal: tipo dirección física.

Entonces es un garrón hacer esto todas las veces que tenemos que cambiar de segmento. En la vida real, no cambiamos de segmento tan seguido.

Los registros caché ocultos son registros internos del procesardor. Cada registro de segmento tiene unos de esos registros. Además la LDTR y TR (task register).

![teo6-9][9]

![teo6-10][10]

[1]: https://lh3.googleusercontent.com/jlLaJ7aWV_TOSoztlEvLAIT7TQk3bfl-Sx9Gmj8X6APsJX1ROEP_UiB-94aw3j1USk9HLjB1Gx1PC0AWzldm7cu5DGJOTmyJ1LQwhLh4UmonkrroJjLaP2yATdUOVTqR1uxvKxYgCc_cVcLFoxwJOG-b4pdtLtvNqS0AMsDQzDLnnNCRlkCCwJkuoBd3Htby7aqrLfDzLd_U_3Xcf8WFFep-XsukZyTDtSeyRQyXngyhn3B5as9UKAJdMHNBrx1PoJFI0G1VH7KxkoxZtVJWLY24IQ74Op6VRPAeRp9Y0ByHdwPhIPOPMsy9-njTe3LjyIriffGiS-Hj4ciaFJ8ZCMpWtfeCX1MQR1csp-sflD-sP2Kbnad6nC_-nS8LfwTekj8zP2XnVwhhvBi1m4sxl0Zt5OsTk_YKjKHtfyAqPJ0wE60NqXGMVudyrwRGC2ygdu7R2xpUIXOJBS6wB0GXhTMM-FqyuK2Qs9w0Tcl5RLYNtLPcw0mrYOrCny4jVaHoFQPiJBgt6nM71xMjeWfHToPM4aJbHGyYv2woLcb0rChXgzkE8CHCWCXYrUcht7h9urTM3bkl59VbONKTwMgabhlnZiSq1CpVNM82iOAjMt3_KdqStSomy-pVRxzw5f92PWtWWCLQX9JDD0Rd6V7qSuDfrD7Y2gt4YQVvGD5phjVHsX--dKZod8YolwsmRFIHfKPOqNKD8nJ2YmNQV097ESEBVS9gAy82ugmZYVcYMxeVGW5ln6i9pHrXilGaowDORG_56OpXJjygiLErVfGUbIMI7AWksvvyF5RVEljk27x_2fbTrCYu_pRpjaoLz-QfzhG9I76efSN9qOpJRuLeakwKL-DYP0SrNhEYblR6EQ=w844-h646-no?authuser=3

[2]: https://lh3.googleusercontent.com/SSbXMswpRaWYNa3_5uy9BDwNaKaxdN_y_3nU8E5K_G4B8WFC_CLETmy9IxhnUIca4-S_5srOkVsvHELB9U50bsYzd7vGlbwfWZWhhYXp9lL4IrhZPB9wOFr7HtFc0shFN7ZIaacq2MGELybExzMS-yRGvMafwFXu-eZOPokMDkxJcXcG1xhc-kxrxHqK0DdHHSLEYPlV99QDE9DedSlYAVRBLtFp23mRdbvrBwLwLiLLcpFGwxXM26ACHI6EkMWaKTxIBTH8ugARaP7JlMaQethXygLtYqOer3WG4eZg8nJXFPB1rM5MdbD1hrKCA6m3qFx37_f2XzzO0nXhb3i5VV77aEleDZMorOotUMa2jNe3SkHV1pBnz371FFsLBGnguseQYv5GFMhajnAhjXMjA_tAwodtfuXIWJGDYupZ6l1mRhEftqV-5GnoJ6-0GYMZlgi2VmPN7xIXOJoGYiPdRToPzo-sdDFFWRy5hq-bmVHCHELHrdkxFnuwHhboccOh1ITBAA-7PtIpHiIZC2IvYOfRUxQYa0dfrJwlbQBVkfwTn5keG99_kfDkwtF-vd8VxILY-IO5JuojnEfV99pintIug5Qw86-nw4HZIZqM1vgv6cuXxHp-tSRM12DaLoTXQO6yNBt-EZOtjKfbAY0UtJ4LhiwjwP6jrD9HoKnllBkXj6Ez2E7Ru5y0lh9t-r-BIzieleyqCCUMG5moWnCi8RPLVocXogF3Z2CA4kBdR2nxs-G5senHsEXkrLR61io7J0f0xpBLPXM9Ua7D1jGxPcPNN7IMfbbgH3mUWJreQ18eEUJ4Q8O1_YrzapiBp6bbLLlLPlihAzbNohY9kQO3IPfePCOZv38M9IjXOpEezQ=w1140-h712-no?authuser=3

[3]: https://lh3.googleusercontent.com/Ve93wXUQ94sBO_x2GnHjSVkcRcckz84oySNMtz_1_Fm62x5y77VVzBh4hIOj_lhttKvuvmxaCfTeIF5oyHqlnteJbShs8BOGJxV3T54NPx3OGr0oKiu7TdwSRZowZzWaCkKkT5-rsphrOg3JQ9zXWKJ43FIZKt_LNOPoHJXZyNnnIHal3UUxbgsN2SYjlxR0GnP1pRoPbp1H5_l1LUZpwYWCNDQckJOMd7Do-p1enAQsnpmXyGVhy2nAOmnRu3_EsFJN_GxW1kD-G0VVRGPvjdqJE7ODBVszVAqvOpva7wVg62c7TZeMnmk1yrodWj6bsT6RInKxCoN81RrvYztYqnGTvKiAW20jgBqE83xNkFnWz2-M34CKfrjCIzjwWyRxJZ1NjBrNRENU1xQkMGbHyfvdJxPnGN04z-9z3SrUsNMNqNnBX0_6v3ZvmXX06Il97lnGw3fgvi1WLA-jfVQ5A9nkso122hozg3gmcDbIT6vMmlLT3azGFwQxJ1XPqc1LzMh0_a_YYAUx91oimop6r41sgx4HXAEz0MzFJo2mftKk2dUT86ulkkAhUHTeBhN9VeyZKO4Qerxk_y73i54ES3crDVZRRjf7ywJYSiUbESWi2MjqtqHvLGBDSFJkHv9rFdFbGfV7fkG-uBlSF01upKCrtdQd7x8Q5eAJC8H9aoDtqndPfvcQC1x_aLIIcaDfK1r8mD9IEkpJb3wxhcfytox42uVk7NnBZ6c6NGwVc9KIa2ZTnNq-2cGgHntTGhJHtZjSKRFU17hakdKoFsyldvzrvMUeIesHBRgIo6aQYR-RbkoNxe3T7qY6GhdvbdOCVHYuApDXp0onNXnee32jEBLIRtP1d7arbKtG204eZA=w1118-h314-no?authuser=3

[4]: https://lh3.googleusercontent.com/FCKXqDXoH9k2kn_cZobLYbK-ayFqtutq1QFveDtsarb6AIVRQ1YoNdw624Nwr7-KArxYU47ItChRIysdlvud5DHPE8Uwo5tNmYLzenCvFpa-UFBAHuylUrjFPAaypaYSCFySVpDQHaxAS4IdlkzzHHO0iJCSYLuKUNTUGXVy4TrXKe87mugY9UqKSYyMlQslmM_iItTOfgrPXYhsBJG62b3mocsMKs0PhITafWcWld-nFWmWWJFNkO6cAQnD70HrwtugngQ9coVNb7YxfUwK8DiXTnyEC16fqYG2vnMMwTbaG01zTG7XpBgD94Gk7g6-yYjdbyMtJV-FFuSH0kapZ0oIsnWgSHcrBcv2DWb_LdY2Vw8zzkJlkZ7VS8NZmIyKVKgHS6aEodtMnYtMzdZebMuPvhf4Y1vDnqWOdNewabDQGOYoz5CQJKGAUeBeNXkw3sX1Ka_8rY83UzNKAs4Bt5NdrvsVG5ojKIw0c5WiYUlTjqOKMtnGvkrqi7hF6gDcgmJbqGmFpRfaY9p-HOfXC3-EV6aLgozVoua_ObPX1rU0dynN79ohB4WH8nP5URuoHRUD5aEkc659rlOvb27WEmWodpsUAQqHo2g3D5aIPy_AG82_KaBAXzFrGHLEQojZrpcfIbvNySdhfmFuTIUC-NuithaupYaQBjiXe-RSnyr9bPT2h7KnU9pHpe6tfpzd9OWP3jRxPrHG4Cktt-TeGnKN-rMjFiIE5qrlw6mHJLW8kIyoocy2yL4blFzMv4rEnOmyJDEjnwNklEit1_X6hrTsw4XbJn-_Qu_v3MjD7l2iW6JVCvHXvBin81UWcA8lBCSAaApSHe4FqXQudDPGobjAunvbTVGhC_FvDq-bsQ=w864-h704-no?authuser=3

[5]: https://lh3.googleusercontent.com/BBSHECBG9gCVaK8zpfXe9UkQQXJSOYZpuKs12Fo_EZvpXoOdEaDn1LFeddI2W3T3o54X8u0nK3P-8bAk580F5Xzo7lLuA0dSx2wi5RW7-mE_DrrEG1qLqpu2c98IzgSb478rCqD_TVy9JT2wqe1MEvEsjZxzvdNgCNEH5ghY2C1D1AkXTOLgy7oDYIxoVKx1m7wCMDWA6kY26rY9diPeGJ7B-U-gj98M1woC18juB3TS3ZZD6_vwdNqEQ_xbbTBy9m02NJYp4AxW11wn5va0SGRCmIbXi0XJcp1IdR-S962boKwBXrLqvzO81g881bf67AL8SP0Cq5MOKTSGPuM291FS08R1VZFtqEaCmJUgVxIr4UWJtvvoMuaiP9AK5rdsG5Z-B5kSu6E8Mazg408BuAC5pbnDBvfevu_MiYRDGgmjYeBQKdWGSxYar06lXiyhZFHVXmr5irELC08YqKSwpKS1Pi-g-eq8afAExwsWH8JnFVjiUvTSOR3EEo1h4IUPhCdH-rtGPkFPcdD9pAiSPUuuMEsrptkII5mZ7PLJppH4cKNglM005K791ybGyeinZjwzVRTPPBY7g5V5Ns_8Iw17GJ1gM25DsWnmLr80j5SSYybvX6DKj0kaCwVQEckX9E7LEJ-OcT76IrsywZznBFTbc9vFofSze_2UwtLM2ann9Jx4UOvwrz_1THi8W4WSdhCUEU0u67zFw6YHrYi0X4ivhvklGIiQbi2GaXD6P2BgjV7CEbzdJv3PMqD87PJNcyGcjyWxbjajiCdNk-DjtfjHTeFXvhiJkhRCFixDfl7cJGZFp7mP1tXq6L7cs32EVQRWYNOuDearpVTLgUvS4PGQAKf4ZQerAxSPNuxhmw=w574-h102-no?authuser=3

[6]: https://lh3.googleusercontent.com/AZa3bOxzahUpHXysghvW9cZP6B6DqJjE_ou0u2xK3uWxvV9ZvuIxhkIgHa42vAGRhVFIUteGOOBCPSNeGbKjsoykgKuZSj5rQ1Ta8BhjP860bqakHZbFhhMpgi6m6vIw3haOwYIgnOBkDIAMobnzXKlP0IQkjL3Mpftg2PHJR3PfvmE5dGBT-moo_ftnlStqUXHggE8C0hDWyZYcpHahdz9QtsmAmYxUTO0pSgMbPySN3qUGX1YLCUZ3OVAtLq_L0YY4LB6xUCdFzFHruEAi3k3_h1eXGuSecclhZbM-nEy5gJZnfejW9OI3uXgZtITjcy0waBUOxG8vm7YKdEvnFKpobU3MjAw3_OWr5CaFp8aEq8P4nuZWHtbDmYPK0Dzu9NXgPiTSHhXRQVVFe27UYmg_vKtxKSPyjPAFZIHVI6gwALMzuIeeLZYb7qWBITjquLv03Jd9CEqeC_fUR5jO9Ds31emtF3sBKOk9BoYExb6s4GoguinOPy8EePNh6tC-mft_hKhylS4poYeR8a0QKSocU1ASqNKnJkIeyAHAv7m1xXMQXnDPhvd7rtUk5RICCgkXUcHNshbcWLkYBraOWbiU7Dmj11LOLXy6CR1oOIkIEYh_lquCimHq94AcBwZ_H3I-71vLKykuU5cy03-mOg4UnlAFWI0s3KmUDBcJJ0y-KpYBd8L6CncfBdQGJfiWZqdOjFcqWK5Q1nz32NmwKJBKU-sUFEFS7ZPmtxAr2KL7dDrGYEyOU8HKAf3_OGm4jKDW34IcxWekhQwwzff8fUcWPDZdjRZp0DolQLE3wrIsljI0OqgPJR-6vCPoG9B2bGXfcPR5gncwJFEQesTSE3JMywXzLSW-rlxqVbMnzw=w1140-h726-no?authuser=3

[7]: https://lh3.googleusercontent.com/LqDCC6tnfv-duvzLMsT3b04eYoGLRcEZmP-LR6fCWejxmynrKrA8GnzcI_fQSbpynWbtOGC_IvpUol9q214evD3uWUsKWXMZtaDgu0sJ2YV-ROMbt3AtzExjwI4_io8UKH1YrCAHeKOGfz-RlWHheQXZRaXxSK3vbPB5cRzVEnIrgGPt_7M_IfUtPhoZvLln8cUd-rVmn86LD-7Jo4H8v2fROX3GhOO-dk9O8pL75Slf3nsNp4v2GVXf_rgZgB9sGS6Yd7k-3psqS4tCepW_1mMIKlMalWBU9uXa9xYg3B4qRQZLto41s5-8budSWrSG75UXccKcjE8w504jxQvR4tWRiQ5VaVAsGvLJBM10XT5B3cca2ifeYTLQaM6UywMW6WEWIXnOcAFOifQrcYDO6Rw3pxEiSCGMTpo736C6EwFKFxz_f0n9bAqSgxb-ujzJVSTC9iFPISSpcdRt5GPelt1u70pCkzuXU0Wwr1gBXSnVg7O5S3FV8eQ0h33A5gO_D63PqSK8dB0Q2JOaN0dovdx6thts8Pu-bGezteKPEleP9N6ANTmcvXyPB4q9oeIpO4Zucwqs9Msu7-XKa6Xn-99Y3vDzgKqzfWSnL2j56GRMn_tcjW7-CW3i_SHc1j1HkRjwHrJfUYarvygYBki6hJSLElJ2eOZB7s9RUp_EZ9LoDP2wV3XMpDq7el1XwRbrqdZJaRsNAR5zFJCCu8YwAIdCRrYkvD7-SDjPS0gk2CMa-dlBwDpNeEOzB0U5T4TGN7cx0EWMqr1PBGPfgggLu8xSO4MCYw4JOk5wLB-P4M-vadDXXL0E5dDMm3I81DSA7Bn0I1fH33H7WSJa6NWHFfGJBbBp1G0PjiP5ewkL0g=w1114-h676-no?authuser=3

[8]: https://lh3.googleusercontent.com/_DLUS8IfKJjlAi9A2zQLwTArofSTXbNv2Yfj-4ZUILve9SWaOgGbVAJPga2jo6RIzC-FMIF1Sm3VsobknzT1g7LTcgG8b_wSMfoWb1BPMSr-yI-L6sCzMiGr1cNGsQKAS75JUa9Wdeso4HHs_UNhOnNbtB6CKYJ8yzt1msNrvVEUFP_eXfnumuwBRMdqj6Q3g94F4h_-srw4j-_rvgV9S5dwnP0dGvBcoV4bp4U1zhSL1KsV5cVvYc39BZmCDkK3i-DIO6x2vGMGNW98kpnKK015vy0NWvs5XPW0w5MhnhVElW3TdSNDCs5dEV965neWQXhEJsHGk6xvIuGbxGKgM5-u9amj8CRLZNMrSu5gBFi-jDqjvRQ1TtdFP7fwodGtBlH2YlMKLVTsuAJiPxyhxfariUkXwaa7gpiAhKROVM2p_esiH8av5cBV6OSzJRvS8XF9lItKfKkTgdGU7YYHh1HuKbo-LKD35P8Uc6tkyFiyrCUrgLlZQXdBkBug7y7dWpCCJpotV_tsOdnN-OGC3dLrdD92gPE1P4mUyRmOWIwhStyIKrR-7S8hUe-wRes80akKdQ-lRM3R2SiPoZALttWZv2rqKwEJ1_q78p5HqzFei_KnA3POiE59jLDt8vso3KZ1vzNQZZyIKh9zw8wXCtXNz_b12Fknrx53wqvv4Dwg2eDZoKyuzdlJCnA3zHn_wZ04GOq0y-cmJx-5qDsVZegY2p9FQwOVkc9VKu0v98bFaR-KhpVW3nKyzarSAWsbFfMa1UCKRhm8z9Q4DKVlAieEoEeUcDyDXYDE8fa52T2bhJ6qM8-WrvJOp0hgpdOb9QWbPmSVHesc09EEMeuw0AkKj4PYxvQCiC7LSccq3g=w1110-h212-no?authuser=3

[9]: https://lh3.googleusercontent.com/OFQey6-sB1eIGORikHiKZwLQJEIYl-_MK9vIElwjBlCibueCLwHxdLXzqO3FNEHFkFHtTDwHZWyYo3DVtEpY4HsywyslirneFeejcs3idsTwOIwzTI7QQ-yXW_leTGboLZnGr_AAuxdkf3hPJwMbowbxJgbzr-hYDYp4Q2E3tEAioj6y9JS0uEgH0oQT9pzXuuK6V0CrNPTVtPuyN9xw7_ab0sTA9vuMIS3BJk3qoxcj-0QQdlCzEmPzerKBa5VRoVaOV02iF0-goq-WqBp5fDh6AMqx1sIP0YvAHE57DwRAaST2i-yB23uGN4Mpq06HfCD27rsg5Qdu84SWavSb_E8L_K8ytShkvlDYsy1kq4m4CqDguFzV6mUCG8LuhH4dx9WvWfO04jYU6DDUptUp4l8r45NF6_VE_g1wSL-S7LpzRUNzIgJrViatz_2nBKBo0YAg39qYldzEDWnp2FptUVVjNf98Ln1x7JRYE8DJIxmZ4fEeO2IWg_J7-BNTiD-Dk7JsHAoKvSi2o4r1q_j2uXTFuCBtPcoiiGrzPwohsgdGLXpc8MpS-bq3zk2KmslhO2X68kbIeFIWQWVJzRbibW0WHFwwntEvCo5LT-uDuVrBC-8sPsT0yMeayHZzr1r0FaJfeVX4jnNpXIJ6vMPpfVOrR9s4cM5ZK77IJuQIzEAHG40_WInrAE5eF3iFyqqE2x9GsfIrpfcKUS_Ifr3HB8IZ4LJFJxiWc6qbuheRT3-odgCQqLIqVYQxy4yYMfqmt5ClY09dXMCeJzyAhrKa-AN0nkCBNJLH6sl4H29_95hTdqBM9rpLtRAQBiDbWpqv5aJ4LFV-644FTX_kWQVhN5DXAVpIQ3d7lvomkjJuZg=w1022-h562-no?authuser=3

[10]: https://lh3.googleusercontent.com/X1tzkuFIeTaH82zoK7mtaIarFg2Ju7EUNl-ezklRFZ-6r95ie5UVtLJrCGygbXDaxpWbJkqNVxSdg374QWA0Ie5jddUNMAHzDJNcpeWkW9peVmJJ7ukZn6VWEJxSmlZEuYfvtzDcEeXcOTho6BDsqCAiyl6_HUHao_ySIAWnJ-LPKuUwyfIPI7UtUrBvtpYhTZxu_asWN6qDJKya5pmBvgFK7R2Hjpp01olW25UyWLe-_uEr9hSy3gyfbw808ei2GV1MCmkV3OW0-VwhmU-HfbhJN3hOyLE_oi13KXvVamyTfsBL45LsbZc_BpH5n6rNi2V4Z42ue7kOlP6ei8_Dau_ysfa8Zv-ZROLa3iqfANAPr7eLu1iGsfVRENR9c28pdpRIdYXXVxVRHCdpF8ctQGX2rTzcT2NU7SGMR1SJDOa0ssbP_s-8yfTZMbNS0XgUrd5HvYjMstvQEnZvUkLflC6BbX57B0OaqX2PRL7Rra3DfWmuT-r4Duo57vIkyQ4gPhauoK5W5D9kPKW7z1YrEDqE-Oi5P3-F9q1bzdTRTdtct6GSxnvrF7Xqn0UAK_MZ_qq1JfZ52bbmHFaziYXhAfJ0dihY7wiPhU3JcLJLvAg6z6HaI0RoyzgXm9pJQGHQaU3vzSykGkBsp9Wq2QLYXTJ4i8cyqB114MhupUXqPZWXy59e6AbiKZRGn3KHX-zy8UtkZZrbbr2zb6LyfkPb2Gy9rh--YNgnZfXoHshbTeMpzI7rpgX8is5BsW6ZthXUUQBJClq0h1qdYwlhjsR9MxZv39F33SYFI9vJhoeH4mE_0UEmAyn_sn-rIMc7b-YItoWp5wRej_jWqVa3lmfEgmkdS59Ylj8fXN9x3oO98A=w1124-h664-no?authuser=3