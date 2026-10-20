# A3 · Ejercicios de concurrencia y asincronía

<div class="ej-gate" data-unit="a3" data-nombre="A3 · Concurrencia y asincronía"></div>

Practica tareas simultáneas, resultados y errores, límites de tiempo, reintentos y datos compartidos. Cada ejercicio indica la **salida esperada**. Las esperas son de decenas de milisegundos, así que los resultados son siempre los mismos.

## Ejercicio A3.1

**Tres descargas a la vez.** Simula tres descargas con una pausa cada una: `uno` (200 ms), `dos` (50 ms) y `tres` (100 ms). Cada descarga imprime `termina <nombre>` al acabar.

Lánzalas **todas a la vez**, espera a que terminen y comprueba que el conjunto tardó **menos de 350 ms** (si fueran una tras otra tardarían unos 350 ms o más). Imprime `todas a la vez: sí` o `todas a la vez: no`.

**Salida esperada:**

```text
termina dos
termina tres
termina uno
todas a la vez: sí
```

!!! note "En C#"
    Usa `Task.Delay(ms)` (no `Thread.Sleep`) y `await Task.WhenAll(...)`. Un `Main` con `await` se permite al nivel superior del archivo.

<details class="sol" data-key="av/a3/A3.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>using System.Diagnostics;
var reloj = Stopwatch.StartNew();
await Task.WhenAll(Descargar("uno", 200), Descargar("dos", 50), Descargar("tres", 100));
Console.WriteLine($"todas a la vez: {(reloj.ElapsedMilliseconds &lt; 350 ? "sí" : "no")}");
async Task Descargar(string nombre, int ms)
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.2

**Sumar en paralelo.** Reparte la suma de los números del `1` al `100` entre **dos tareas** que trabajan a la vez: una suma del `1` al `50` y la otra del `51` al `100`. Espera las dos y muestra la suma total.

**Salida esperada:**

```text
5050
```

!!! note "En C#"
    `Task.Run(() => ...)` lanza el trabajo en otro hilo del *pool* y `await Task.WhenAll(...)` recoge los resultados.

<details class="sol" data-key="av/a3/A3.2">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Task&lt;int&gt; primera = Task.Run(() =&gt; Enumerable.Range(1, 50).Sum());
Task&lt;int&gt; segunda = Task.Run(() =&gt; Enumerable.Range(51, 50).Sum());
int[] parciales = await Task.WhenAll(primera, segunda);
Console.WriteLine(parciales.Sum());
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.3

**Con límite de tiempo.** Escribe una función asíncrona `consulta(ms)` que espere `ms` milisegundos y devuelva `42`. Después escribe `intentar(ms, limiteMs)`, que espera el resultado de la consulta **como máximo** `limiteMs` milisegundos: si llega a tiempo imprime `resultado: 42`, y si no, imprime `tiempo agotado`.

Pruébala con `intentar(300, 100)` y con `intentar(50, 200)`.

**Salida esperada:**

```text
tiempo agotado
resultado: 42
```

!!! note "En C#"
    `tarea.WaitAsync(TimeSpan)` lanza `TimeoutException` si se pasa el tiempo.

<details class="sol" data-key="av/a3/A3.3">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>await Intentar(300, 100);
await Intentar(50, 200);
async Task&lt;int&gt; Consulta(int ms)
{
    await Task.Delay(ms);
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.4

**Reintentos.** Escribe una operación asíncrona que **falle las dos primeras veces** que se llama y funcione la tercera (devolviendo el texto `listo`). Escribe `conReintentos(maximo)`, que llama a la operación y, si falla, la repite hasta `maximo` veces en total.

En cada intento imprime `intento N falló` o `intento N correcto`; al final, `resultado: listo`. Prueba con un máximo de 3.

**Salida esperada:**

```text
intento 1 falló
intento 2 falló
intento 3 correcto
resultado: listo
```

<details class="sol" data-key="av/a3/A3.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>int llamadas = 0;
string resultado = await ConReintentos(3);
Console.WriteLine($"resultado: {resultado}");
async Task&lt;string&gt; Operacion()
{
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.5

**Un contador compartido, bien protegido.** Lanza **5 tareas** que suman 200 veces cada una a un mismo contador, y muestra el valor final (siempre `1000`).

Hazlo de forma que **no se pierda ninguna suma** aunque las tareas se ejecuten a la vez.

**Salida esperada:**

```text
1000
```

!!! note "En C#"
    `Interlocked.Increment(ref contador)` o un `lock` evitan que dos hilos mezclen el «leer, sumar, escribir».

<details class="sol" data-key="av/a3/A3.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>int contador = 0;
var tareas = Enumerable.Range(0, 5).Select(_ =&gt; Task.Run(() =&gt;
{
    for (int i = 0; i &lt; 200; i++)
    {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.6

**Productor y consumidor.** Un **productor** envía los números del `1` al `5` y un **consumidor** los va recibiendo y sumando. Se comunican por un **canal** (cola, *stream*...), no por una variable compartida. Cuando el productor termina, avisa de que no hay más.

Muestra solo la suma final.

**Salida esperada:**

```text
suma: 15
```

!!! note "En C#"
    Un `Channel<int>` (`System.Threading.Channels`): `WriteAsync`, `Complete()` para avisar del fin y `await foreach (var n in canal.Reader.ReadAllAsync())` para recibir.

<details class="sol" data-key="av/a3/A3.6">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>using System.Threading.Channels;
var canal = Channel.CreateUnbounded&lt;int&gt;();
var productor = Task.Run(async () =&gt;
{
    for (int n = 1; n &lt;= 5; n++)
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
