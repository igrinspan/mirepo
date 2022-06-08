---
layout: page
title: Teórica 4
permalink: /apuntes/orga2/teo4/
---

# Teo 4 Orga 2 (03:00hs)

# Microarquitectura

### Tecnología

Mejoras en tecnología. El transistor es cada vez menor en tamaño. Velocidad de cómputo (3GHz). Consumo / energía. Potencia dinámica. 

### Arquitectura vs. Microarquitectura

La arquitectura es el conjunto de recursos accesibles al progranador. Registros, set de instrucciones, estructuras de memoria (descriptores de segmento y de página, etc).

La microarquitectura es la implementación de la arquitectura. Cambia de un modelo a otro. Es **cómo se implementa la arquitectura.**

Definir la arquitectura es la tarea más ardua de un diseñador. Hay que definir los aspectos más relevantes para que maximicen el rendimiento del computador sin dejar de satisfacer otras limitaciones como tener un costo económico o poco consumo de energía.

Hay que diseñar el set de instrrucciones, el manejo de memoria y modos de direccionamiento, el diseño lógico, el diseño del circuito integrado, su alimentación, etc.

### Instruction Set Architecture

La ISA es el set de instrucciones visibles por el programador.

Clases, modos de direccionamiento, tamaños y tipos, operaciones complejas o reducidas, etc.

---

La Microarquitectura es Organización + Hardware.

La **organización** se refiere a los detalles de **implementación** de la ISA. 

El **hardware** se refiere al **diseño lógico y tecnología de fabricación.**

---

## Sistema de Memoria

 Jerarquía de memorias y principios de vecindad: a medida que se alejan del procesador, las memorias pasan a ser más lentas (y más grandes).

El controlador caché trabaja mediante dos principios:

- **Principio de vecindad temporal**: una dirección de memoria que está siendo accedida, tiene una muy alta probabilidad de ser accedida de nuevo en el futuro inmediato.
- **Principio de vecindad espacial**: si se está accediendo a una dirección, es muy probable que esta y las direcciones vecinas sean accedidas en el futuro inmediato.

### Memorias no volátiles (ej: ROM)

**Contienen la información** almacenada cuando se les desconecta la alimentación. Parten de las viejas **ROM**, que anteriormente eran no modificables. Luego vinieron las PROM, EPROM y EEPROM. 

Las memorias actuales tienen tecnología flash, y pueden ser borrables y programables eléctricamente.

### Memorias volátiles (ej: RAM)

Cada vez que se interrumpe la alimentación eléctrica, **se pierde la información almacenada**. Son más grandes y más rápidas de modificar que las no volátiles.

Se clasifican según tecnología y diseño interno en **DRAM** y **SRAM** (dinámica y estática).

**Usos:**

La no volátil se usa más que nada para almacenar el programa de arranque. Se conectan en un espacio de dirreciones determinado por el microprocesador. 

El resto de memoria es RAM. Incluso a veces se copia el programa de arranque en la RAM cuando se enciende para que se ejecute más rápido.

**En general nuestros programas y el sistema operativo residen en las RAM.**

La memoria dinámica almacena la información en un capacitor y consume mínima energía porque el capacitor que lo controla está apagado siempre que no necesite cambiar el estado. Lo malo es que a cada rato hay que regenerar la carga cada vez, porque tiene lectura destructiva.

¿Qué tipo de memoria RAM usar en el sistema?

- **Consumo** → DRAM
- **Capacidad de almacenamiento** → DRAM
- **Costo por bit** → DRAM
- **Tiempo de acceso** → SRAM

Esto se resuelve con las memorias caché.

---

# Memorias caché

Es un banco de SRAM (ram estática) de muy alta velocidad, con copias de datos e instrucciones de la memoria principal. Tiene **poca capacidad** pero **mucha velocidad.**

Requiere hardware adicional **(controlador de caché)**, que se encarga de llevar los datos de la memoria principal a la caché y de saber qué datos están ahí y qué datos no. Este controlador se basa en los principios de vecindad. 

El **tamaño del banco de memoria caché** debe ser:

- lo suficientemente grande como para resolver la mayor cantidad posible de búsqueda de código y datos.
- lo suficientemente pequeña como para no afectar el costo ni el consumo del sistema.

Se espera un **hit rate** (#hits/#accesos totales) lo más alto posible. Hit es cuando el dato o código que buscamos se encuentra en la memoria cache. Miss, cuando no.

### **Operación de acceso a memoria para lectura**

![Untitled](../teos/teo4/Teo%204%20Orga%202%20(03%2000hs)%2022a12dfca0244e809e000f9560848ce0/Untitled.png)

## Organización del caché

Vamos a ver cómo ve el controlador caché la memoria.

**Líneas - Asociativo**

Mínima unidad de memoria para el controlador. Está compuesta por n bytes (n depende del procesador). Usa el **principio de vecindad espacial**: en vez de traernos una instrucción de memoria, **nos traemos una línea completa.** A cada línea se le asigna una **etiqueta**, que tiene que ver con la dirección de inicio de esa línea en la memoria principal. El controlador se fija en ese tag cuando está buscando en la caché. 

**Mapeo directo - Sets**

Se divide a la memoria principal en bloques/páginas/bancos del tamaño de la caché. Cada vez que se traen cosas de memoria se traen de a sets, donde cada set tiene 8 líneas. En el controlador se fija en el Nro de Set y de bloque de memoria y además, hay un bit que indica si los datos son válidos o hay basura. 

**Caché asociativo de 2 (ó +) vías**

La caché se divide en 2 vías (2 bancos) de memoria principal. Entonces ahora la memoria se divide en el doble de bancos, lo que le agrega un bit más al tag. Por cada Nro de set, entran 2 sets (de 2 bancos distintos). Si se quiere agregar un set con ese número de un 3er banco, va a ver que desalojar uno de los sets de la caché.

### Coherencia de la caché

**¿Qué pasa con las escrituras?**

- Una variable en la caché también está alojada en alguna dirección de la DRAM.
- Idealmente ambas copias deben tener el mismo valor.
- Cuando un procesador modifica la variable, se pierde la coherencia entre las copias de esa variable. El procesador **suele** escribir en la caché primero.
- Hay que definir **políticas de escritura** para cuando se modifica el valor de una variable.

**Políticas de escritura**

- **Write through**: el procesador escribe en la DRAM y el controlador refresca el caché con el dato actualizado. Garantiza coherencia pero el **más costoso** en cuanto a tiempo.
- **Write through buffered:** el procesador actualiza la caché y el controlador **luego** actualiza la memoria DRAM, mientras el procesador sigue ejecutando instrucciones y usando datos de la caché. El controlador, para esto, necesita un buffer de escrituras para ir guardando ahí las operaciones que tiene que hacer (HW adicional).
    
    Si se llena el buffer de escrituras (no suele pasar), el controlador medio que detiene al procesador y le dice "esperá hasta que yo escriba todo en RAM".
    
- **Write back - copy back:** sólo se actualiza la DRAM cuando se elimine esa línea de la memoria caché (cuando sea desalojada). Hay un bit que marca que esa lína está modificada (sucia/dirty). No mantiene coherencia pero es muy buena su performance. **Hay problemas cuando el sistema es con múltiples procesadores.**

Si el procesador realiza un miss mientras el controlador está escribiendo en memoria, el procesador va a tener que esperar a que el controlador de caché termina para poder acceder a la DRAM.

### **Coherencia en sistemas de multiprocesadores**

Cada procesador tiene su caché. Ambas caché tienen una copia de una variable. Se modifica la copia (se modifica sólo en una caché). La otra caché y la principal tienen basura, no hay coherencia. 

Independientemente de las políticas de escritura, hay que intentar que quede coherente. Lo más difícil es actualizar la otra caché. Suponiendo que estamos usando write through y estamos actualizando la memoria ¿cómo hacemos para que se entere la otra caché?

Se agrega un bus adicional a los controladores de caché (**snoop bus,** o **bus de espiar**). Lo que hay que espiar es el **bus de address**, para actualizar (o no) el controlador de caché. En caso de que esté la variable que actualizamos, el controlador la invalida. 

El **snoop bus** une a cada controlador de caché con el bus de address del sistema. El snoop bus **lee** lo que hay en el bus de direcciones. Es un bus de address. 

Además, el snoop bus tiene 2 líneas de control para espiar si la dirección va a ser leída o escrita. Si la van a leer, no pasa nada (con la coherencia). El tema es cuando van a escribir en la cache.

Es un bus de direcciones ENTRANTE al controlador. Espía direcciones y 2 líneas del bus de control (MemRead y MemWrite).

El método **copy back** es el óptimo, pero no es tan bueno para mantener los datos coherentes porque el otro procesador no se entera de que cambiamos una variable. Entonces la idea es usar este método siempre que sea posible pero reemplazarlo cuando la misma dirección esté en 2 cachés. Para esto se desarrollaron **protocolos de coherencia.** 

Cada controlador debe saber si la variable que tiene la tiene otro procesador o él solo. 

Dos controladores no pueden acceder al mismo tiempo al mismo bus de sistema (por cuestiones eléctricas y además se mezcla toda la información). Para acceder al bus, un controlador debe avisarle al otro que va a hacerlo, así no se cruzan

### **Protocolo MESI**

Es un protocolo para que los controladores sepan si tienen variables compartidas, si sólo tienen variables propias, etc. Para implementar este protocolo se agregan 2 líneas que unen cada controlador (Shared y RFO).

- **Modified**: cuando la línea presente solamente en ésta caché varió respecto de su valor en memoria. Requiere write back hacia la memoria antes que otro procesador quiera leer ese dato. (**Dirty**).
- **Exclusive**: línea presente **sólo** en esta caché, que coincide con el valor en memoria. Es como la modified antes de cambiarla. (**Clean**).
- **Shared**: línea presente en esta caché y **tal vez** en otras. Es un estado impreciso porque la otra cache la puede haber invalidado, y  cuando se invalida una línea no se avisa a nadie. Cuando está en shared no podemos hacer copy back así nomás.
- **Invalid**: línea de caché no válida. (En esa línea hay basura).

Sabemos que cuando está en modified o exclusive podemos hacer copy back sin problema, porque no está en otra caché.

Se agregan 2 buses que conectan controladores de caché entre sí. Cuando se usa MESIF los cachés funcionan en conjunto y en vez de ir a buscar una línea invalid a memoria, la busca en la otra caché. 

Todas las lecturas se resuelven desde el caché, excepto cuando la línea está **invalid.** 

El procesador escribio una variable, y la fue a escribir también a la RAM. Entonces por el bus de address fue la dirección, por el bus de control fue memwrite y por el bus de datos fue el valor de la variable. Al controlador sólo le importa el address.  

El snoop bus espía las direcciones y qué van a hacer con esas direcciones. (Address y Control). Eso lo saca del bus de sistema y lo lleva al controlador de cache. 

Entonces, cuando el snoop bus se da cuenta de que otro procesador está actualizando esa variable, le avisa al controlador y este se fija si tiene a esta variable en su cache. Si la tiene, la marca como inválida. Esto asegura la coherencia, pero hay mejores métodos para hacer esto más eficientemente.