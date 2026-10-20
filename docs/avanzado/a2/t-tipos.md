# A2.A Tipos parametrizados

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5, en especial clases, interfaces y colecciones.

## Por qué existe

Imagina una caja que guarda **un valor de cualquier tipo**. Sin genéricos hay dos malas opciones: escribir una clase por cada tipo (`CajaInt`, `CajaTexto`...) o guardar `object` y **convertir** al sacar el valor, perdiendo la comprobación de tipos. Los **tipos parametrizados** (genéricos) resuelven esto: escribes la clase **una vez**, con un **parámetro de tipo** (`T`) que se concreta al usarla, y el compilador sigue comprobando todo.

Mira lo que pasa con la colección antigua `ArrayList` (que guarda `object`) frente a la genérica `List<int>`:

```csharp
using System.Collections;

// ArrayList (de antes de los genéricos) guarda cualquier cosa como object
var vieja = new ArrayList { 1, 2, "tres" };
try
{
    int suma = 0;
    foreach (object o in vieja)
    {
        suma += (int)o;      // el compilador no avisa: falla al ejecutar
    }
}
catch (InvalidCastException)
{
    Console.WriteLine("ArrayList: InvalidCastException al ejecutar");
}

// List<int> solo admite int: el error se detecta al compilar
var nueva = new List<int> { 1, 2 };
// nueva.Add("tres");      // error CS1503: no se puede convertir de string a int
Console.WriteLine($"List<int>: suma {nueva.Sum()}");
```

Salida:

```text
ArrayList: InvalidCastException al ejecutar
List<int>: suma 3
```

## La sintaxis en C#

* Una clase genérica declara su parámetro entre ángulos: `class Caja<T>`. Dentro de la clase, `T` se usa como cualquier otro tipo. Al usarla se concreta: `new Caja<int>(42)`.
* Una función genérica lo declara tras su nombre: `T Primero<T>(List<T> lista)`. Casi siempre **no hace falta escribir el tipo al llamarla**: el compilador lo **deduce** de los argumentos.
* Puede haber varios parámetros de tipo: `Par<A, B>`, `Dictionary<K, V>`. Por convención se usan letras mayúsculas sueltas, o `TElemento` si quieres más claridad.
* Los genéricos funcionan también en `record`, `struct`, `interface` y `delegate` (`Func<T, R>` es un delegado genérico).

## Un ejemplo

```csharp
var cajaNumero = new Caja<int>(42);
var cajaTexto = new Caja<string>("hola");
Console.WriteLine($"{cajaNumero.Valor + 1} y {cajaTexto.Valor.ToUpper()}");
Console.WriteLine(cajaNumero);
Console.WriteLine(cajaTexto);

// el compilador deduce el tipo en las funciones genéricas
Console.WriteLine(Primero(["a", "b", "c"]));
Console.WriteLine(Primero([10, 20]));

var (x, y) = Intercambiar(1, "uno");
Console.WriteLine($"{x} / {y}");

T Primero<T>(List<T> lista) => lista[0];

(B, A) Intercambiar<A, B>(A a, B b) => (b, a);

class Caja<T>(T valor)
{
    public T Valor { get; } = valor;

    public override string ToString() => $"Caja<{typeof(T).Name}>({Valor})";
}
```

Salida:

```text
43 y HOLA
Caja<Int32>(42)
Caja<String>(hola)
a
10
uno / 1
```

`typeof(T).Name` demuestra que `T` **existe en tiempo de ejecución**: una `Caja<int>` y una `Caja<string>` son tipos distintos de verdad (se explica más en [A2.C](t-coleccion.md)). Y `Intercambiar` devuelve una tupla de tipos `(B, A)`: el compilador sabe que `x` es un `string` y `y` un `int`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `ArrayList` o `object` para guardar «cualquier cosa» | Usar `List<T>` y genéricos |
| Escribir `new T()` o comparar con `<` dentro de una función genérica | `T` no se sabe hacer eso: hay que **restringirlo** ([A2.B](t-restricciones.md)) |
| Especificar el tipo cuando se puede deducir | Dejar que el compilador lo deduzca, salvo que haya ambigüedad |
| Ponerle nombres largos al parámetro de tipo sin necesidad | `T` basta; usa `TClave` y `TValor` cuando haya varios |

## Para practicar

Los ejercicios [A2.1 y A2.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
