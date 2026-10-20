# 3.1 Listas y tuplas

Una **lista** guarda varios valores **en orden** y permite añadir, quitar y consultar por posición. En C# hay dos formas de «lista»: el **array** (`int[]`, tamaño fijo) y la **lista** `List<T>` (crece y se encoge). Casi siempre se usa `List<T>`.

## Operaciones básicas

```csharp
var numeros = new List<int> { 5, 3, 8 };

numeros.Add(1);                 // al final
numeros.Insert(0, 9);           // en una posición
Console.WriteLine(string.Join(", ", numeros));

numeros.Remove(3);              // quita el valor 3
numeros.RemoveAt(0);            // quita la posición 0
Console.WriteLine(string.Join(", ", numeros));

Console.WriteLine($"tamaño: {numeros.Count}");
Console.WriteLine($"¿contiene 8? {numeros.Contains(8)}");
Console.WriteLine($"posición del 8: {numeros.IndexOf(8)}");
Console.WriteLine($"primero: {numeros[0]}, último: {numeros[^1]}");

numeros.Sort();
Console.WriteLine("ordenada: " + string.Join(", ", numeros));
numeros.Reverse();
Console.WriteLine("al revés: " + string.Join(", ", numeros));

int[] fijo = [10, 20, 30];       // array: el tamaño no cambia
fijo[1] = 99;
Console.WriteLine(string.Join(", ", fijo));
```

Salida:

```text
9, 5, 3, 8, 1
5, 8, 1
tamaño: 3
¿contiene 8? True
posición del 8: 1
primero: 5, último: 1
ordenada: 1, 5, 8
al revés: 8, 5, 1
10, 99, 30
```

* La lista se crea con `new List<int> { ... }`; el tipo entre `< >` dice qué guarda.
* `Count` es el número de elementos de una lista y `Length` el de un array.
* `Remove(valor)` borra el **primer** elemento igual; `RemoveAt(i)` borra por posición.
* Las posiciones empiezan en **0**.

## Recorrer, copiar y listas anidadas

```csharp
var original = new List<string> { "Ana", "Luis", "Eva" };

foreach (string nombre in original)
{
    Console.WriteLine(nombre);
}

var misma = original;                     // NO es una copia: es la misma lista
var copia = new List<string>(original);   // esto sí es una copia
misma.Add("Pepe");
Console.WriteLine($"original: {string.Join(", ", original)}");
Console.WriteLine($"copia:    {string.Join(", ", copia)}");

List<string> otraCopia = [.. original];   // otra forma de copiar (operador de propagación)
Console.WriteLine(otraCopia.Count);

// una matriz es una lista de listas
var matriz = new List<List<int>>
{
    new() { 1, 2, 3 },
    new() { 4, 5, 6 },
};
foreach (var fila in matriz)
{
    Console.WriteLine(string.Join(" ", fila));
}
Console.WriteLine($"fila 1, columna 2: {matriz[1][2]}");

int[,] tabla = { { 1, 2 }, { 3, 4 } };    // matriz rectangular
Console.WriteLine($"tabla[1, 0] = {tabla[1, 0]}");
```

Salida:

```text
Ana
Luis
Eva
original: Ana, Luis, Eva, Pepe
copia:    Ana, Luis, Eva
4
1 2 3
4 5 6
fila 1, columna 2: 6
tabla[1, 0] = 3
```

Asignar una lista a otra variable (`var misma = original`) **no copia**: las dos variables apuntan a la misma lista. Para copiar de verdad, `new List<T>(original)` o `[.. original]`. Esa copia es **superficial**: si los elementos son listas, comparten las listas interiores.

## Tuplas y transformar listas

Una **tupla** agrupa varios valores sin crear una clase. En C# se escribe entre paréntesis y los elementos pueden tener nombre:

```csharp
(string nombre, int edad) persona = ("Ana", 25);
Console.WriteLine($"{persona.nombre} tiene {persona.edad}");

var (n, e) = persona;                       // se puede «desempaquetar»
Console.WriteLine($"{n}, {e}");

(int minimo, int maximo) MinMax(List<int> lista) => (lista.Min(), lista.Max());
var (menor, mayor) = MinMax(new List<int> { 4, 9, 1, 7 });
Console.WriteLine($"menor {menor}, mayor {mayor}");
```

Salida:

```text
Ana tiene 25
Ana, 25
menor 1, mayor 9
```

Para **transformar listas** se usa **LINQ** (métodos como `Where`, `Select`, `OrderBy`, `Sum`...), que ya viene incluido:

```csharp
var numeros = new List<int> { 5, 2, 8, 1, 9, 4 };

var pares = numeros.Where(n => n % 2 == 0).ToList();           // filtrar
var cuadrados = numeros.Select(n => n * n).ToList();            // transformar
var ordenados = numeros.OrderByDescending(n => n).ToList();     // ordenar

Console.WriteLine("pares: " + string.Join(", ", pares));
Console.WriteLine("cuadrados: " + string.Join(", ", cuadrados));
Console.WriteLine("de mayor a menor: " + string.Join(", ", ordenados));
Console.WriteLine($"suma: {numeros.Sum()}, media: {numeros.Average()}, máximo: {numeros.Max()}");
Console.WriteLine($"¿alguno mayor que 8? {numeros.Any(n => n > 8)}");
Console.WriteLine($"¿todos positivos? {numeros.All(n => n > 0)}");
```

Salida:

```text
pares: 2, 8, 4
cuadrados: 25, 4, 64, 1, 81, 16
de mayor a menor: 9, 8, 5, 4, 2, 1
suma: 29, media: 4,833333333333333, máximo: 9
¿alguno mayor que 8? True
¿todos positivos? True
```

`n => n % 2 == 0` es una **función lambda**: «dado `n`, devuelve si es par». Los métodos de LINQ **no modifican** la lista original: devuelven una secuencia nueva (con `ToList()` se convierte en lista).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Pedir una posición que no existe | Lanza `ArgumentOutOfRangeException`; usa `Count - 1` como última |
| Modificar la lista mientras la recorres con `foreach` | Lanza `InvalidOperationException`; recorre una copia o usa `RemoveAll` |
| Creer que `var b = a` copia la lista | Es la misma lista; copia con `new List<T>(a)` |
| Confundir `Count` y `Length` | `Count` en listas, `Length` en arrays y textos |
| Olvidar `ToList()` y esperar una lista | El resultado de LINQ es una secuencia; conviértela |

## Para practicar

Haz los [ejercicios 3.1 de listas y tuplas](listas.md).
