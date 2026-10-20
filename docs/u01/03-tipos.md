# 1.3 Tipos de datos

C# es de **tipado estático**: cada variable tiene un tipo que se conoce al compilar. Con `var` casi nunca hay que escribirlo, pero sigue estando ahí.

## Tipos básicos

| Tipo | Ejemplo | Nota |
|---|---|---|
| `int` | `42` | entero de 32 bits |
| `long` | `42L` | entero de 64 bits |
| `byte` | `255` | entero de 0 a 255 |
| `double` | `3.14` | decimal de 64 bits (el habitual) |
| `float` | `3.14f` | decimal de 32 bits |
| `decimal` | `19.99m` | decimal **exacto**, pensado para **dinero** |
| `bool` | `true` | solo `true` o `false` |
| `char` | `'A'` | un carácter |
| `string` | `"hola"` | texto, inmutable |

!!! warning "Dinero con `decimal`, no con `double`"
    `double` guarda los decimales en binario y no es exacto: `0.1 + 0.2` da `0,30000000000000004` (con coma en un equipo en español). Para importes, usa `decimal`: `0.1m + 0.2m` es exactamente `0,3`.

## Tipos por valor y por referencia

Los números, `bool` y `char` son **tipos por valor**: la variable contiene el dato. `string`, las listas y los objetos son **tipos por referencia**: la variable apunta al dato. Cada tipo básico tiene también un nombre de .NET (`int` es `System.Int32`, `double` es `System.Double`...); son lo mismo.

## Cadenas

```csharp
string s = "C#";
Console.WriteLine(s.Length);                 // 2
Console.WriteLine("hola".ToUpper());         // HOLA
Console.WriteLine($"Hola, {s} y {s.Length}");  // interpolación
Console.WriteLine("a,b,c".Split(',').Length);  // 3
Console.WriteLine(" texto ".Trim());         // texto
```

## Colecciones

```csharp
var lista = new List<int> { 3, 1, 2 };          // lista que crece
var conjunto = new HashSet<int> { 1, 1, 2 };    // sin repetidos
var mapa = new Dictionary<string, int> { ["a"] = 1, ["b"] = 2 };   // clave -> valor
int[] fijo = [10, 20, 30];                      // array de tamaño fijo (C# 12)

lista.Add(4);
Console.WriteLine(string.Join(", ", lista));     // 3, 1, 2, 4
Console.WriteLine(conjunto.Count);               // 2
Console.WriteLine(mapa["a"]);                    // 1
Console.WriteLine(fijo.Length);                  // 3
```

Las colecciones genéricas (`List<T>`, `HashSet<T>`, `Dictionary<K, V>`) están en `System.Collections.Generic`. El tipo entre `< >` indica qué guardan.

## Conversiones

```csharp
Console.WriteLine(int.Parse("42"));              // 42
Console.WriteLine(int.TryParse("abc", out int n));   // False (no lanza error)
Console.WriteLine(Convert.ToInt32("42"));        // 42
Console.WriteLine(42.ToString());                // 42
Console.WriteLine((int)3.9);                     // 3  (trunca)
Console.WriteLine((double)3);                    // 3
long grande = 5;                                 // conversión automática: int cabe en long
```

- **Ampliar** (de un tipo pequeño a uno grande: `int` a `long` o `double`) es automático.
- **Reducir** (de `double` a `int`) pide un **cast** explícito `(int)`, porque se pierde información.
- `int.Parse` lanza una excepción si el texto no es un número; **`int.TryParse`** devuelve `false` y no falla, y deja el resultado en la variable marcada con `out`.

!!! note "Los decimales dependen del idioma"
    `double.Parse("3.5")` y la forma en que se escribe un `double` dependen de la **configuración regional**: en un equipo en español se espera y se escribe `3,5`. Para fijar el punto siempre, usa `CultureInfo.InvariantCulture` (de `System.Globalization`): `double.Parse("3.5", CultureInfo.InvariantCulture)`.

## `object` y `dynamic`

`object` es el tipo del que derivan todos los demás: cualquier valor cabe en una variable `object`. `dynamic` desactiva la comprobación de tipos al compilar y no se recomienda al empezar.
