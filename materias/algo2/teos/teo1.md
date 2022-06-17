---
layout: page
title: Teórica 1
permalink: /materias/algo2/teo1/
---

Problema → Especificación → Diseño → Implementación

## TADs

Herramienta principal que vamos a usar para especificar problemas. La solución de los problemas siempre empieza con la especificación.

**Tipo**: conjunto de valores asociados a operaciones

**Tipo abstracto**: no conocemos la forma de los valores. nos interesa qué operaciones podemos realizar sobre ellos.

### Ej. TAD restorán

Tenemos que ver qué nos interesa saber de cada cosa del problema (hay platos, días y un restorán).

¿Qué nos interesa de cada cosa?

De los días, que pasan.
De los platos, diferenciarlos entre sí.

Se usan **renombres:**

ej: TAD Día ES Nat
TAD Plato ES String

El tipo de datos es Plato, pero lo representamos como string para estar más cerca de la definición del problema.

Hay **operaciones**: nos van a importar los nombres, los parámetros y qué tipo retornan (**signatura**). Las operaciones de los TADs son **funciones totales**. Deben estar definidas para todos los valores del dominio. Por eso, en algunos casos en los que la operación no aplica, hay que **restringir el dominio**.

Ya tenemos la signatura, ahora hay que darle **semántica**: comportamiento de las operaciones. Para eso vamos a usar **axiomas**.

### ¿Qué es un TAD?

> Es una herramienta lógico/matemática. Nos da la rigurosidad para entender qué hay que hacer.
> 

> Es una herramienta poderosa y flexible
> 

Vamos a trabajar con lógica trivaluada: true, false, undefined.
Con cuantificadores (universal y existencial) y variables ligadas. También vamos a usar rangos: $(1 ≤ x)$  → $x/x$ $= 1$.

### Vemos el TAD bool...

## Secciones de un tad

- Géneros: es el nombre que recibe el conjunto de instancias del tipo. Suele ser el mismo del TAD.
- Usa: se incluyen las operaciones y géneros de otros tipos abstractos.
- Exporta: qué operaciones y géneros se dejan a disposición de los usuarios del tipo.
- Generadores: operaciones que permiten construir valores del tipo. Todas las instancias posibles del tipo se deben poder construir con combinaciones de estos generadores.
- Observadores: permiten distinguir instancias del tipo.
- Otras operaciones
- Axiomas: describen el comportamiento de las funciones de forma semántica.

### Vemos el TAD nat y TAD bool extendido