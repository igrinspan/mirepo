---
layout: page
title: Teórica 1
permalink: /apuntes/orga2/teo1/
---

# Teo 1 Orga 2(04:10hs)

## Modos de direccionamiento

Modo implícito (instrucciones especiales, no se especifica registro ni nada)

Modo inmediato

Modo registro

### Modos de direccionamiento a memoria:

### Organización de memoria: Segmentos vs. paginación

## Segmentos

- Para ir a buscar datos hay un selector de segmentos (16bits) y un offset (64bits).
- 4 registros de segmento, CS (segmento de código), SS (segmento de pila), DS (segmento de datos), ES (segmento de datos extra)
    - CS: opcode fetch, referencia a instrucciones
    - SS: push, pop, o que use registros ESP, EBP
    - DS: referencia a algún dato
    - ES: manejo de strings

### ¿Cómo se hace el desplazamiento (offset)?

- Al menos 1 de los siguientes componentes:
    1. Base (reg)
    2. Índice (reg)
    3. Escala (1,2,4,8)
    4. Desplazamiento (0, 8bits, 16bits, 32bits)
- Dirección: Base + (Índice * Escala) + Desplazamiento
- RSP no se puede usar como índice. Escala sólo se usa cuando hay un índice.

Hay varios modos de usarlo:

1. Modo desplazamiento
2. Modo base directo
3. Base + desplazamiento
4. Índice*escala + desplazamiento
5. Base + índice + desplazamiento
6. Base + índice*escala + desplazamiento

## Tipos de datos:

- Byte (8bits)
- Word (16bits)
- Double word (32bits)
- Quadword (64bits)
- Double quadword (128bits)

Almacenamiento en memoria

### Endianness

Little endian vs. Big endian

Little endian:

- Se invierten los BYTES. O sea la word A108 se almacena como 08 A1. (Así lo usa el procesador).

Big endian:

- Lo lógico. Se usa así en unas redes (lo vemos después)

### Alineación en memoria

Ej: ALIGN 16. Lo que hace es dejar (desde donde estás) todas las posiciones de memoria libres hasta encontrar la próxima posición múltiplo de 16. Sirve para que no queden desfasadas las words, double words, etc.

## Pila

Área de memoria referenciada por un segmento cuyo selector esta en el registro SS (stock segment).

Puede tener hasta 4GB de memoria y se recorre mediante el SP (stack pointer)

Operaciones: PUSH y POP 

Es un segmento expand down. Se expande hacia abajo. Por eso cuando se hace un PUSH, el SP se decrementa y cuando se hace un POP, el SP se incrementa.

La pila se usa cuando:

- Llamamos una subrutina desde Assembler (instrucción CALL)
- Se envía una interrupción al procesador de HW o SW
- Cuando desde un lenguaje (como C) se invoca alguna función

### Alineación de STACK:

El SP debe apuntar a direcciones alineadas de acuerdo con el ancho de bits con el que estemos trabajando. ej: 32 bits, ESP debe estar alineado a double words.
El tamaño de cada elemento (que aparece en el push/pop) tiene que corresponderse con el tamaño del segmento. 
Entonces **no se decrementa/incrementa en base al tamaño del dato pusheado/popeado, sino al tamaño de las palabras con las que estamos trabajando.**