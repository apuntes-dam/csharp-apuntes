# P2.3 · Captura de excepciones

<div class="ej-gate" data-unit="u02" data-nombre="U2 · Estructuras de control"></div>

Salvo que se diga lo contrario, el objetivo es **que ninguna excepción llegue al programa principal y lo aborte**. Controla cualquier dato incorrecto que pueda introducir el usuario.

## Ejercicio 2.3.1

Pide la edad y muestra todos los años cumplidos (de 1 hasta su edad). Controla que lo escrito sea un número.

## Ejercicio 2.3.2

Pide un entero positivo y muestra los impares de 1 hasta ese número, separados por comas. Controla las entradas incorrectas.

## Ejercicio 2.3.3

Pide un entero positivo y muestra la cuenta atrás hasta cero. Repite la petición hasta que el número sea correcto.

## Ejercicio 2.3.4

Pide un entero. Si la entrada no es correcta, muestra `La entrada no es correcta` y vuelve a lanzar la excepción capturada.

!!! note "En C#"
    Usa `int.Parse`, que lanza la excepción; `int.TryParse` no la lanza y devuelve `false`. Para volver a lanzarla escribe `throw;` (no `throw e;`, que pierde la pila de llamadas).

<details class="sol" data-key="p2-3/2.3.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("Escribe un entero: ");
try
{
    int n = int.Parse(Console.ReadLine()!);
    Console.WriteLine($"Has escrito {n}");
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 2.3.5

Pide una contraseña y, si no coincide con la guardada, lanza una excepción propia con el mensaje `Incorrect Password!!`.

<details class="sol" data-key="p2-3/2.3.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("Contraseña: ");
try
{
    Comprobar(Console.ReadLine()!);
    Console.WriteLine("Acceso concedido");
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
