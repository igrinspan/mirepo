---
layout: page
title: Teórica 2
permalink: /materias/algo2/teo2/
---
Ya vimos 

- Necesidad de especificar antes de solucionar el problema
- Conveniencia de hacerlo de manera formal

- Introducción a TADs
- Cuantificadores
- Algunos tipos básicos: Bool, Nat

Veremos:

- Más tipos básicos
- En detalle las partes de un TAD
- Especificación y modelado, conceptos importantes.

TADs no es la única manera de especificar, también se pueden usar **métodos informales**. Estos son más fáciles de construir pero no es tan fácil encontrar errores/inconsistencias en la especificación. 
Entre los métodos **formales** reconocemos los **algebraicos** (ej: TADs), **operacionales** (se hacen en lenguajes de alto nivel, tipo programación) y los **basados en estados** (otras cosas).

### Seguimos con TAD Restorán..

¿Qué pasa con algunas cosas de la descripción informal del restoran, como por ejemplo el dueño?

Utilizamos un proceso llamado **abstracción**. Consiste en identificar los aspectos importantes e ignorar el resto. 

También utilizamos la **modularidad**. No es necesario ni conveniente escribir una gran especificación que mezcle todo.

¿Es eficiente lo que especificamos?
No corresponde la eficiencia a la etapa de especificación. Es una pregunta para la etapa de **diseño**. Es la etapa intermedia entre la especificación y la implementación.

### Vemos el TAD Conjunto ($a$)...

### Axiomatización (cosas que sí y cosas que no)

Las operaciones son funciones, hay que evitar inconsistencias. Evitaremos la **sobeespecificación**.

No hay pattern matching.

```cpp
**es_positivo(0) = true
es_positivo(n) = if n > 0 then
                    true
                  else
                    false**
```

Nada nos dice que primero va a evaluar el primer axioma y luego el segundo, entonces si entra es_positivo(0) al segundo axioma va a dar false, pero debería ser true.

La forma correcta de hacerlo es con un *if then else.*

```cpp
 **es_positivo(n) = if n = 0 then
										true
									else if n > 0 then
													true
												else 
													false
												fi
										fi**
```

No debemos especificar sobre casos restringidos, porque están fuera del dominio.

No hay que **subespecificar**, excepto que sea el objetivo.

Teo 2.4. ejemplo interesante pattern matching y violación de encapsulamiento.

**Libertades**:

- El usa no es tan necesario, el exporta ya tiene valores por default (los observadores básicos y generadores).
- Todas las variables libres se suponen cuantificadas universalmente. Sólo cuantificamos cuando se pueden generar confusiones.
- Podemos escribir  $a = b$  en vez de $a ≡ b$. Se interpreta como *"a semánticamente equivalente a b"*

### Elección de generadores y observadores

- Conviene que sean minimales, sino se corre el riesgo de generar inconsistencias.
- ¿Cómo elegirlos? Complicado, pero nos va a servir la ***igualdad observacional*.**

¿Por qué es difícil elegir los observadores (y generadores)?

Las descripciones del problema en lenguaje informal pueden tener algunas **ambigüedades**. Tenemos que tener bien en claro qué queremos saber acerca del problema. 
Ejemplos de ambigüedades:

- *"Se calcula el índice de desempleo"* ¿Y cómo se mide?
- *"En el mismo texto pero más adelante"* ¿significa número de página mayor o menor?
- *"Para transportar cosas se necesita permiso"* ¿Cuándo hace falta permiso? ¿Al salir, al llegar?

Diferencia entre especificación informal y formal ——> **brecha semántica.**

**La igualdad observacional**

Es un predicado entre instancias del tipo que nos dice cuándo son iguales. Hay cosas que son sintácticamente distintas pero desde el punto de vista de comportamiento nos gustaría que sean iguales.

Para escribir la igualdad observacional vamos a usar los observadores básicos.

$=$obs → igualdad observacional