---
layout: page
title: Teórica 3
permalink: /materias/orga2/teo3/
---

# Teo 3 Orga 2 (03:10hs)

## SIMD - Procesamiento de señales digitales

**Digitalización de imágenes:** 

Muestreo → Cuantificación → Codificación

Con esos 3 pasos, una señal analógica se convierte en una secuencia de números binarios. La mayor cantidad de **errores** se da cuando la señal presenta cambios abruptos (mucha pendiente).

**Teorema de muestreo (Nyquist):**
¿Por qué no tomamos + o - muestras? El criterio de Nyquist muestra cuál es la frecuencia de muestreo necesaria para no perder nada.

### Procesador de señales digitales

- Es una CPU de propósito dedicado, para realizar cálculos y procesamiento de un único tipo de datos: secuencias de valores que vienen de la codificación de las muestras de una señal de entrada.
- Su arquitectura está pensada para optimizar el procesamiento de datos que no son de gran tamaño (a lo sumo 32 bits).
- La característica distintiva está en sus algoritmos de cálculo. Normalmente se procesa no sólo el valor actual, sino n valores, que por ejemplo en imágenes suelen ser los píxeles vecinos.
- Para optimizar se deben leer y procesar varios datos en paralelo. 
(**Data Level Paralelism).** Se usa otra arquitectura.

## Single Instruction Multiple Data (SIMD)

Es un modelo de ejecución que computa **una sola operación sobre un conjunto de múltiples datos.** Es útil para procesar audio, video e imágenes en donde se apliquen algoritmos repetitivos.

Un **procesador normal** debería hacer un **loop** para realizar esto.

**MMX - Multimedia extensions**

Los registros MMX son específicos para SIMD. Tienen 64 bits y los tipos de datos que soportan son: 8 bytes, 4 words, 2 doublewords o 1 quadword.

### **Registros XMMx**

Son como los MMX pero de 128 bits. Pueden trabajar con datos de punto flotante. 4 números de punto flotante de simple precisión (32 bits) o 2 doubles (64 bits). 
Hay 16 registros XMMx. XMM0 a XMM15

---

## Números Reales

Los números reales son infinitos. Un computador sólo va a poder representar un subconjunto de los números Reales, porque los registros tienen resolución finita.

---

**Quedan los últimos 2 videos (1:15hs)**

## Instrucciones modelo SIMD

Hay demasiadas instrucciones, vamos a ver algunas. El resto está en el manual de intel, vol. 2.

**Instrucciones de transferencia**

- MOVD, MOVDQ → mover 32/64 bits a la parte baja
- MOVDQA → mover un valor de 64 bits alineado
- MOVDQU → mover un valor de 64 bits alineado
- MOVAPD → double 128 bits alineado
- MOVAPS → double 128 bits desalineado

### **Aritmética en algoritmos de procesamiento de señales digitales**

**Aritmética de desborde**: llegamos a un overflow/carry, se setea el flag correspondiente y en el registro destino queda el valor truncado. Pero este no es el comportamiento esperado en algunos casos, por ejemplo cuando aumentamos el brillo de un píxel que está en 255, no nos gustaría que vuelva a 0. 

**Aritmética saturada:** cuando llegamos a fuera de rango, nos quedamos en ese borde al que habíamos llegado (puede ser el mínimo o el máximo).

![Teo3-1][1]

Suma, resta, multiplicación. Suma horizontal, suma de productos. Instrucciones lógicas.

Las instrucciones lógicas toman el contenido de los registros como bits, sin importar si son números o que. Las 4 instrucciones son PAND, POR, PXOR y PNAND.

**Instrucciones de desplazamiento**

- Packed Shift Left Logical
- Packed Shift Right Logical
- Packed Shift Right Aritmetical

Packed Shift Right Arithmetical: se conserva el signo cuando se reemplaza a la derecha. El bit más significativo sigue siendo el mismo.

**Instrucciones de comparación**

- PCMPEQ (packed compare equal)
- PCMPGT (packed compare greater)

Si la condición no se cumple, en el destino se pone 0. Si se cumple, se pone 1.

Conversiones de datos empaquetados y desempaquetados.

[1]: https://lh3.googleusercontent.com/w8G1uVDMQdvykIK2MRmeG7O37yrv98Z1EyDtAaZmg9F-S2DT9a8-c8jnylNzZUFMy4YDnA0txOJzKjHtVL2fMlWub0E9Gf5DYVRfLEfIHyy9pzmMmtNYz94_0moIRleRYE_2b52TbkDe8fG6yNjqK9dr4iWbUZr5QK3K1as3epujki3-cQ3Na-5z1dAiwiUSvAFeMIjPVdXrj8ejEPboc5vhuj-U8-BGwW_4BwliUKzhh61Mig7Pj6PmC_Gdp5xXwOR2qfBTAIwmdDL6uVq60KIirGDBRJl7w4Rg270HgAdifF5Ic29NSzruDFlcGK1OB6kBIrWrgk5NjAhlGOFEWIwCiJz8-kCMaVwq5vSirMeazJsRWYJbOihdrNcaWEM_aoaXXCM6RGsu7EUaspDYrYtHWOC2TjbOizmNXCl_fpuk9uQIMApzMxLyIVD9Jat7DTZm8dmRF9ws1Jh7QwfAGI-58xJy07jYCTnZpbB29ls_FHHnkkLhg4OwoylFtQJSmqCXrkZxk6aatO7DfaSNV7_i3gHN8HhTLSP-P0Np3HEIVQqOeBwEmd6J0Ii12r9dGcVsiiecg-dzvQ363LMvsGKPiwTDnniRVDNkNGGRYmAh-zUVH2YPCrVTEAW6aXVeNix29zT6GhTcb49yL4T9VTl8Giu6RXR8NsYWisBrJoS1lvgPdpEF4fUUrGzT-BpkgTL6xztJFzehEbkdjkx2aCxjSymoqwh-jt0JZTB3Xk6h__jESPoP3L7DyvQb2qZqdalihD0iONkgVyfwlYWGnz6mk7kZaHn2yLq_--kzD-8ZYWeqaQkBszxBeUNSZaVmieVcBNc9IiRa9M6BMROQ73N8QFM-_KaUz2I5JWjW-g=w1110-h290-no?authuser=3