---
layout: page
title: Teórica 8
permalink: /materias/orga2/teo8/
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

![teo8-1][1]

Un segmento es un espacio lineal de direcciones, entonces usa **dirección lineal**. De no mediar otra etapa, este número sale por el bus de address convertido en **dirección física**. O sea dirección lineal = dirección física.

En la Unidad de Paginación se debe habilitar seteando el bit **CR0.PG.** 
(Bit 32 del CR0). El procesador **DEBE** estar antes en Modo Protegido.

**Paginación**

![teo8-2][2]

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

![teo8-3][3]

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

![teo8-4][4]

De la unidad de segmentación, nos llega una dirección lineal.

Los 10 bits más altos nos lleva a la página que contiene la tabla que contiene a los descriptores de página. Los 10 del medio nos dan el descriptor de la página a la que queremos ir y los 12 menos significativos representan el offset dentro de esa página.

Las entradas en el DPT se llaman **Page Directory Entry,** y las entradas en cada Tabla se llaman **Page Table Entry.** Notar que estas entradas son descriptores de 32 bits.

**Primeras conclusiones:**

1. No es necesario que las páginas de un segmento estén contiguas, porque la traducción se encarga de todo eso. Si una instrucción queda al final de una página, al realizarse el próximo fetch, el sistema de Paginación generará entradas PDE-PTE diferentes de la intrucción anterior.
2. Por cada fetch, escritura o lectura de memoria hay que acceder a memoria 2 veces para leer el PDE y PTE. Tenemos que leer cada vez 2 descriptores. En la unidad de paginación existe un cache de traducciones, el **Traslation Lookaside Buffer.**
    
    En la segmentación se resolvía con los registros caché ocultos, ahora con el TLB.
    

### Traslation Lookaside Buffer (TLB)

![teo8-5][5]

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

![teo8-6][6]

Bit 4: **Page caché disable**. Para deshabilitar el caching de la página.

Bit 3: **Page write through.** Te administra cómo manejar la coherencia, cuando está habilitado el caching.

**Descriptor de una Tabla de Páginas.**

![teo8-7][7]

- Bit 7 - PS (Page Size): si usamos páginas de 4Kb o de 4Mb. En nuestro caso, como PSE-36 va a estar desactivada, debería ser siempre 0.
- Bit 6 - IGN (ignore): siempre en 0.
- Bits 4 y 3 - (PCD y PWT): lo mismo que en el CR3.
- Bit 2 - U/S (User/Supervisor): si está en 0, supervisor. Si está en 1, user. En los segmentos teníamos 4 niveles de privilegio, ahora hay sólo 2.
- Bit 1 - R/W: si está en 0, es Read-Only. Sino es Read and Write.
- Bit 0 - P (present): igual que en segmentación.
- Bit 5 - A (accessed): igual que en segmentación.

**Descriptor de una Página.**

![teo8-8][8]

Dirty: una página está sucia / fue modificada. Cuando hay que desalojar la página le vemos el D y si está en 0 no tiene sentido guardarla en la memoria virtual, simplemente la pisamos. Pero si está en 1, significa que alguien la modificó, entonces necesitamos guardarla antes de pisarla.

Global: la página es global. Cuando cambiábamos el CR3 (cambiábamos de tarea) se limpiaba el TLB. Pero si queremos usar páginas en varias tareas, para cosas que se utilizan muy seguido, podemos marcar esa página como global. En ese caso, el TLB no se va a limpiar por completo cuando se cambie CR3, las entradas global van a quedar ahí hasta que (muy pocas chances) sea la LRU.

O sea, las páginas globales no se ven afectadas por cambios en CR3 pero sí les caben las políticas de desalojo de las páginas.

## Paginación PAE

**Características:**

- Permite trabajar con páginas de 4Kb o de 2Mb y así direccionar el espacio físico completo (+4Gb), pudiendo combinar ambos tamaños.
- Actualmente PAE permite trabajar con 52 bits de memoria física (hasta 4Pbyte de RAM).

PAE se activa en **CR4**

![teo8-9][9]

El bit 5 - PAE hay que ponerlo en 1 y el bit 4 - PSE en 0.

Una vez activado, PAE permite traducir una dirección lineal de 32 bits a una dirección física de hasta 52 bits.

![teo8-10][10]

En PAE se arma una estructura jerárquica de 3 niveles. Hay más de un directorio de tablas de página.

---

**Quedan 3 videos (01:15hs)**


[1]: https://lh3.googleusercontent.com/KjOLS_NozOVnqe0GCu0Lp0r0gcrFmLLeT-eX2wUHJ-CHIHX6IVh5jy3jcfrO5Jd6PDGXQzVqD3aZME3bbUs7p4u6AbaAKxyLJfEFnvSeqbtkslZj1RCkvTjZbBaeW6_eQ10VW_Bas9TExKnZv8Q_UIlLDexF16ny3odnxpHUkClpVyX9dpx2jCmj36BqtyNZYhZ518pgafGDz0O2xN_KM5Ff_cy1Ubmj83hYZnvnXFtnxcDajmFkmhBa7SdFreO1mroqHfQf7ZEqG00V-nu4JEnHvjYfJiG2mYMYRgqAJwGRZffoIMrocEHF0EXfOIxObiZF2oKayf06Ho1zVcTtoH9m8bY1qA8cMfG4dM8cH52GR23C1gOlSiUY5dQQsyxfQNCOx6Fzq4rv8VhC-652mtU3WZs1aL10lD9c0Q7hCLgIO2z3DeoX29ouwYP9Yj9VWgO639xyb2cmGxC_vL-LtNW2GcQQl7E81OLNzzPD-BQ3shxz9Dz5Jrpa1tI9WAUmA7QUb24z40RBeiuMCVuFIUmCz7BPBgXodXOq7dT-od6uuQXsU0KDPv9hn4kSWgCqZ03DdgQN_S6gWE2XvT2Rylbfr_pAgr8a7izyH2bg0ptxbUIhrhLh7Jzd2zK2MhxuwckKum0NIDPrPrFpEGkuSFfeSrWq4eRF7qhALkwMTHZg3zekQsZG6dM08HjGBVCYSirw2D0GY2nrD_BQgZlhiUnvzZ37SE8dP2N15f93zodoafpIw87mx-XeZpvPi2jvIohSpI9LI2Ti1wsTOZxs4GfO-CcAjIsNiYoe9FRGZzPa8TQLrOt58QqA3Q6WG82C7cO8JUIVRvgm8-EDesRajYXkTiqKJs-0chEgxEIAyQ=w982-h438-no?authuser=3

[2]: https://lh3.googleusercontent.com/p4qk870Sawig5FjVZnwy1u7I4PuOUjWNajn48TRnuupEUPyx-zUKrVC7WDZpEl0EVzHjmx2g3HEYkCdfpNxSXTS7sq0k79ZnLnDAOPCokEf75T8dbi1Y-XuxbQmFyXKq8vEyBqC93Cjrvdb9nivLE3u0okUF4EKAQPOmuLO9CSr3gSFdj5i0HX_4I6bV0_718nEA3hVqYsg7r_uOd70mDt1BD1p3QSIsSfb9iplLL5g5G0mcRIJg8nuHEFdLjlMPkAU-G68J1DtlKm65Uc474zggKgpGiZec7RpIYWo8v8yJdC5CBWdH64GqiG67Lljv0PAgyWxdCzOLfALFS_mP4BQPFNWZlMHzcowb3ix2WOq3L8hi_MJ_CvuAwxXLF9AgyRaTcKXg9lzLgZbZUc9ejauICKAOTh6VYUf_JSKtds6BPlKedHJerftOi4BNbiHZe9gHowDpZ8R4splmI1KMX46SzFkLBfjHMkyTWyPuD-k_j7uG0UgxN1lmOhihJD2YHP12nN0K3_8qFn-066UxbFxslmIfKCRxC6qrf1O6wPeR9bNGF-XSYLPeu8mIrblnAMVcRyjTEee06C3FEGc0DTC61YD-ucdgXTvPpMw8fXxnVfkKjvHqi-DbP3DWQqnDfQrXLluSsxPGfRwYV6fnRcZv3njFLLzCZlfODi6Do43w5ul52XKmsrR_6ivxqaqdIXbQlBxbnb1OOv8AKsRZ5g5nP8nVJhqV9K6U8sYgTqg-_PB7vmPXAtoS74OXAW8Vp6kPQTX_rybSw5UFfptu-Ng9CBKONb0xV0CIMPksyGCXNmedqVm-wM_s66uOxGmeXp_5MUkEDetGyP9pfC7lqEs0DfBwF611s4rXmlNx6w=w898-h654-no?authuser=3

[3]: https://lh3.googleusercontent.com/5drsmIYiGqPhnfKnBWJ1aBnAnyS1XOd4oKbUw8wgQHhNOrTe1ngXCSq-z1V3IJ7_J1gHKyJeAec5thcnODoi_Qa1lWAWySv_gc4ek0-GtLABZMJqaMhuGfLOIvf3TC382e4qtDWEEtFbDTmI4Go0Hxa4PAUrX_FJ1UzzjfwKjBpgKJfa9Ax5CjdSCHp4JG9xXEa_tE7-qv9SwxlsgHrRu4ilORD9Q002pvskZKqnuXSWEoxUIlJAdxJ7qWGIrZQgvRlYeaTrNunX9YJmIxMma0dE34W368ddXpplCcgfXvFZBwoEQTzL6s0BRNkDYxW56RXhlhhRObvnR0r-gSgbqZy2UL2xTAxuKrTowY9XMRk0dq1RUipojLi-WJb5PSI0fx4VKxCCAtOjp1ZozTII9fwK7uR_Sy8T-_UTnBjrzar-e7w-7JK-herw6xOajViVn6sgQ8ybiuHggRFo29tQbI5sZr7LS9ltk53riuuNxigucYIWKgsWZpD7H2M0-6ucEu7uzFhrgADjm4r8ciaEzZn6fw2B3ZQWDQeuHApcYDl8aLphuQQdOh0Y1S4vRt4OZpEGVxKMI_YAnqjPd0d739aWrB0SDWJSUIq2hsVEbg5r4kSSJ-CqNNB61Wv1Hz8_CykvgPSzlki48IHBU4_ZzKLA5BjTa6QG75vOlU8Eo6GcEBtD4vkG6hru_M4l2VyLwLv5DkzeSktM9Zp-MApdmppQRsCh9A1xh_zevUuMuqSllmEVCJtK1z01MBx5fhGXYiJ_u1_QYr5M8_9ImDyTtySeE_5r37h9802oONsuqKvhxErED96y-pWFVlfOOSBSgZNohLrQFCLRb9MBgYNQpZ_YB3mXPiCbbOlueBYKkA=w1122-h714-no?authuser=3

[4]: https://lh3.googleusercontent.com/6MSyoruylk2Vf_3oEH_y7KWJfwr4yVwIaRD7A1cM_PsAYvGT_J5aJtjUABRW73voqKtHiQHHvwrhWRsddnVFslrGCtaXM1oshdqr1MRZ43hqlpI1vkA9VVNIpCKFtprOGU0x8NS5E4OMyqY703W8789oTK3DJiDiN0tp1nRWo17sB3AHEOvAMv0gqTRLzCbA9Fc8NnYovasUcTJ7hsnMzrvJTtGPdVQ-pizLK9V43JWG3Msfom0Xvz9aCZbgIHS0sl0Yc8jX9t8394H3cWlqZ4Gze4r-ZakWfc5um9Y84deeHEpH8FqXkz74JbtkD73AgNljGhZ_JJt1Jjamhq4HQ21G-MZ8-e9b3CBb6cL7bw7D2_s1cnmJyLcmx7IJZHGE-MzlEZNNTTBh5fpN58SNFu4mMjpK1NmH4H2y46YBxbIgiu3k2cNXaOqACoUos-TEhku8dwBjSU_TDBOdkEqyUAB8EZeGkNEiNA0A433NZtzDbrcJyRaS2XInWTtwMR5Tn10_WGEqeQ_fp9w-IexvmMHT-EHulOCtWJwjQZFSiSgh6pZrz1MHw1-e8chrwUL8K5KRydsxZqgcDC5ItdXB7tQa1WcQUQBI6nd3h6y4TosbM6VAXQofDzfT6YI-rXxYXMX7X_siXJvnMaI1VwiNpjPu-3g-rVRG6rf6NFOSkjAv8CF9WLoMe7FfUdDt5_Plp4rdR7wMZXlJKT1UJ5UrVvBakb-7PNFlnqBf94gHPBjY639XBJleL8Y6EDhm1YNi9tIkDrANp636VceT8O7ZGqpgchGVg82IkRrZmsgWMoFTGDS3sAcRp5us3cWyknTcfYOHPhpKq9mxJMT22PnZ40ogeE04m7lUHmSEGH3urw=w1112-h696-no?authuser=3

[5]: https://lh3.googleusercontent.com/_xL4Gr-3KsYPrG24zwyFu5jGp3vDxGsiBgKknAiff7z5lMwdC87KRniRtGP8R8HNjSOsS08oYkaCeks-Zzly5zCo73rS_iCnx8JTiQke-VRsY-9roIFKQYuPkDtdTd-vjNFmmHMKH3wCSUgdpntJS5JkhyzxzegUuRkN71kY5ymfVyF_m79otxcwjY-Js5uZXVhG2fhINAyD_V9JiTeQ9kjV48xmUwecXvxwXKFbCAjJ-pZ5NGUFZW0m_x2xFqSm_r-DeplArrxy6_sGaP-P6z0l-_sFCJGMcEkdWSruh8dmmJpNLx3zA_YbbAhy9jUhHfBGPtnTEN2ihj3D0HTk_4XWUrrMJfhSLvIn1dJOJZ1T-s1LkdI-z39oFitRV-t45kDuKLGZpTTdNznnkSQufJJbPTD8x-A6dd4Cu2EjNi76Bq_xUa6vDVHkq6twKXyLYIA3Th6kEERK5eKqrsjhgtcWo3GAlgv1kYjMWop1x87gJdnxRFArkmL0Uhuni9MQXItFlCyHaAm4smSS1L50OJO2qeTD7YFoaEAOFz-hBXU01zbOXq-89fxoMRMAaYwapvE8GfQwL9XXWVg3-YzjBVxUbDC2XH0hJRByIh0hCxwg_aCiqNiEaYqlgCwxAvJe_kmB7T9JEidqmR0kyAJYUZqpDOId7QnLkZehyuKbx8SevUL23iGc_QFlKnXfxLcezo09YAOcXDElFdedsXqFgspoaQyMLVJKEdPGLamteeUcql49s5RVxj2hggEBnkMlnyZtfLzJznYO85_uJr-PI1xSWmYLAOQc0fyT6-SGmJOm6S2N_0sZl7I0pC5xr0OGTGlRAWRn-8QRaeVzBetKl6mkM0j1kUPRfXajrmb9BA=w894-h652-no?authuser=3

[6]: https://lh3.googleusercontent.com/xVK_CrV9I9CjFdvlt3Jr6lEyhAWI5GBs6hiRcUrdQieoHjthqIP05hT5IXst2cZTtDEOI03wkeZHMFELa9tRvIRAO23SofnuJ60O0DlHbOVDULTs52ux-aUhVQSBq-o93LoipUoUoHxI3DKgohUbwW27rEHl1kXXw7C02qcXone4KY0qjhtnqiyox4ErBTCETdS4nxb67uUqLFYCOMxmbitRUpxGskwmmabkmVUElZ6ymuzbM4IHcvyOf6BKAefoAbdFxDUHpjarP04C7p_JnfP3azIebj3aQROVbVZWD5MXn50ObCLXv6729BPkcR5SqETyPl7PpSwlRIjBW7klYCjYaFKxovIYUF5a1Ox5XSw-ibq2UcJ85ilUpCn4TMVGtZs5huMVBVu06mFvjvU7cTSHPWfkZ0j1qr5HzgY1xF0FzyCgxvDi_14bzCKhAXMMg8S9D8VDYwlnQqBY7rbaDZKSLUGO4VbLXuhdGzpxR6Mrn4-ievc9-Vy3MAYTpaD-5itobNFqAdRvvq4YKQAzvbkP_ky6baLkeFlkWcPy38GqgIzoxXVGOnrOPjKwrnyTz_OQb3DrlUNDNfhQLVq6dG6nTcNYP--IgKXtFrVww9O88T64GASh-tknUqy_GIYSU8KBNlFrlVX5np3As_poxUIB1PkF_3m6ayYW94bw5WOEyT7Zg9YWk5a5lyplGieCA8l4Sn0D-WVx8pvIffHDhSXdZj-tdhpJQgrMezYWWCj_gfZF3c_XdGb3ELpEVvVwddnTakVjsfH_tfw5IPytpcF9SB91AbChaJytTMZyVHx-1DMy8gCFS4UQPwCCUr1OCQEs2G8ij0Ufvm0ES22jiI-RjqRGDPzaPxPGHV49AQ=w1108-h132-no?authuser=3

[7]: https://lh3.googleusercontent.com/m8YcGQ9n6xNbz6nsHk8m_3fvToQqWP85Zg5A6PYdmcnudTL-8OhnY7wDFpVJkeZAoX1-dxBNX4pD2HJh43Ft0fGD0PHDvaDffM6tYeoTK3Plk9r_0ZYA2JJU8wET4eNqjZyVsUvleDEfkfWPaM2Eo-IGG75l3ENvpZXvSQc6J1_eKFyHrDoumTldqjiLMhkmI7aEYpGB_k2Hl0Zmdd-MVO6lNCRk5rwez3IMoSUJzN92Oa8NrMT1wASmC8bfNLIUmibNXfaro6xBpgdMVu72L4_9d22AfFV5K05KAXrEKMrLaq4dUeTZLTyh5S6n1PCj2Hzc9u2yKeWObdm5QjByaAQCCsqqz1JcIvgPzcZYpaoRCc7XAyaNAv-HTCMQxsbA2dTELIiB-fQpmSxJYrxIMD7muZ7pBo2v9DjjWbT7M6gt5wvXtMFenaNzQJN1K4PNZT1o8wzqlISDaEOE60PceRbZV15U4uZXVHE6XB6oJTpt-WPdNNZ7NazBWgUBl8PAtb3HySYQ9Dp-1SQhn3zqhSMoGEy17zlQdDmy40owZWSHAL_G_AY94o__CqhLZpOaDHrHyU8W6oyYBaHLQE6g8TqOXdsYrZ30fqNJp8JzUIV4NzQi7yZeaT_DVn4iot5Hn5pjid3oPhq4l3ftSfl4eo8DAD7GntbWpxcW9X8LOHwCuSPUlfe7NIluIufzhHLM-_QgTARvwVuAFSpkIFDkzW9VvizWnezbdRZcYs5QtWoP1WNjD5eCFCQSC3sq1ukIChmGwtBrhwplssDXoU8SpjWl2GFLPdPEepEErBHTbUwKyrg5U6IGaiPQzLW4ihTeLoE8K1Rgj2dli4qOefaW_s-dexwCoK-Gnx0fnFOiNQ=w996-h150-no?authuser=3

[8]: https://lh3.googleusercontent.com/Oge8kTbFDYnz14LgJb1zw7mbYS3mWSSOVDCKIDwmfhDuY3FvIOVBEPEgcjYXuwPmWydzI2CA6eY_mVMpZX9gHraNgOsZ0hgZA6Nm_1WEhqAegaHZTqeSQzy9E5WDdcwDBMKXnmFbGL2FVKR8cu3ppGPUb9dZnNBKORQIYbhPCrHQJ-n1NYnjO-V6IAFHUZe1kzd6s5ETVIQWssOlbthbOjvBRTiUk7NGTyhNBsfwDsLZMpd0-HSUsKEWjr4PpQP2qcPAlhZSdOUHqpTsjIeTd4vuwWKmj8rqIAgvOW-6U1yK3wxZ90saPczQaPrO5NZZgyxeIIkXlbTBN5FosL3u9Ejtbp1ZMdb-EffkQM54hibmvkjo7vm9ekkUkzCw9kp47ZhSZ6iKE99VgvXO7zKBl1mJPOdr0u-wMdXkP4ylirZYJldYUwLOljNenp_ZUUJfD32jxnldn-S2v_RHRv9o8ManK_XVr4THz4dZWLkRO0Ofw9OV063ShnMmxqj-As9U7yGxutKM61SzOWDbzdpv9rPNQnUPLRSUbZyj7A5VqOMQ1RSU1jaYsZR6hYZ3BQF6UG1alH8LPRIAw8yW-Udim5q5i3a5wxOpPBpGXjAp96AFrp4khGVaVxG080B87gBWA7-tNom16T-aiZFJZNvym6CtO9-urx5BwYJ4l6gg4GSZeJFD-qgxCXVOxX2s_eX8BgFN_bESpqlqt43hfMG2TpeDsjMVkGFI3JOdwaaiq8phdEkc4tJhDjPzIqXNIwv_womdtloTYLdblLi3o2cX6iHMtupjSaX7UBP_qZ8U4ZPWHUlvqKmAtPnOQNTdzbQuYAG9jcN5ShDxS1qBHL30yj9Rpv6JGLp_e-K1Ctti1A=w1096-h184-no?authuser=3

[9]: https://lh3.googleusercontent.com/zlzjyIcKuPuLJdhSQkcgPnQhcqIr8BzQrywgGSLAo_80VC7XA8vf_iDul90fvN-xXoxvwTfKbDgXNTIwRZ3Lpeczm1tiPdgZQ3UNUrvxp8g-3VDzx01J7NG-jqiZRczfcH3_Lf7g5PYW2cvLWv-MVmtXWVuz5C27CQ9e9HXNRpA_2nWgBy9y3YqdmgAYm_CgkbElAtY52tcWu4bpXajNaP6mYXCPIBHWC9W4ISzpa8e1H6DVbykw3zDCFjfiHX7VAI_hRMRQc9b9jRt7dKzSBzSWM2wOjuutNfxQRi2ssNiEE356zF4i9Uljomo2Qz-TghU-7gfMtONVOx3uz4q6njt1uJo96bCQZ5zONplSYZT1WBzwRi_YnQmF9Siblita97jLyC2coJwi_7swQobaV3FW4ER-bfkw5nTpq5fZ8zbhUN3XzXSmHTYCgAS7wVN5XdUCbSKOcJdPhCndM3ZH2HCwZcrO2wXpwcJmlFOpE7YU34f9SPG-x9d7d8H2lfhsogm2NTMqkrvYr2ku35nsioPLs-sBoNI7BOBS_ZecKG-7_6n0VYapy7x45Hh79W11XjH70MxhvLKDeHaA8-wecaVKO64GZOKO1dkonZsqEfsluIKqiLQnGe9tMI86xKefqDdXWBMfGRQrAt8Q5_Dre8dGzHFxFI1SsZH77aIptXvCThi5NoyYaTcXljYl1OQygCydO2ew4SSAxTJKegYzfaOa6IIH3s2gsH9lwygj26AV39paqS1qB6ZF_PIe1CeLnJg3cA6beAA61jZDQyVTna5oz19wj9tLzOt06obtm_9ewRR1TSJk47_sSboYIyAUX9WBXdeU2N81XOFnpVgz3z4PctuhI0p8FzGP6mGTqA=w1100-h124-no?authuser=3

[10]: https://lh3.googleusercontent.com/m8AenOKoFR69pZ2e4zunvNNv2JO63uzc7HoiDsbxRuG9JK8-BpFp8ZGt8VP3idvEgvDsSqLSB5CAgXqyF891FIxxkxvF6VPs4DPAOKkee1xLxzL71U_j535JmxnoNC5qZB0Xm_iPsVMy3epCbwD1-DN3FBhZnP0HW-vEOArbzSwcQxE1BfUfo-w6RFxbeYahPrEmVk3b1Vs5NPR1iq92gYth0lbN8xElRstNgC--T_eXbj4agcmrTGF_C1oocDdxvAe16XPk3850wX1UwG-V80LAzjoiupHGODbxp30TfIjw09E8eu5hb4G0X6K5qirJCBTX9wbBw-j0K0d1jW018MxGnwryS50k1qKllCqKQhTeLshHEosz9bBkn1e1tIs0qo7ohNwk3W3giR0o5OhgS4KA68H8KZkzXU_MXz0a2JV3d0LlduYkNEMHLCWp6AsvrbGwYVf1aHAMMMXAbTcNKczwF4mlWRZ3WwIHq7hOLfvhzSbVfcBFmQ051-4wOwRrkQrdc4xcPUvrC73ivi2g1jy5jxhpCC7n7q-8nTKEVUiMVieyHwUjD3bHqztCdiHjQyJZpPK8qmcMRL37sFnnyqRNwDvg79bm6-EuFHkWrjEVamt_uj6vdFPR4Ni4qXQ1mBXIOENGyFfuDv2km4SVQJSrI20m8Px-EMQEW4HvorTtV8WdRNiFEcIeqAVlstmRelgGQvRcL-M3amekRuLqiBA9N110UpFPBpDhgr-ESYkSpY-QtfSDzc7I6Jag7L7lU-IbNtG7qyUCSy0yl6S5U85e7Go3kZ6U4xIZalJ3vyzJGmpLlBbdG44E-Fa0bo_zHVfIZ6mlmowZhBGQuH4WoFCS-nWkkPZAR1YxSxJHDQ=w1104-h138-no?authuser=3