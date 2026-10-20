# A1.C Componer, esperar e inmutabilidad

## Componer funciones

Si una función devuelve algo que otra función necesita, se pueden **encadenar**. Y como las funciones son valores, se puede escribir una función que **construya** la cadena: `Componer(f, g)` devuelve una función nueva que aplica `f` y luego `g`. Con una lista de funciones, `Aggregate` las compone todas, que es una **tubería** (*pipeline*):

```csharp
Func<int, int> doble = n => n * 2;
Func<int, int> masUno = n => n + 1;
Func<int, int> cuadrado = n => n * n;

// Componer dos funciones: primero f, después g
Func<int, int> Componer(Func<int, int> f, Func<int, int> g) => x => g(f(x));

var dobleYMasUno = Componer(doble, masUno);
Console.WriteLine($"doble y +1 de 5 = {dobleYMasUno(5)}");

// Una tubería con cualquier número de funciones
var tuberia = new[] { doble, masUno, cuadrado }.Aggregate(Componer);
Console.WriteLine($"doble, +1 y cuadrado de 5 = {tuberia(5)}");

// El orden importa
var alReves = new[] { cuadrado, masUno, doble }.Aggregate(Componer);
Console.WriteLine($"cuadrado, +1 y doble de 5 = {alReves(5)}");
```

Salida:

```text
doble y +1 de 5 = 11
doble, +1 y cuadrado de 5 = 121
cuadrado, +1 y doble de 5 = 52
```

Fíjate en que el orden cambia el resultado: `((5·2)+1)²` es 121, pero `((5²)+1)·2` es 52. Con una tubería **vacía** hay que decidir qué devuelve: lo natural es la función identidad, `x => x` (es lo que pide el ejercicio A1.5).

## Calcular solo lo necesario

Las consultas LINQ son **perezosas**: `Select(...)` no calcula nada; solo **describe** el cálculo. Se ejecuta cuando alguien recorre el resultado, y **cada vez** que se recorre. Esto tiene dos consecuencias importantes que el programa siguiente cuenta con un contador de cálculos:

```csharp
int calculos = 0;

int Cuadrado(int n)
{
    calculos++;
    return n * n;
}

// Definir la consulta NO calcula nada
var consulta = Enumerable.Range(1, 5).Select(Cuadrado);
Console.WriteLine($"tras definirla: {calculos} cálculos");

// Cada vez que se recorre, se vuelve a calcular
Console.WriteLine($"suma: {consulta.Sum()}, cálculos: {calculos}");
Console.WriteLine($"máximo: {consulta.Max()}, cálculos: {calculos}");

// ToList() calcula una vez y guarda el resultado
calculos = 0;
var guardada = Enumerable.Range(1, 5).Select(Cuadrado).ToList();
Console.WriteLine($"suma: {guardada.Sum()}, máximo: {guardada.Max()}, cálculos: {calculos}");

// Una secuencia infinita: solo se calcula lo que se pide
IEnumerable<int> Naturales()
{
    int n = 1;
    while (true)
    {
        yield return n++;
    }
}
Console.WriteLine($"primeros pares: {string.Join(", ", Naturales().Where(n => n % 2 == 0).Take(4))}");
```

Salida:

```text
tras definirla: 0 cálculos
suma: 55, cálculos: 5
máximo: 25, cálculos: 10
suma: 55, máximo: 25, cálculos: 5
primeros pares: 2, 4, 6, 8
```

* Definir la consulta no hace ningún cálculo (`0 cálculos`).
* Si la recorres dos veces (`Sum` y `Max`), se calcula **dos veces** (5 + 5 = 10). Si vas a usar el resultado varias veces, guárdalo con **`ToList()`** (o `ToArray()`) y se calcula una sola vez.
* La perezosa permite trabajar con **secuencias infinitas**. Un método con **`yield return`** produce los valores de uno en uno, solo cuando se piden: `Naturales()` no acaba nunca, pero `Take(4)` pide únicamente cuatro pares y se detiene.

!!! warning "Efectos secundarios en una consulta"
    Si dentro de un `Select` o un `Where` haces algo más que calcular (escribir en pantalla, cambiar una variable), ese efecto ocurrirá cuando se recorra y tantas veces como se recorra, no donde escribiste la consulta. Mantén las funciones de una consulta **puras**: que solo calculen un resultado a partir de sus parámetros.

## No modificar los datos

Una programación funcional cómoda evita **cambiar** los datos: en vez de modificar un valor, se crea otro. Así nadie recibe una sorpresa porque otra parte del programa haya tocado «su» lista. En C# hay varias herramientas:

| Herramienta | Qué da |
|---|---|
| `record` con propiedades `init` | Objetos cuyos valores no cambian tras crearlos; `with` crea una copia con un valor distinto |
| `readonly` y `init` | Campos y propiedades que solo se asignan al crear el objeto |
| `IReadOnlyList<T>` | Una **vista** de solo lectura (no impide que el dueño de la lista la cambie) |
| `ImmutableList<T>` (`System.Collections.Immutable`) | Una lista que **nunca cambia**: cada `Add` devuelve una lista nueva |
| `ToList()` | Una copia, para que los cambios posteriores del original no te afecten |

```csharp
using System.Collections.Immutable;

// record: los valores no cambian; "with" crea una copia con algo distinto
var original = new Persona("Ana", 20);
var mayor = original with { Edad = 21 };
Console.WriteLine($"{original} -> {mayor}");

// una lista de solo lectura no deja añadir, pero otra referencia a la misma lista sí podría
var datos = new List<int> { 1, 2, 3 };
IReadOnlyList<int> vista = datos;
datos.Add(4);
Console.WriteLine($"la vista de solo lectura ve el cambio: {vista.Count} elementos");

// una lista inmutable de verdad: cada cambio devuelve una lista nueva
var inmutable = ImmutableList.Create(1, 2, 3);
var masUno = inmutable.Add(4);
Console.WriteLine($"original: {inmutable.Count} elementos, nueva: {masUno.Count} elementos");

// una copia defensiva evita que cambien tu lista desde fuera
var copia = datos.ToList();
datos.Clear();
Console.WriteLine($"la copia conserva {copia.Count} elementos");

record Persona(string Nombre, int Edad);
```

Salida:

```text
Persona { Nombre = Ana, Edad = 20 } -> Persona { Nombre = Ana, Edad = 21 }
la vista de solo lectura ve el cambio: 4 elementos
original: 3 elementos, nueva: 4 elementos
la copia conserva 4 elementos
```

Observa la diferencia entre `IReadOnlyList` e `ImmutableList`: la primera solo **prohíbe** cambiar *a través de ella*; la segunda **no puede cambiar nunca**.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Recorrer varias veces una consulta cara y repetir el cálculo | Guardar el resultado con `ToList()` |
| Hacer cosas con efectos dentro de un `Select` | Mantener las funciones puras |
| Creer que `IReadOnlyList` hace la lista inmutable | Es solo una vista; usa `ImmutableList` o una copia |
| Modificar una lista mientras la recorres con `foreach` | Crear una nueva lista con el resultado (lanza `InvalidOperationException`) |
| Componer funciones en el orden equivocado | Comprobar con un caso pequeño a mano |

## Para practicar

Los ejercicios [A1.5 y A1.6](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
