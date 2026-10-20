# A3.C Compartir datos con seguridad

## La condición de carrera

Cuando varias tareas **modifican el mismo dato** a la vez, pueden pisarse. `contador++` parece un solo paso, pero son tres: **leer** el valor, **sumarle 1** y **escribirlo**. Si dos tareas leen el mismo valor antes de que la otra escriba, una suma se pierde. Es una **condición de carrera**: el resultado depende de quién llegue antes, y cambia de una ejecución a otra, lo que hace que estos errores sean muy difíciles de encontrar.

```csharp
const int Tareas = 4;
const int Vueltas = 100_000;

// 1. Sin proteger: contador++ son TRES pasos (leer, sumar, escribir) y las tareas se pisan
int sinProteger = 0;
await Task.WhenAll(Enumerable.Range(0, Tareas).Select(_ => Task.Run(() =>
{
    for (int i = 0; i < Vueltas; i++)
    {
        sinProteger++;
    }
})));
Console.WriteLine($"sin proteger, ¿se perdió alguna suma? {sinProteger != Tareas * Vueltas}");

// 2. Con lock: solo una tarea a la vez dentro del bloque
int conLock = 0;
object candado = new();
await Task.WhenAll(Enumerable.Range(0, Tareas).Select(_ => Task.Run(() =>
{
    for (int i = 0; i < Vueltas; i++)
    {
        lock (candado)
        {
            conLock++;
        }
    }
})));
Console.WriteLine($"con lock: {conLock}");

// 3. Con Interlocked: una operación atómica, sin candado
int atomico = 0;
await Task.WhenAll(Enumerable.Range(0, Tareas).Select(_ => Task.Run(() =>
{
    for (int i = 0; i < Vueltas; i++)
    {
        Interlocked.Increment(ref atomico);
    }
})));
Console.WriteLine($"con Interlocked: {atomico}");
```

Salida:

```text
sin proteger, ¿se perdió alguna suma? True
con lock: 400000
con Interlocked: 400000
```

(Con la versión sin proteger, el resultado exacto cambia cada vez; solo es seguro que casi siempre se pierden sumas. Por eso el ejemplo imprime únicamente si se perdió alguna.)

Hay tres formas de arreglarlo:

| Herramienta | Qué hace |
|---|---|
| **`lock (objeto) { ... }`** | Solo una tarea a la vez puede estar dentro del bloque |
| **`Interlocked`** (`Increment`, `Add`, `Exchange`...) | Operaciones **atómicas** sobre un número: indivisibles y sin candado, más rápidas |
| **Colecciones concurrentes** (`ConcurrentDictionary`, `ConcurrentQueue`, `ConcurrentBag`) | Colecciones diseñadas para usarse desde varios hilos |

!!! warning "`lock` y `await` no se mezclan"
    Dentro de un bloque `lock` **no se puede** usar `await` (el compilador lo prohíbe, error CS1996). Cuando necesites un candado con esperas, usa un **`SemaphoreSlim`** con `await WaitAsync()` y libéralo en un `finally`:

```csharp
var semaforo = new SemaphoreSlim(1, 1);      // un "candado" que sí se puede esperar con await
int total = 0;

async Task Sumar(int cuanto)
{
    await semaforo.WaitAsync();
    try
    {
        int leido = total;
        await Task.Delay(5);      // con un lock normal, esto no compila
        total = leido + cuanto;
    }
    finally
    {
        semaforo.Release();
    }
}

await Task.WhenAll(Sumar(1), Sumar(2), Sumar(3));
Console.WriteLine($"total: {total}");
```

Salida:

```text
total: 6
```

## Mejor que compartir: pasarse mensajes

La mejor manera de no tener condiciones de carrera es **no compartir** datos modificables. En su lugar, una tarea **envía** valores a otra por un **canal** (`Channel<T>`): un **productor** escribe, un **consumidor** lee, y nadie toca el dato del otro.

```csharp
using System.Threading.Channels;

// Un canal con capacidad 2: si está lleno, el productor espera
var canal = Channel.CreateBounded<int>(2);

var productor = Task.Run(async () =>
{
    for (int n = 1; n <= 5; n++)
    {
        await canal.Writer.WriteAsync(n);
        Console.WriteLine($"envía {n}");
    }
    canal.Writer.Complete();      // avisa de que no habrá más
});

int suma = 0;
await foreach (int n in canal.Reader.ReadAllAsync())
{
    await Task.Delay(20);         // el consumidor es más lento
    suma += n;
}
await productor;
Console.WriteLine($"suma: {suma}");
```

Salida:

```text
envía 1
envía 2
envía 3
envía 4
envía 5
suma: 15
```

El canal tiene capacidad para 2 elementos: cuando está lleno, `WriteAsync` hace esperar al productor hasta que el consumidor saque alguno. Eso evita que un productor rápido llene la memoria (*contrapresión*). Cuando el productor termina llama a `Complete()` y el `await foreach` del consumidor acaba solo.

## Reglas para dormir tranquilo

* **Prefiere no compartir.** Datos inmutables ([A1.C](../a1/t-composicion.md)) y canales eliminan casi todos los problemas.
* Si compartes, **protege siempre el acceso completo**: tanto escribir como leer, con la misma herramienta.
* Mantén lo que hay **dentro** del `lock` lo más corto posible, sin esperas ni llamadas a código desconocido.
* Para un solo número, `Interlocked`. Para una colección, la versión `Concurrent...`.
* Si necesitas varios candados a la vez, tómalos **siempre en el mismo orden** para evitar un *interbloqueo* (cada tarea espera el candado que tiene la otra).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| `contador++` desde varias tareas | `Interlocked.Increment` o `lock` |
| Proteger la escritura pero no la lectura | Proteger todos los accesos |
| `await` dentro de `lock` | `SemaphoreSlim` |
| Una colección normal (`List`, `Dictionary`) modificada desde varias tareas | Una colección `Concurrent...` o un `lock` |
| Comprobar que «funciona» con una sola ejecución | Probarlo muchas veces: las carreras no siempre se ven |

## Para practicar

Los ejercicios [A3.5 y A3.6](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
