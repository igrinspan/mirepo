---
layout: page
title: Teórica 2
permalink: /materias/orga2/teo2/
---

# Teo 2 Orga 2 (02:20)

Teo2. 30 de marzo 2021.

## Lenguajes de programación - conceptos

Los microprocesadores distinguen 2 estados (Verdadero-Falso, 1-0, xV-0V). O sea "hablan" en binario. A nosotros no nos sirve mucho (muy difícil programar y reconocer errores).
Por eso usamos un lenguaje más humano, el **lenguaje ensamblador**. 

### Lenguaje ensamblador.

- Es el lenguaje de máquina
- Una línea = una instrucción
- Cada instrucción tiene un nombre alusico a la operación que realiza (en inglés y abreviado)
- Con ayuda del **programa** Assembler, se convierte ese texto a números binarios
- Lo que escriben los humanos se llama "**código fuente**"

Para lograr el mismo programa con mucho menos código fuente, se crearon lenguajes de alto nivel.

```wasm
global main
extern printf

section .data
zHola db 'Hola Mundo', 10, 13, 0

section .text
main:
push dword zHola
call printf
mov eax, 1
int 0x80
```

### Lenguajes de alto nivel

- Cada línea pueden ser varias instrucciones del procesador
- Aplicaciones de mayor complejidad con menos texto
- Con ayuda del **programa Compilador** se convierte a números binarios, transformando cada línea en 1 o más instrucciones.
- También se llama **"programa fuente"**

## Primeros pasos en C

```c
/*esto es un comentario*/
#include <stdio.h>
int main(){
	printf("Hola mundo! \n");
	return 0;
}
```

- En un programa de C hay dos elementos básicos: **variables y funciones.** Las **funciones** contienen sentencias que definen las operaciones a ejecutar y las **variables** contienen los datos que el programa tiene almacenado y tal vez modificará.
- Hay una función obligatoria **main** que define el punto de entrada del programa. Es la función que organiza el trabajo.
- **stdio.h no contiene el código de la biblioteca**, sino que es un archivo en el que simplemente se declaran las funciones que componen la biblioteca. Así, el compilador puede conocer la sintaxis correcta para su invocación. El código fuente de las funciones que componen la biblioteca, **tampoco está en stdio.h.**
- Las funciones pueden recibir una lista de valores que se denominan **argumentos.**
- Entre los caracteres **{ }** se encuentra el **cuerpo** de la función.
- **printf()** es una función que imprime en pantalla lo que le pasamos por argumento. La función no está en nuestro archivo fuente, está contenida en una biblioteca.
- \**n** es algo que usa **C** para representar el caracter **"Nueva Línea"**

# Ciclo de desarrollo

![Teo%202%20Orga%202%20(02%2020)%20a021ced03da647fdb186fc1150c0f11f/Untitled.png](../teos/teo2/Teo%202%20Orga%202%20(02%2020)%20a021ced03da647fdb186fc1150c0f11f/Untitled.png)

1. Codeamos el programa en lenguaje humano, como por ejemplo **C**. (Programa Fuente) "hola.c".
2. Luego el compilador lo transforma a un programa objeto, escrito en binario. "hola.o". Aún no es un binario crudo, tiene información adicional, por ejemplo indica que hay una función externa que se debe traer.
3. El linker (ld) es quien toma el programa objeto, accede a las bibliotecas de objetos que tenemos y resuelve las referencias de nuestro programa a esas bibliotecas.
4. Como producto del linker, se crea nuestro programa ejecutable "hola".
5. Una vez que tenemos el ejecutable es muy probable que no funcione bien 🙂 . Entonces lo ejecutamos en un debugger (ej: gdb).
6. Seguramente encontramos los problemas y volvemos al programa fuente. Por eso es un ciclo.

## ¿Qué hace entonces cada herramienta?

### Compilador

- Analiza sintácticamente un archivo de texto, que es donde está el programa fuente.
- Si está escrito de manera correcta, genera un código binario para ser ejecutado por el microprocesador.
- Además de analizar, reemplaza los nombres lógicos que usemos para variables/funciones por las direcciones de memoria donde se ubican las mismas.
- No puede resolver referencias a funciones externas al archivo fuente. Eso se hace en la próxima fase.

### Linker

- Toma el programa objeto, lo enlaza (linkea) con otros programas objeto y bibliotecas de código y genera un programa ejecutable por el Sistema Operativo.
- Enlazar significa:
    1. Ordenar código y datos en secciones comunes para luego guardar ese conjunto en un único archivo ejecutable.
    2. Una vez ordenado, resolver cada referencia a variable o función externa. (ej: printf).
    3. Identificar y marcar el punto de entrada del programa (la dirección de main).
- Es una fase crucial de la generación de nuestro programa, aunque no lo parezca.

<aside>
⚠️ **Warnings**
Son advertencias. No impiden que se generen los archivos objeto y ejecutable pero no debemos ignorarlos. Debe prestársele especial atención.

La opción -Wall en las líneas de compilación y linkeo, indica que las herrramientas presenten todos los warnings que hay.

</aside>

Otro programa:

```c
#include <stdio.h>
#include <math.h>

#define N 1234567890

int main(){
	double res;
	res = sqrt(N);
	printf('La raíz de %d es: %f ', N, res);
	return 0;
}

/* Para compilarlo: gcc -c sqrt.c -o sqrt.o */
/* Para linkearlo: gcc  sqrt.o -o sqrt -lm */
```

"**-l"** sirve para especificar el nombre de una biblioteca.
"**m"** es el nombre de la biblioteca math, cuyos prototipos y todo eso están en math.h.

¿Por qué no hicimos lo mismo con printf?
Porque por default el linker ya busca en algunas bibliotecas de código, las más comunes.

### Vemos código ASM...

Declaración de variables/constantes → section .data

```wasm
section .data
msg db "hello world!", 10  ;10 representa nueva línea
msg_len EQU $-msg
```

Etiquetas

section .text

Inicio del programa (_start:)

Nemónicos (nombre de la instrucción)

Operandos (registro, memoria, constantes)

Comentarios

### **Faltan videos 0204, 0205, 0206 aprox 50'**