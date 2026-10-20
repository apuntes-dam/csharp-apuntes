# A1.B Colecciones sin bucles

Casi todo lo que haces con una lista con un `for` entra en unas pocas operaciones: **quedarse con** unos elementos, **transformarlos**, **reducirlos** a un valor, **ordenarlos**, **agruparlos** y **preguntar** si alguno o todos cumplen algo. Dar nombre a cada operación hace que el código diga **qué** quieres, no **cómo** recorrer la lista.

## Las operaciones en C#: LINQ

En C# esas operaciones se llaman **LINQ** (*Language Integrated Query*) y son métodos de extensión de `System.Linq` (ya incluido con `using` implícito en los proyectos nuevos) que funcionan sobre cualquier colección, array o secuencia (`IEnumerable<T>`):

| Qué quieres | Cómo se hace |
|---|---|
| Quedarse con los que cumplen | `Where(p => ...)` |
| Transformar cada elemento | `Select(p => ...)` |
| Reducir a un solo valor | `Sum()`, `Max()`, `Min()`, `Average()`, `Count(...)`, o `Aggregate(inicial, (acum, x) => ...)` |
| Ordenar con un criterio | `OrderBy(p => ...)` / `OrderByDescending(p => ...)` y `ThenBy` para desempatar (devuelven una secuencia nueva) |
| Agrupar o contar por clave | `GroupBy(p => ...)`, `ToDictionary(...)` |
| ¿Alguno / todos? | `Any(...)` / `All(...)` |
| Primero, o nada | `First(...)` (lanza error si no hay) / `FirstOrDefault(...)` |
| Sin repetidos · los primeros n | `Distinct()` · `Take(n)` |

## Un ejemplo completo

Una pastelería guarda sus pedidos (producto, cantidad y precio por unidad). Con las operaciones anteriores se responde a todo sin un solo bucle explícito para calcular:

```csharp
var pedidos = new List<Pedido>
{
    new("tarta", 2, 18),
    new("galleta", 12, 1),
    new("tarta", 1, 18),
    new("magdalena", 3, 2),
    new("galleta", 7, 1),
};

// 1. transformar: el importe de cada pedido
var importes = pedidos.Select(p => p.Cantidad * p.Precio).ToList();
Console.WriteLine($"importes: {string.Join(", ", importes)}");

// 2. reducir: el total
Console.WriteLine($"total: {importes.Sum()}");

// 3. filtrar y contar: pedidos de más de 10
Console.WriteLine($"pedidos de más de 10: {pedidos.Count(p => p.Cantidad * p.Precio > 10)}");

// 4. agrupar: unidades por producto
var porProducto = pedidos.GroupBy(p => p.Producto).ToDictionary(g => g.Key, g => g.Sum(p => p.Cantidad));
foreach (var (producto, unidades) in porProducto.OrderBy(x => x.Key, StringComparer.Ordinal))
{
    Console.WriteLine($"  {producto}: {unidades} unidades");
}

// 5. ordenar: el pedido más caro primero
var masCaro = pedidos.OrderByDescending(p => p.Cantidad * p.Precio).First();
Console.WriteLine($"el más caro: {masCaro.Producto} ({masCaro.Cantidad * masCaro.Precio})");

// 6. preguntar
Console.WriteLine($"¿alguno de magdalena? {pedidos.Any(p => p.Producto == "magdalena")}");
Console.WriteLine($"¿todos de más de 1 unidad? {pedidos.All(p => p.Cantidad > 1)}");

// 7. productos distintos, ordenados
Console.WriteLine($"productos: {string.Join(", ", pedidos.Select(p => p.Producto).Distinct().Order(StringComparer.Ordinal))}");

record Pedido(string Producto, int Cantidad, int Precio);
```

Salida:

```text
importes: 36, 12, 18, 6, 7
total: 79
pedidos de más de 10: 3
  galleta: 19 unidades
  magdalena: 3 unidades
  tarta: 3 unidades
el más caro: tarta (36)
¿alguno de magdalena? True
¿todos de más de 1 unidad? False
productos: galleta, magdalena, tarta
```

Cada bloque de la salida es una de las operaciones de la tabla. Compruébalas a mano: los importes son `2·18`, `12·1`, `1·18`, `3·2` y `7·1`; el total es `79`; y hay `19` galletas porque `12 + 7`.

## Lo que conviene saber

* **No modifican la colección original.** `Where`, `Select` u `OrderBy` devuelven una secuencia **nueva**; la lista de partida queda igual. (A diferencia de `List<T>.Sort()`, que sí ordena la propia lista.)
* **Son perezosas.** Hasta que recorres el resultado (con `foreach`, `ToList()`, `Sum()`...), no se calcula nada. Se explica en [A1.C](t-composicion.md).
* **Hay dos sintaxis.** La de métodos (`Where(...).Select(...)`) y la de consulta (`from ... where ... select`), que el compilador traduce a la primera. Dan el mismo resultado:

```csharp
int[] numeros = [5, 8, 12, 3, 20];

var conMetodos = numeros.Where(n => n > 4).OrderBy(n => n).Select(n => n * 10);
var conConsulta = from n in numeros
                  where n > 4
                  orderby n
                  select n * 10;

Console.WriteLine(string.Join(" ", conMetodos));
Console.WriteLine(string.Join(" ", conConsulta));
```

Salida:

```text
50 80 120 200
50 80 120 200
```

!!! warning "Un orden estable"
    `OrderBy` es **estable**: los elementos con el mismo criterio mantienen su orden relativo. Para desempatar de forma explícita, usa `ThenBy`. Y cuando ordenes texto, decide la comparación: sin indicarla, `OrderBy` usa la **cultura del equipo**, que puede colocar distinto las tildes o los signos; `StringComparer.Ordinal` compara por código de carácter y da el mismo resultado en cualquier máquina.

```csharp
var palabras = new[] { "pera", "uva", "kiwi", "fresa", "mora" };

// OrderBy es estable: las de la misma longitud mantienen su orden original
Console.WriteLine(string.Join(", ", palabras.OrderBy(p => p.Length)));

// ThenBy desempata por un segundo criterio
Console.WriteLine(string.Join(", ", palabras.OrderBy(p => p.Length).ThenBy(p => p, StringComparer.Ordinal)));
```

Salida:

```text
uva, pera, kiwi, mora, fresa
uva, kiwi, mora, pera, fresa
```

## Para practicar

Los ejercicios [A1.3 y A1.4](ejercicios.md) usan estas operaciones. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
