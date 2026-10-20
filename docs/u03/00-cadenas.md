# 3.0 Cadenas de texto

Una **cadena** (`string`) es una secuencia de caracteres. Es uno de los tipos más usados y tiene una propiedad importante: es **inmutable**, es decir, ningún método la cambia: todos **devuelven una cadena nueva**.

## Recorrer y buscar

```csharp
string palabra = "murciélago";

Console.WriteLine($"longitud: {palabra.Length}");
Console.WriteLine($"primera: {palabra[0]}, última: {palabra[^1]}");
Console.WriteLine($"posición de la c: {palabra.IndexOf('c')}");
Console.WriteLine($"posición de la z: {palabra.IndexOf('z')}");
Console.WriteLine($"¿contiene «cié»? {palabra.Contains("cié")}");
Console.WriteLine($"¿empieza por «mur»? {palabra.StartsWith("mur")}");

int vocales = 0;
foreach (char letra in palabra)
{
    if ("aeiouáéíóú".Contains(letra))
    {
        vocales++;
    }
}
Console.WriteLine($"vocales: {vocales}");
```

Salida:

```text
longitud: 10
primera: m, última: o
posición de la c: 3
posición de la z: -1
¿contiene «cié»? True
¿empieza por «mur»? True
vocales: 5
```

* `texto[i]` da el carácter de la posición `i` (se empieza a contar en **0**); `texto[^1]` es el último, `texto[^2]` el penúltimo...
* `IndexOf` devuelve la posición de la primera coincidencia o **-1** si no está.
* `foreach` recorre un texto carácter a carácter.

## Transformar

Como la cadena no cambia, el resultado de cada método hay que **guardarlo**:

```csharp
string s = "  Hola, Mundo  ";

s.ToUpper();                       // no hace nada útil: se descarta el resultado
Console.WriteLine($"[{s}]");        // sigue igual

string limpio = s.Trim();
Console.WriteLine($"[{limpio}]");
Console.WriteLine(limpio.ToUpper());
Console.WriteLine(limpio.ToLower());
Console.WriteLine(limpio.Replace("o", "0"));
Console.WriteLine(limpio.Substring(0, 4));       // desde la posición 0, 4 caracteres
Console.WriteLine(limpio[6..]);                   // desde la posición 6 hasta el final
Console.WriteLine(limpio[..4]);                   // desde el principio hasta la 4 (sin incluirla)
Console.WriteLine(limpio[^5..]);                  // los 5 últimos

string[] partes = "manzana,pera,uva".Split(',');
Console.WriteLine(partes.Length);
Console.WriteLine(string.Join(" | ", partes));
Console.WriteLine("7".PadLeft(3, '0'));
```

Salida:

```text
[  Hola, Mundo  ]
[Hola, Mundo]
HOLA, MUNDO
hola, mundo
H0la, Mund0
Hola
Mundo
Hola
Mundo
3
manzana | pera | uva
007
```

Los **rangos** (`[inicio..fin]`) funcionan como el *slicing* de Python: incluyen el inicio y **no** incluyen el final.

!!! tip "Construir un texto largo: `StringBuilder`"
    Como cada `+=` crea una cadena nueva, concatenar miles de veces en un bucle es lento. En ese caso se usa `StringBuilder`:

```csharp
using System.Text;

var sb = new StringBuilder();
for (int i = 1; i <= 5; i++)
{
    sb.Append(i).Append(' ');
}
Console.WriteLine(sb.ToString().Trim());
```

Salida:

```text
1 2 3 4 5
```

## Los métodos más útiles

| Necesitas... | Usa... |
|---|---|
| Longitud | `texto.Length` |
| Buscar una posición | `IndexOf("x")`, `LastIndexOf("x")` (devuelven -1 si no está) |
| Saber si contiene, empieza o acaba | `Contains`, `StartsWith`, `EndsWith` |
| Mayúsculas y minúsculas | `ToUpper()`, `ToLower()` |
| Quitar espacios de los extremos | `Trim()`, `TrimStart()`, `TrimEnd()` |
| Reemplazar | `Replace("viejo", "nuevo")` |
| Un trozo | `Substring(inicio, longitud)` o `texto[a..b]` |
| Trocear y unir | `Split(',')`, `string.Join(", ", partes)` |
| Rellenar | `PadLeft(n, '0')`, `PadRight(n)` |
| ¿Vacío? | `string.IsNullOrEmpty(t)`, `string.IsNullOrWhiteSpace(t)` |
| Comparar sin mirar mayúsculas | `string.Equals(a, b, StringComparison.OrdinalIgnoreCase)` |

## Unicode

Un `char` de C# es una unidad de **UTF-16**, y algunos caracteres (los emojis, por ejemplo) ocupan **dos**. Por eso `Length` puede no coincidir con lo que *ve* una persona:

```csharp
using System.Globalization;

string s = "Hola 😀";
Console.WriteLine($"Length: {s.Length}");
Console.WriteLine($"caracteres que se ven: {new StringInfo(s).LengthInTextElements}");
Console.WriteLine($"Length de «é»: {"é".Length}");
```

Salida:

```text
Length: 7
caracteres que se ven: 6
Length de «é»: 1
```

Para casi todo el texto en español (con tildes y la ñ) `Length` sirve. Si tienes que contar lo que ve el usuario en textos con emojis, usa `StringInfo`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a `ToUpper()` y no guardar el resultado | Recuerda: la cadena es inmutable; escribe `s = s.ToUpper();` |
| `Substring` con una longitud mayor que el texto | Lanza `ArgumentOutOfRangeException`: comprueba `Length` antes |
| Suponer que `IndexOf` devuelve un valor válido | Devuelve **-1** si no encuentra; compruébalo |
| Comparar sin tener en cuenta las mayúsculas | `string.Equals(a, b, StringComparison.OrdinalIgnoreCase)` |
| Un `+=` dentro de un bucle largo | Usa `StringBuilder` |

## Para practicar

Haz los [ejercicios 3.0 de cadenas](cadenas.md).
