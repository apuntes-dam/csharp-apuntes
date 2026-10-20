# A2.C Tu colección y qué pasa al ejecutar

## Una colección genérica propia

Con lo que ya sabes se puede escribir una colección propia. Una **pila** (*stack*) es una colección donde el último en entrar es el primero en salir: `Apilar` añade por arriba y `Desapilar` quita el de arriba. Si además implementa `IEnumerable<T>`, se puede recorrer con `foreach` y usar con LINQ como cualquier colección de la biblioteca:

```csharp
using System.Collections;

var pila = new Pila<string>();
pila.Apilar("uno");
pila.Apilar("dos");
pila.Apilar("tres");

Console.WriteLine($"elementos: {pila.Cantidad}");
Console.WriteLine($"recorrida con foreach: {string.Join(", ", pila)}");
Console.WriteLine($"desapila: {pila.Desapilar()}");
Console.WriteLine($"desapila: {pila.Desapilar()}");
Console.WriteLine($"desapila: {pila.Desapilar()}");

try
{
    pila.Desapilar();
}
catch (InvalidOperationException e)
{
    Console.WriteLine($"error: {e.Message}");
}

class Pila<T> : IEnumerable<T>
{
    private readonly List<T> elementos = [];

    public int Cantidad => elementos.Count;

    public void Apilar(T x) => elementos.Add(x);

    public T Desapilar()
    {
        if (elementos.Count == 0)
        {
            throw new InvalidOperationException("la pila está vacía");
        }
        T ultimo = elementos[^1];
        elementos.RemoveAt(elementos.Count - 1);
        return ultimo;
    }

    // recorre de la cima hacia abajo
    public IEnumerator<T> GetEnumerator()
    {
        for (int i = elementos.Count - 1; i >= 0; i--)
        {
            yield return elementos[i];
        }
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

Salida:

```text
elementos: 3
recorrida con foreach: tres, dos, uno
desapila: tres
desapila: dos
desapila: uno
error: la pila está vacía
```

Qué hay que fijarse:

* `Pila<T>` sirve para cualquier tipo, y el compilador sabe que `Pila<string>.Desapilar()` devuelve un `string`.
* Implementar **`IEnumerable<T>`** exige dos métodos `GetEnumerator` (uno genérico y uno no genérico por compatibilidad). El genérico se escribe cómodamente con **`yield return`**, que genera el recorrido por ti.
* Una pila vacía no tiene qué devolver: lanzamos una `InvalidOperationException`, que es lo que hace la `Stack<T>` de la biblioteca. En C# ya existe `Stack<T>`; escribirla a mano sirve para aprender.

## Qué pasa con los tipos al ejecutar

Los genéricos de C# son **reales** también en tiempo de ejecución (se dice *reificados*). Es una diferencia importante con Java y Kotlin, donde los tipos genéricos se «borran» al compilar:

```csharp
var a = new List<int>();
var b = new List<string>();
Console.WriteLine($"tipo de a: {a.GetType()}");
Console.WriteLine($"tipo de b: {b.GetType()}");
Console.WriteLine($"¿mismo tipo? {a.GetType() == b.GetType()}");

// el tipo se conoce en tiempo de ejecución
Console.WriteLine(Describir<int>());
Console.WriteLine(Describir<string>());
Console.WriteLine(Describir<List<DateTime>>());

// con tipos por valor no hay "boxing": List<int> guarda int de verdad
var enteros = new List<int> { 1, 2, 3 };
Console.WriteLine($"suma: {enteros.Sum()}");

string Describir<T>() => $"T es {typeof(T).Name} (¿por valor? {typeof(T).IsValueType})";
```

Salida:

```text
tipo de a: System.Collections.Generic.List`1[System.Int32]
tipo de b: System.Collections.Generic.List`1[System.String]
¿mismo tipo? False
T es Int32 (¿por valor? True)
T es String (¿por valor? False)
T es List`1 (¿por valor? False)
suma: 6
```

* `List<int>` y `List<string>` son **tipos distintos** al ejecutar, y se puede preguntar por `typeof(T)` dentro de una función genérica. En Java no se puede: el tipo se borra y `new T()` o `T.class` no existen.
* Con tipos por valor (`int`, `double`, `struct`), `List<int>` guarda enteros directamente, **sin convertirlos** en objetos (*boxing*). Por eso es más rápida y gasta menos memoria que la antigua `ArrayList`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Escribir una colección propia cuando ya existe en la biblioteca | Usa `List<T>`, `Stack<T>`, `Queue<T>`, `Dictionary<K, V>`... y escribe la tuya solo para aprender o si necesitas un comportamiento distinto |
| No hacer la colección recorrible | Implementar `IEnumerable<T>` (con `yield return`) |
| Devolver `default` de un `T` cuando no hay valor | Lanzar una excepción o usar el patrón `TryXxx(out T valor)` |
| Creer que los genéricos de C# se borran como los de Java | En C# se conocen en tiempo de ejecución |

## Para practicar

Los ejercicios [A2.4 y A2.6](ejercicios.md) usan una pila y una caché genérica. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
