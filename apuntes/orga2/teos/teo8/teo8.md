---
layout: page
title: Teórica 8
permalink: /apuntes/orga2/teo8/
---
# Teo 8 - Orga 2

(03:20hs)

# Paginación de la Memoria

### **Segmentación:**

- ✅ Provee un entorno flexible en la programación de aplicaciones.
- ❌ Para administrar memoria por parte de un sistema operativo, la variabilidad del tamaño de los segmentos nos complica mucho los algoritmos de un sistema de memoria virtual.

Cada vez que había un segmento que causaba una falla (o sea que aún no estaba cargado en la memoria virtual), el sistema operativo tenía que ir a buscarlo para traerlo a la memoria virtual. **Cuando no hay espacio suficiente** en memoria tiene que **desalojar segmentos** con la política correspondiente (LRU), que usa el bit Accessed. Además, ¿hay espacio suficiente una vez que desalojamos para el nuevo segmento?. Hay que chequear, porque tienen tamaño variable. Si no alcanza el espacio hay que seguir desalojando segmentos.

 —> Por todo esto, el sistema de segmentación puede ser muy lento en tiempo de procesamiento.

### **Paginación:**

- ❌ Más rígido para aplicar en la programación de aplicaciones.
- ✅ Que los bloques sean del mismo tamaño hace que el desarrollo del algoritmo de memoria virtual sea mucho más simple.

Si queremos acceder a una dirección, además de chequear el segmento, hay que chequear la página. Si la página no está, la buscamos y luego simplemente buscamos la LRU y las intercambiamos.

---

## Paginación

Los sistemas operativos, tradicionalmente, diseñaron su sistema de memoria virtual usando páginas (bloques de memoria del mismo tamaño).

### **Generación de la dirección física**

**Segmentación**

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled.png)

Un segmento es un espacio lineal de direcciones, entonces usa **dirección lineal**. De no mediar otra etapa, este número sale por el bus de address convertido en **dirección física**. O sea dirección lineal = dirección física.

En la Unidad de Paginación se debe habilitar seteando el bit **CR0.PG.** 
(Bit 32 del CR0). El procesador **DEBE** estar antes en Modo Protegido.

**Paginación**

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%201.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%201.png)

Ahora, con paginación, la dirección lineal pasa por la Unidad de Paginación y ahí se converte en la dirección física. La dirección física no tiene por qué ser igual a la dirección lineal. Si queremos que valgan lo mismo, podemos hacer un mapeo conocido como **Identity Mapping.** 

### La Unidad de Paginación:

- Permite raccionar el espacio lineal de un segmento en páginas de menor tamaño.
- El SO tiene en RAM unas pocas páginas de cada segmento.
- El resto están en Memoria Virtual para ser intercambiadas cuando se las necesite. No es necesario tener el segmento completo en la RAM.
- Así, el SO puede alojar en memoria mayor cantidad de procesos.
- La frecuencia con que cada proceso bien programado tiene que traer otra página de memoria es bastante moderada.

## Modos de paginación

- **Modo original**: páginas de 4Kbytes.
- **PAE**: permite mapear por encima de los 4Gb. Mejor método.

Después hay otros para 64 bits.

Modo de paginación que vamos a usar → IA-32: páginas de 4Kb

## **Paginación IA-32**

- Tamaño de página de 4Kb.
- El espacio lineal de 4Gb, que es lo máximo que podemos direccionar, se divide en páginas de 4Kb de manera completa (2^20 páginas).

Vamos a necesitar algunas **estructuras** para tratar las páginas.

1. Dirección base.
2. Límite (no es tan necesario, sabemos que son de 4Kb).
3. Atributos de acceso.

Pero hay algunas simplificaciones:

1. Cada página comienza en la dirección de memoria siguiente al último byte de la página anterior. Los 12 bits menos significativos siempre valen 0, porque están alineadas a 4Kb (2^12 bytes). Entonces sólo necesitamos saber los 20bits más significatimos.
A la dirección base (esos 20 bits) se la conoce como **Page Frame.**
2. No hace falta especificar el límite
3. Se necesitan bits para permisos y derechos de acceso.

Entonces, necesitamos 20 bits para la base y nos arreglamos con 12 bits para atributos. Nos queda un descriptor de página de 32 bits.

### Tabla de descriptores - Niveles

Como tenemos 4Gb direccionables, podemos tener hasta 2^20 = 1 millón de páginas. Pero ¿vamos a tener una tabla con 1 millón de descriptores que ocuparían 4Mb? 

No, vamos a usar **tablas de descriptores de paginación por niveles.**

Vamos a tener 1024 tablas de descriptores de páginas, cada una con 1024 descriptores de página. Además, tenemos una tabla que dirige todo desde arriba, que es la que tiene 1024 **descriptores de TABLAS de páginas.**

Cada tabla ocupa una página. (1024*32 bits = 4Kb)

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%202.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%202.png)

No necesitamos tener TODAS las tablas a la vez en la RAM, por cada tarea vamos a ir a buscarlas, por eso no terminamos ocupando 4Mb de una.

Nivel 1 → **Directorio de Tablas de Páginas (DPT)**

Nivel 2 → **Tablas de Páginas (PT)**

### Páginas y tareas

La paginación IA-32 se pensó para administrar la memoria por tarea, entonces cada tarea tiene su propia estructura de páginas. Eso le da más seguridad al Sistema Operativo en la administración de memoria. 

Este sistema permite **iniciar** una tarea sólo con un directorio de tablas (DPT) y una tabla de páginas (PT). O sea 2 páginas.

Las 1024 entradas de la PT describen páginas de memoria que describen datos de la tarea, cantidad más que suficiente para iniciar un proceso.

El SO habilita a cada tarea unas pocas páginas para código y datos. Si el proceso requiera más memoria, se van a ir agregando en la tabla de páginas del segundo nivel nuevos descriptores.
Esto permite asignar a la tarea hasta 4Mb de memoria. Si se solicita aún más memoria, se va a tener que generar un segundo descriptor en el DPT, que permitirá habilitar otra tabla de páginas para esa tarea, o sea otros 4Mb.

Para cada tarea, el procesador necesita conocer la **dirección física** en donde se inicia la estructura de Descriptores de Páginas. El registro **CR3** va a contener la dirección física donde comienza el DPT.

El procesador, una vez que recibe la dirección lineal, la divide en 3 campos de bits que serán utilizados como:

- Índice en el DTP, para ver qué PT usamos y qué atributos tiene.
- Índice en la PT, para ver qué página de esa PT queremos.
- Desplazamiento relativo al comienzo de la página que encontramos para buscar nuestra variable o código.

### Traducción de dirección lógica

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%203.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%203.png)

De la unidad de segmentación, nos llega una dirección lineal.

Los 10 bits más altos nos lleva a la página que contiene la tabla que contiene a los descriptores de página. Los 10 del medio nos dan el descriptor de la página a la que queremos ir y los 12 menos significativos representan el offset dentro de esa página.

Las entradas en el DPT se llaman **Page Directory Entry,** y las entradas en cada Tabla se llaman **Page Table Entry.** Notar que estas entradas son descriptores de 32 bits.

**Primeras conclusiones:**

1. No es necesario que las páginas de un segmento estén contiguas, porque la traducción se encarga de todo eso. Si una instrucción queda al final de una página, al realizarse el próximo fetch, el sistema de Paginación generará entradas PDE-PTE diferentes de la intrucción anterior.
2. Por cada fetch, escritura o lectura de memoria hay que acceder a memoria 2 veces para leer el PDE y PTE. Tenemos que leer cada vez 2 descriptores. En la unidad de paginación existe un cache de traducciones, el **Traslation Lookaside Buffer.**
    
    En la segmentación se resolvía con los registros caché ocultos, ahora con el TLB.
    

### Traslation Lookaside Buffer (TLB)

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%204.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%204.png)

La primera vez que vayamos a una página hay que hacer toda la bola y conseguirse la dirección. Una vez que hacemos eso, nos guardamos en el TLB el índice del DPT y el índice de la PT. 

La próxima vez que se quiera acceder a esa página, nos fijamos (en la parte azul de la izquierda) si ya fuimos a esa página y usamos su dirección base (parte del medio) + el offset original para ir a buscar nuesto dato.

O sea (creo que) la parte de la izquierda nos sirve para comparar y la parte del medio para ir a esa dirección.

Los bits de control sirven para varias cosas, como por ejemplo para políticas de desalojo.

Actualmente se tiene un TLB para código y otro para datos. Se ubican en la caché L1 de código y datos del procesador.

Como la paginación se organiza en base a tareas, cada tarea tendrá su propia estructura. Entonces, el **CR3** va a ir cambiando de una tarea a la otra. Cuando cambiamos el **CR3** se resetea el TLB (excepto entradas globales).

---

---

**Pentium Pro** introduce 2 métodos para extender la capacidad de direccionamiento físico, para que se pudiera acceder por encima de los 4Gb:

- Physical Address Extensions (PAE)
- Page Size Extensions (PSE-36).

Ambos son excluyentes y se activan en **CR4.**

PAE es el que se impuso (💪).

---

---

### Cómo son los descriptores de página

**CR3: Descriptor de la Página que contiene al DPT.**

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%205.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%205.png)

Bit 4: **Page caché disable**. Para deshabilitar el caching de la página.

Bit 3: **Page write through.** Te administra cómo manejar la coherencia, cuando está habilitado el caching.

**Descriptor de una Tabla de Páginas.**

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%206.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%206.png)

- Bit 7 - PS (Page Size): si usamos páginas de 4Kb o de 4Mb. En nuestro caso, como PSE-36 va a estar desactivada, debería ser siempre 0.
- Bit 6 - IGN (ignore): siempre en 0.
- Bits 4 y 3 - (PCD y PWT): lo mismo que en el CR3.
- Bit 2 - U/S (User/Supervisor): si está en 0, supervisor. Si está en 1, user. En los segmentos teníamos 4 niveles de privilegio, ahora hay sólo 2.
- Bit 1 - R/W: si está en 0, es Read-Only. Sino es Read and Write.
- Bit 0 - P (present): igual que en segmentación.
- Bit 5 - A (accessed): igual que en segmentación.

**Descriptor de una Página.**

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%207.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%207.png)

Dirty: una página está sucia / fue modificada. Cuando hay que desalojar la página le vemos el D y si está en 0 no tiene sentido guardarla en la memoria virtual, simplemente la pisamos. Pero si está en 1, significa que alguien la modificó, entonces necesitamos guardarla antes de pisarla.

Global: la página es global. Cuando cambiábamos el CR3 (cambiábamos de tarea) se limpiaba el TLB. Pero si queremos usar páginas en varias tareas, para cosas que se utilizan muy seguido, podemos marcar esa página como global. En ese caso, el TLB no se va a limpiar por completo cuando se cambie CR3, las entradas global van a quedar ahí hasta que (muy pocas chances) sea la LRU.

O sea, las páginas globales no se ven afectadas por cambios en CR3 pero sí les caben las políticas de desalojo de las páginas.

## Paginación PAE

**Características:**

- Permite trabajar con páginas de 4Kb o de 2Mb y así direccionar el espacio físico completo (+4Gb), pudiendo combinar ambos tamaños.
- Actualmente PAE permite trabajar con 52 bits de memoria física (hasta 4Pbyte de RAM).

PAE se activa en **CR4**

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%208.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%208.png)

El bit 5 - PAE hay que ponerlo en 1 y el bit 4 - PSE en 0.

Una vez activado, PAE permite traducir una dirección lineal de 32 bits a una dirección física de hasta 52 bits.

![Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%209.png](../teos/teo8/Teo%208%20-%20Orga%202%20ca981e6b5d0e460a82f72c54a732a757/Untitled%209.png)

En PAE se arma una estructura jerárquica de 3 niveles. Hay más de un directorio de tablas de página.

---

**Quedan 3 videos (01:15hs)**