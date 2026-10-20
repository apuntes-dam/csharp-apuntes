# A2.B Restricciones y variación

## Restringir el tipo

Dentro de una función genérica, `T` es «cualquier tipo», y de «cualquier tipo» no se sabe nada: no se puede compararlo con `<`, ni sumarlo, ni crearlo con `new T()`. Una **restricción** (`where`) dice qué tiene que cumplir `T`, y a cambio permite usar esas capacidades:

| Restricción | Significa | Permite |
|---|---|---|
| `where T : IComparable<T>` | `T` se puede comparar con otro `T` | `a.CompareTo(b)` |
| `where T : class` / `struct` | `T` es un tipo de referencia / de valor | Comparar con `null` / usar `T?` como `Nullable` |
| `where T : new()` | `T` tiene constructor sin parámetros | `new T()` |
| `where T : notnull` | `T` no admite `null` | Usarlo como clave de un diccionario |
| `where T : MiInterfaz` | `T` cumple una interfaz (o hereda de una clase) | Llamar a sus miembros |
| `where T : INumber<T>` | `T` es un número (C# 11) | Operadores `+`, `*`... y `T.Zero` |

```csharp
Console.WriteLine(Mayor(3, 9));
Console.WriteLine(Mayor("pera", "casa"));
Console.WriteLine(Mayor(2.5, 1.5));

// Aritmética genérica (C# 11): cualquier tipo numérico
Console.WriteLine(SumaTodos([1, 2, 3]));
Console.WriteLine(SumaTodos([1.5, 2.5]));

// Mayor<T> necesita poder comparar
T Mayor<T>(T a, T b) where T : IComparable<T> => a.CompareTo(b) >= 0 ? a : b;

// INumber<T> da operadores como + y el valor T.Zero
T SumaTodos<T>(IEnumerable<T> elementos) where T : System.Numerics.INumber<T>
{
    T total = T.Zero;
    foreach (T x in elementos)
    {
        total += x;
    }
    return total;
}
```

Salida:

```text
9
pera
2,5
6
4
```

`Mayor` solo compila con tipos que sepan compararse (`int`, `string`, `double`...). Si lo llamas con un tipo propio que no implementa `IComparable<T>`, el compilador lo rechaza antes de ejecutar. La última, `INumber<T>`, es una novedad de C# 11 («matemática genérica») y no tiene equivalente directo en Java, Kotlin ni Dart: permite escribir **una sola** `SumaTodos` para `int`, `double`, `decimal`...

## ¿Una lista de enteros es una lista de números?

Parece que si `string` es un `object`, entonces una `List<string>` debería poder usarse como `List<object>`. **No puede**, y es una protección: si lo permitiera, podrías añadir un `42` a la lista de textos a través de la referencia `List<object>`.

La **variación** dice cuándo sí es seguro:

| Tipo | Variación | Qué permite |
|---|---|---|
| `List<T>` | **Invariante** | Solo `List<string>` con `List<string>` |
| `IEnumerable<out T>` | **Covariante** (`out`) | Un `IEnumerable<string>` donde se pide `IEnumerable<object>`: solo se **sacan** valores |
| `Action<in T>` | **Contravariante** (`in`) | Un `Action<object>` donde se pide `Action<string>`: solo se **meten** valores |

Una regla para recordarlo: lo que solo **produce** valores (`out`) puede ser más específico; lo que solo **consume** (`in`) puede ser más general; lo que hace las dos cosas (`List<T>`) no puede variar.

```csharp
string[] textos = ["a", "b"];
object[] objetos = textos;           // los arrays son covariantes... (permitido)
try
{
    objetos[0] = 42;                 // ...pero esto falla al EJECUTAR
}
catch (ArrayTypeMismatchException)
{
    Console.WriteLine("array: ArrayTypeMismatchException");
}

List<string> listaTextos = ["a", "b"];
// List<object> listaObjetos = listaTextos;      // error CS0029: no compila (List<T> es invariante)

IEnumerable<string> secuenciaTextos = listaTextos;
IEnumerable<object> secuenciaObjetos = secuenciaTextos;      // IEnumerable<out T> es covariante: sí compila
Console.WriteLine($"IEnumerable<object> con {secuenciaObjetos.Count()} elementos");

Action<object> mostrarObjeto = o => Console.WriteLine($"objeto: {o}");
Action<string> mostrarTexto = mostrarObjeto;                 // Action<in T> es contravariante: sí compila
mostrarTexto("hola");
```

Salida:

```text
array: ArrayTypeMismatchException
IEnumerable<object> con 2 elementos
objeto: hola
```

Los **arrays** son un caso especial y peligroso: C# permite tratar un `string[]` como `object[]` (por compatibilidad con los inicios del lenguaje), pero al meter un valor que no es del tipo real el programa falla al **ejecutar**. Es otra razón para preferir `List<T>`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Comparar con `<` valores de tipo `T` | Restringir con `where T : IComparable<T>` (o `INumber<T>` para números) |
| Pasar una `List<string>` donde se pide `List<object>` | Pedir `IEnumerable<object>` (solo lee) o hacer la función genérica |
| Usar `new T()` sin restricción | `where T : new()` |
| Usar `T` como clave de diccionario sin `notnull` | `where T : notnull` |
| Fiarse de la covarianza de los arrays | Usar `List<T>` |

## Para practicar

Los ejercicios [A2.3 y A2.5](ejercicios.md) usan restricciones y funciones como parámetro. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
