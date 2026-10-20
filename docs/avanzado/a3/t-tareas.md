# A3.A Tareas a la vez

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5, en especial excepciones y clases. Aquí las esperas son de decenas de milisegundos para que las salidas sean siempre las mismas.

Un programa que hace una cosa tras otra pierde mucho tiempo **esperando**: una respuesta de la red, un archivo, un temporizador. La **concurrencia** consiste en aprovechar esa espera para hacer otras cosas, y la **asincronía** en no bloquear el programa mientras se espera.

## El modelo de C#: `Task`, `async` y `await`

C# tiene un modelo muy extendido y fácil de leer, que han copiado otros lenguajes:

| Pieza | Qué es |
|---|---|
| **`Task`** / **`Task<T>`** | Una operación **en curso**, que terminará (o fallará) más tarde; con `T`, además producirá un resultado |
| **`async`** | Marca un método que puede **esperar** sin bloquear; devuelve un `Task` |
| **`await`** | «Espera a que esta tarea termine y sigue»; mientras espera, **el hilo queda libre** para otras cosas |
| **`Task.Delay(ms)`** | Una espera de verdad asíncrona (lo contrario de `Thread.Sleep`, que bloquea el hilo) |
| **`Task.Run(...)`** | Ejecuta código de **cálculo** en un hilo del *pool* |
| **`Task.WhenAll(...)`** / **`WhenAny(...)`** | Esperar a todas / a la primera |

Un `await` no crea hilos: reanuda el método cuando la espera termina. Los hilos solo aparecen con `Task.Run` o cuando el trabajo lo exige. Por eso el mismo programa puede atender miles de esperas con muy pocos hilos.

## Un ejemplo: tres tareas a la vez

Tres «descargas» con una pausa cada una. Primero una tras otra y luego a la vez:

```csharp
using System.Diagnostics;

async Task Descargar(string nombre, int ms)
{
    await Task.Delay(ms);          // espera SIN bloquear el hilo
    Console.WriteLine($"termina {nombre}");
}

// Una tras otra
var reloj = Stopwatch.StartNew();
await Descargar("uno", 200);
await Descargar("dos", 50);
await Descargar("tres", 100);
bool secuencial = reloj.ElapsedMilliseconds >= 340;

Console.WriteLine("--- ahora a la vez ---");

// Las tres a la vez: se lanzan todas y luego se espera a todas
reloj.Restart();
await Task.WhenAll(Descargar("uno", 200), Descargar("dos", 50), Descargar("tres", 100));
bool simultaneo = reloj.ElapsedMilliseconds < 340;

Console.WriteLine($"una tras otra tarda 350 ms o más: {secuencial}");
Console.WriteLine($"a la vez tarda menos de 340 ms: {simultaneo}");
```

Salida:

```text
termina uno
termina dos
termina tres
--- ahora a la vez ---
termina dos
termina tres
termina uno
una tras otra tarda 350 ms o más: True
a la vez tarda menos de 340 ms: True
```

* Con `await` cada tarea **antes de lanzar la siguiente**, el tiempo se suma (200 + 50 + 100 = 350 ms).
* Con `Task.WhenAll`, se **lanzan las tres** (llamar a un método `async` ya empieza la tarea) y después se espera a todas: el tiempo es el de la **más lenta** (200 ms). Se ve en el orden en que terminan: `dos`, `tres`, `uno`, que no es el orden en que se lanzaron.

## Qué usar en cada caso

| Situación | Herramienta |
|---|---|
| **Esperar** algo externo (red, disco, temporizador) | `async`/`await` con `Task.Delay`, `HttpClient`, `File.ReadAllTextAsync`... |
| **Calcular** mucho (CPU) sin bloquear la interfaz | `Task.Run(() => ...)` |
| Repartir un **bucle de cálculo** entre núcleos | `Parallel.For` / `Parallel.ForEach` |
| Un hilo de verdad, con vida propia | `new Thread(...)` (raro: casi siempre basta `Task`) |

```csharp
int principal = Environment.CurrentManagedThreadId;
int dentroDeTask = await Task.Run(() => Environment.CurrentManagedThreadId);
Console.WriteLine($"Task.Run usa otro hilo: {principal != dentroDeTask}");

// Parallel.For reparte un bucle de CÁLCULO entre los núcleos
long suma = 0;
Parallel.For(1, 101, i => Interlocked.Add(ref suma, i));
Console.WriteLine($"suma en paralelo: {suma}");
```

Salida:

```text
Task.Run usa otro hilo: True
suma en paralelo: 5050
```

!!! warning "No bloquees una tarea"
    Dentro de un método `async`, **no uses** `Thread.Sleep`, `.Result` ni `.Wait()`: bloquean el hilo y pueden dejar el programa colgado (sobre todo en aplicaciones con ventana). Usa `await`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| `Thread.Sleep` dentro de código `async` | `await Task.Delay(...)` |
| Hacer `await` de cada tarea nada más lanzarla (queda secuencial) | Lanzarlas todas y esperar después con `Task.WhenAll` |
| Llamar a un método `async` y no esperar su `Task` | `await` siempre (o guardar el `Task`): si no, los errores se pierden |
| `async void` | Usar `async Task`; `async void` solo en manejadores de eventos |
| Usar `Task.Run` para esperar una respuesta de red | La espera ya es asíncrona; `Task.Run` es para cálculo |

## Para practicar

Los ejercicios [A3.1 y A3.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
