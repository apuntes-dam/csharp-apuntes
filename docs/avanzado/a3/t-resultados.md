# A3.B Resultados, errores y esperas

## Resultados futuros: `Task<T>`

Un `Task<int>` es una **promesa**: «dentro de un rato tendrás un `int`, o un error». Con `await` se obtiene el resultado, y si la tarea falló, `await` **vuelve a lanzar** la excepción en el sitio donde esperas, de modo que se captura con un `try`/`catch` normal. No hay un mecanismo de errores aparte.

## Un ejemplo: tres consultas y un error

```csharp
async Task<int> Consulta(string nombre, int ms, bool falla = false)
{
    await Task.Delay(ms);
    if (falla)
    {
        throw new InvalidOperationException($"falló {nombre}");
    }
    Console.WriteLine($"{nombre} responde");
    return ms;
}

// 1. Una tarea con resultado
int valor = await Consulta("a", 30);
Console.WriteLine($"resultado: {valor}");

// 2. Varias a la vez, todas correctas: WhenAll devuelve un array con los resultados, EN EL ORDEN en que las pasaste
int[] todos = await Task.WhenAll(Consulta("b", 60), Consulta("c", 20));
Console.WriteLine($"resultados: {string.Join(", ", todos)}");

// 3. Si una falla, await lanza la excepción de la primera que falló
var tareas = new[] { Consulta("d", 30), Consulta("e", 10, falla: true), Consulta("f", 50, falla: true) };
try
{
    await Task.WhenAll(tareas);
}
catch (InvalidOperationException e)
{
    Console.WriteLine($"error recogido: {e.Message}");
}
// Pero se conservan TODAS: cada tarea sabe cómo terminó
foreach (var t in tareas)
{
    Console.WriteLine($"  tarea {(t.IsFaulted ? "con error" : "correcta")}");
}
```

Salida:

```text
a responde
resultado: 30
c responde
b responde
resultados: 60, 20
d responde
error recogido: falló e
  tarea correcta
  tarea con error
  tarea con error
```

Tres cosas que conviene saber:

* **El orden de `WhenAll`** es el de la lista que le pasas, no el de llegada: en `b, c` el resultado es `60, 20` aunque `c` termine antes.
* Si **varias fallan**, `await Task.WhenAll(...)` lanza **solo la primera** excepción. Para ver todas, mira cada tarea (`t.IsFaulted`, `t.Exception`) como hace el bucle final. `t.Exception` es una `AggregateException` que agrupa las excepciones de la tarea.
* Una tarea que falla **no detiene a las demás**: las otras dos siguen y terminan.

## Límites de tiempo y cancelación

Esperar sin límite es peligroso: un servidor que no responde deja el programa colgado. Hay tres herramientas:

| Quiero... | Herramienta |
|---|---|
| Dejar de **esperar** pasado un tiempo | `tarea.WaitAsync(TimeSpan)` → lanza `TimeoutException` |
| Pedir a la tarea que **pare** | `CancellationToken` (de un `CancellationTokenSource`) → la tarea lanza `OperationCanceledException` |
| **Repetir** una operación que falla a veces | Un bucle con `try`/`catch` (o la biblioteca *Polly* en proyectos reales) |

```csharp
async Task<int> Consulta(int ms, CancellationToken token = default)
{
    await Task.Delay(ms, token);
    return 42;
}

// 1. Límite de tiempo: WaitAsync lanza TimeoutException
try
{
    await Consulta(300).WaitAsync(TimeSpan.FromMilliseconds(100));
}
catch (TimeoutException)
{
    Console.WriteLine("tiempo agotado");
}

// 2. Cancelación: el token avisa a la tarea de que debe parar
using var cancelacion = new CancellationTokenSource(TimeSpan.FromMilliseconds(100));
try
{
    await Consulta(300, cancelacion.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("tarea cancelada");
}

// 3. Reintentos: repetir una operación que falla a veces
int llamadas = 0;
async Task<string> Inestable()
{
    await Task.Delay(5);
    if (++llamadas < 3)
    {
        throw new InvalidOperationException("fallo temporal");
    }
    return "listo";
}

for (int intento = 1; intento <= 3; intento++)
{
    try
    {
        Console.WriteLine($"intento {intento}: {await Inestable()}");
        break;
    }
    catch (InvalidOperationException)
    {
        Console.WriteLine($"intento {intento} falló");
    }
}
```

Salida:

```text
tiempo agotado
tarea cancelada
intento 1 falló
intento 2 falló
intento 3: listo
```

La diferencia entre `WaitAsync` y la cancelación es importante: **`WaitAsync` solo deja de esperar**; la tarea sigue ejecutándose por detrás. **Cancelar** es pedirle a la tarea que pare, y la tarea tiene que cooperar comprobando el token (como hace `Task.Delay(ms, token)`). Para que una tarea sea cancelable, acepta un `CancellationToken` y pásalo a todas las operaciones que haga.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Esperar sin límite una respuesta externa | `WaitAsync` o un `CancellationTokenSource` con tiempo |
| Pensar que `WaitAsync` cancela la tarea | Solo deja de esperarla; para pararla, usa un token |
| Capturar solo una excepción de varias tareas | Revisar cada tarea (`IsFaulted`, `Exception`) |
| Reintentar sin límite | Fijar un máximo de intentos (y, en la vida real, esperar un poco entre ellos) |
| `.Result` o `.Wait()` en vez de `await` | Siempre `await`; bloquean y envuelven el error en `AggregateException` |

## Para practicar

Los ejercicios [A3.3 y A3.4](ejercicios.md) usan límites de tiempo y reintentos. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
