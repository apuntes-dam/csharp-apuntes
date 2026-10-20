# 7.A Consola: entrada y salida estándar

Cuando un programa se ejecuta en una terminal, el sistema le da **tres canales** de texto, llamados **flujos estándar**:

| Flujo | Nombre | Para qué sirve | Por defecto | En C# |
|---|---|---|---|---|
| `stdin` | Entrada estándar | Lo que el programa **recibe** | El teclado | `Console.In` · `Console.ReadLine()` |
| `stdout` | Salida estándar | Los **resultados** normales | La pantalla | `Console.Out` · `Console.WriteLine()` |
| `stderr` | Salida de error | Los **avisos y errores** | La pantalla | `Console.Error` · `Console.Error.WriteLine()` |

Los dos de salida van a la pantalla, pero son **canales distintos**, y esa diferencia permite separar los resultados de los mensajes de error.

## Mostrar datos con formato

```csharp
using System.Globalization;

var equipos = new List<Equipo> { new("Águilas", 45, 12), new("Lobos", 41, 5), new("Tigres", 38, -2) };

Console.WriteLine($"{"Equipo",-12}{"Pts",5}{"Dif",6}");
Console.WriteLine(new string('-', 23));
int total = 0;
foreach (Equipo e in equipos)
{
    Console.WriteLine($"{e.Nombre,-12}{e.Puntos,5}{e.Diferencia,6:+0;-0;0}");
    total += e.Puntos;
}
double media = (double)total / equipos.Count;
Console.WriteLine($"Media de puntos: {media.ToString("F2", CultureInfo.InvariantCulture)}");

Console.Error.WriteLine("Aviso: datos de ejemplo, no oficiales");
Console.WriteLine("Fin del informe");

record Equipo(string Nombre, int Puntos, int Diferencia);
```

Salida:

```text
Equipo        Pts   Dif
-----------------------
Águilas        45   +12
Lobos          41    +5
Tigres         38    -2
Media de puntos: 41.33
Fin del informe
```

Qué conviene aprender de este ejemplo:

* **Columnas alineadas.** En una cadena interpolada, `{valor,-12}` reserva **12 posiciones** y alinea a la **izquierda**; `{valor,5}` reserva 5 y alinea a la **derecha**. Sin eso, las columnas quedan descuadradas.
* **Formato detrás de los dos puntos.** `{x:F2}` escribe dos decimales; `{x:+0;-0;0}` es un formato con **tres secciones** (positivo; negativo; cero) y por eso el `+` aparece solo en los positivos.
* **El aviso va por otro canal.** Verás que la línea «Aviso: datos de ejemplo, no oficiales» **no** aparece en el recuadro de arriba: se escribe en `Console.Error`, no en la salida estándar. En una terminal sí se vería.

| Necesito... | En C# |
|---|---|
| Alinear a la izquierda | `{nombre,-12}` o `nombre.PadRight(12)` |
| Alinear a la derecha | `{puntos,5}` o `puntos.ToString().PadLeft(5)` |
| Decimales fijos | `{media:F2}` (o `{media:0.00}`) |
| Repetir un texto | `new string('-', 23)` (un solo carácter repetido) |
| Mostrar sin salto de línea | `Console.Write("texto")` |
| Con signo | `{d:+0;-0;0}` |

**La cultura del equipo.** Los números se escriben según la configuración regional del equipo. Mira lo que pasa en un equipo configurado en español:

```csharp
double media = 124.0 / 3;
Console.WriteLine($"con la cultura del equipo: {media:F2}");
Console.WriteLine($"con la cultura invariable: {media.ToString("F2", System.Globalization.CultureInfo.InvariantCulture)}");
```

Salida:

```text
con la cultura del equipo: 41,33
con la cultura invariable: 41.33
```

En un Windows en español el separador decimal es la **coma**. Si necesitas siempre el punto (para un archivo que lee otro programa, por ejemplo), pasa `CultureInfo.InvariantCulture`, como en el ejemplo anterior. Al revés, para mostrar al usuario un número en su formato, deja la cultura por defecto.

## Salida estándar y salida de error

Separarlas sirve para que un programa pueda **guardar los resultados en un archivo sin mezclarlos con los errores**. En una terminal, el operador `>` redirige la salida normal y `2>` la de error:

```bash
dotnet run cons1.cs > resultados.txt 2> errores.txt
```

Con el ejemplo anterior, `resultados.txt` contendría la tabla y la media y `errores.txt` solo el aviso. Para **dar entrada desde un archivo** en lugar del teclado se usa `<` (`dotnet run programa.cs < entrada.txt`), y para encadenar programas, la **tubería** `|`.

## Leer datos

Se lee con **`Console.ReadLine()`**, que devuelve un `string?`: es **`null`** cuando ya no queda entrada (*fin de archivo*), por eso hay que contemplarlo. A diferencia de Java o Python, **no lanza ninguna excepción** al llegar al final. Para escribir sin saltar de línea, `Console.Write`.

```csharp
Console.WriteLine("Escribe líneas (una vacía para terminar):");
int lineas = 0;
int palabras = 0;
string? linea;
while ((linea = Console.ReadLine()) is not null && linea != "")
{
    lineas++;
    palabras += linea.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
}
Console.WriteLine($"Líneas: {lineas}");
Console.WriteLine($"Palabras: {palabras}");
```

Salida:

```text
Escribe líneas (una vacía para terminar):
Líneas: 2
Palabras: 6
```

Este programa lee líneas **hasta encontrar una vacía** (o hasta que se acabe la entrada). Aquí se le han dado `hola mundo`, `esto es una prueba` y una línea vacía. Fíjate en la condición del `while`: asigna la línea leída, comprueba que no sea `null` y que no esté vacía, todo a la vez.

Una entrada del usuario es **siempre texto**: si necesitas un número, conviértelo con `int.TryParse` y **prevé que falle** (mira el patrón de «pedir hasta que sea válido» en [2.3 Excepciones](../u02/02-excepciones.md)).

!!! warning "`Console.ReadKey()` y la entrada redirigida"
    `Console.ReadKey()` espera **una tecla** de la consola de verdad. Si la entrada viene de un archivo o de una tubería (`< entrada.txt`), lanza una `InvalidOperationException`. Para programas que deban funcionar con entrada redirigida, usa `ReadLine`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Columnas descuadradas | Fijar un ancho para cada columna (`{x,-12}`, `{x,5}`) |
| Mezclar resultados y errores en la misma salida | Escribir los avisos con `Console.Error` |
| Dar por hecho que siempre hay una línea más que leer | Comprobar `null` en `ReadLine()` |
| Usar el número leído sin convertirlo ni validarlo | `int.TryParse` y volver a pedir |
| Decimales con coma en unas máquinas y con punto en otras | Fijar `CultureInfo.InvariantCulture` al formatear para otros programas |
| `ReadKey` con la entrada redirigida | Usar `ReadLine` |

## Para practicar

Haz los ejercicios de [U7.1 · Consola](consola.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
