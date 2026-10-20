# P2.2 · Sentencias iterativas y saltos

<div class="ej-gate" data-unit="u02" data-nombre="U2 · Estructuras de control"></div>

Practica `for`, `while` y `do-while`. En estos ejercicios **no uses métodos que hagan el trabajo por ti** (contar, invertir, buscar...): programa tú el recorrido. Puedes usar `length` y el acceso por posición para recorrer una cadena. Para cortar un bucle, usa una variable de control (ver [1.6](../u01/06-bucles.md)).

## Ejercicio 2.2.1

Pide una palabra y muéstrala 10 veces.

## Ejercicio 2.2.2

Pide la edad y muestra todos los años cumplidos, de 1 hasta su edad.

## Ejercicio 2.2.3

Pide un entero positivo y muestra los impares de 1 hasta ese número, separados por comas.

## Ejercicio 2.2.4

Pide un entero positivo y muestra la cuenta atrás hasta cero, separada por comas.

## Ejercicio 2.2.5

Pide una cantidad a invertir, el interés anual y los años, y muestra el capital obtenido cada año. Cada año: `capital *= 1 + interes / 100`.

!!! note "En C#"
    Si lees el interés como entero, `interes / 100` es una división entera y da `0` (con `5` sale `0`, no `0,05`) y el capital no crece. Lee el interés como `double` o divide entre `100.0`.

## Ejercicio 2.2.6

Pide un entero y dibuja un triángulo rectángulo de esa altura:

```text
*
**
***
****
*****
```

<details class="sol" data-key="p2-2/2.2.6">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("Altura: ");
int altura = int.Parse(Console.ReadLine()!);
for (int fila = 1; fila &lt;= altura; fila++)
{
    string linea = "";
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 2.2.7

Muestra las tablas de multiplicar del 1 al 10.

## Ejercicio 2.2.8

Pide un entero y dibuja este triángulo (cada fila empieza en un impar y baja de 2 en 2 hasta 1):

```text
1
3 1
5 3 1
7 5 3 1
9 7 5 3 1
```

## Ejercicio 2.2.9

Guarda una contraseña en una variable y pregunta al usuario hasta que escriba la correcta.

<details class="sol" data-key="p2-2/2.2.9">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>const string secreta = "csharp123";
bool correcta = false;
while (!correcta)
{
    Console.Write("Contraseña: ");
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 2.2.10

Pide un entero y muestra si es primo.

<details class="sol" data-key="p2-2/2.2.10">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("Número: ");
int n = int.Parse(Console.ReadLine()!);
bool primo = n &gt; 1;      // 0, 1 y negativos no son primos
int divisor = 2;
while (primo &amp;&amp; divisor * divisor &lt;= n)
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 2.2.11

Pide una palabra y muestra sus letras una a una, empezando por la última.

## Ejercicio 2.2.12

Pide una frase y una letra, y muestra cuántas veces aparece la letra en la frase.

## Ejercicio 2.2.13

Repite (eco) todo lo que escriba el usuario hasta que escriba `salir`.

## Ejercicio 2.2.14

Lee enteros hasta que se introduzca 0 y muestra la suma de todos.

<details class="sol" data-key="p2-2/2.2.14">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>int suma = 0;
int numero = 1;
while (numero != 0)
{
    Console.Write("Número (0 para terminar): ");
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 2.2.15

Lee enteros hasta que se introduzca 0 y muestra la suma de los **positivos**.

## Ejercicio 2.2.16

Lee enteros positivos hasta que se introduzca 0 y muestra el mayor.

## Ejercicio 2.2.17

Lee un entero positivo y muestra la suma de sus dígitos.

## Ejercicio 2.2.18

Pide enteros positivos y, por cada uno, muestra la suma de sus dígitos. Termina al introducir -1 y muestra cuántos de los números eran pares.

## Ejercicio 2.2.19

Muestra un menú: 1 – comenzar, 2 – imprimir listado, 3 – finalizar. Si la opción es incorrecta, avisa del error. El menú vuelve a mostrarse tras cada opción; las opciones 1 y 2 imprimen un texto y la 3 termina el programa.

## Ejercicio 2.2.20

Pide una frase y una letra. Recorre la frase carácter a carácter: si no coincide, indica la posición sin coincidencia y sigue; si coincide, indica la posición y termina.

## Ejercicio 2.2.21

Lee los importes de las compras de un cliente hasta que se introduzca 0 (no se sabe cuántos serán). Si el importe es negativo, no se procesa y se vuelve a pedir. Al final muestra el total a pagar, con un 10 % de descuento si supera 1000.

## Ejercicio 2.2.22

Lee enteros positivos hasta que se introduzca 0. Por cada uno, indica cuántos dígitos pares e impares tiene. Al final muestra el total de dígitos pares y de impares leídos.

## Ejercicio 2.2.23

Lee títulos de libros hasta que se escriba `*`. Cada vez que se escriba `/`, termina una línea. Por cada línea completa, muestra cuántos dígitos numéricos aparecen en total en sus títulos. Al final indica cuántas líneas se completaron.

```text
Libro: Los 3 mosqueteros
Libro: Historia de 2 ciudades
Libro: /
Línea completa. Aparecen 2 dígitos numéricos.
Libro: 20 años después
Libro: *
Fin. Se leyó 1 línea completa.
```

## Ejercicio 2.2.24

Lee números mayores que 1 hasta que se introduzca 0 y muestra cuántos eran primos.

## Ejercicio 2.2.25

Pide una frase y muestra su palabra más larga (la primera si hay empate) y cuántas palabras tiene. El separador es el espacio, y puede haber varios seguidos.
