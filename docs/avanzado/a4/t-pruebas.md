# A4.A Qué es una prueba y cómo se escribe

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 6, en especial excepciones, clases e interfaces. Si aún no has visto cómo se crea un proyecto de pruebas, mira [1.4 Pruebas con xUnit](../../u01/04-pruebas.md).

Una **prueba automática** es un programa pequeño que ejecuta tu código con unos datos y **comprueba que el resultado es el esperado**. Se escribe una vez y se ejecuta cuantas veces haga falta: cada vez que cambias algo, en segundos sabes si sigue funcionando todo lo anterior.

## La receta: preparar, actuar, comprobar

Casi toda prueba tiene tres partes (el patrón **AAA**, de *Arrange, Act, Assert*):

| Parte | Qué hace | Ejemplo |
|---|---|---|
| **Preparar** (*arrange*) | Crea los objetos y datos de partida | `var carrito = new Carrito();` |
| **Actuar** (*act*) | Ejecuta **lo que se prueba**, una sola cosa | `int total = carrito.Total();` |
| **Comprobar** (*assert*) | Compara lo obtenido con lo esperado | `Assert.Equal(0, total);` |

Dos consejos: una prueba comprueba **una idea** (por eso su nombre dice qué comprueba: `ElTotalSumaPrecioPorCantidad`), y las pruebas son **independientes** entre sí: no deben depender de que otra se haya ejecutado antes.

## El marco de pruebas de C#: xUnit

En .NET los marcos de pruebas más usados son **xUnit**, NUnit y MSTest. Esta web usa **xUnit**, que es el que crea por defecto `dotnet new xunit`. Una prueba es un método marcado con **`[Fact]`** dentro de una clase pública, y las comprobaciones son métodos de **`Assert`**:

| Comprobación | Significa |
|---|---|
| `Assert.Equal(esperado, obtenido)` | Son iguales (¡el **esperado va primero**!) |
| `Assert.True(condicion)` / `Assert.False(...)` | La condición es cierta / falsa |
| `Assert.Null(x)` / `Assert.NotNull(x)` | Es / no es `null` |
| `Assert.Throws<T>(() => ...)` | El código lanza esa excepción ([A4.B](t-casos.md)) |
| `Assert.Contains(x, coleccion)` | La colección contiene el elemento |

Un detalle de xUnit que lo diferencia de otros marcos: crea una **instancia nueva de la clase de pruebas para cada `[Fact]`**. Por eso un campo o el constructor de la clase hacen de «preparación previa» (lo que otros marcos llaman `SetUp`) sin que lo pidas.

## Un ejemplo: un carrito de la compra

La clase que se prueba (precios en céntimos, para no usar decimales) y su proyecto de pruebas, que se ha ejecutado con `dotnet test`:

`Carrito.cs`:

```csharp
public class Carrito
{
    private readonly List<(string Producto, int Precio, int Cantidad)> lineas = [];

    // Los precios van en céntimos
    public void Agregar(string producto, int precio, int cantidad)
    {
        if (cantidad <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(cantidad), "la cantidad debe ser positiva");
        }
        lineas.Add((producto, precio, cantidad));
    }

    public int Total() => lineas.Sum(l => l.Precio * l.Cantidad);

    public int Unidades => lineas.Sum(l => l.Cantidad);
}
```

`CarritoTests.cs`:

```csharp
using Xunit;

public class CarritoTests
{
    [Fact]
    public void UnCarritoVacioTotalizaCero()
    {
        var carrito = new Carrito();                   // preparar

        int total = carrito.Total();                   // actuar

        Assert.Equal(0, total);                        // comprobar
    }

    [Fact]
    public void ElTotalSumaPrecioPorCantidad()
    {
        var carrito = new Carrito();
        carrito.Agregar("café", 120, 2);
        carrito.Agregar("tostada", 250, 1);

        Assert.Equal(490, carrito.Total());
    }

    [Fact]
    public void CuentaLasUnidades()
    {
        var carrito = new Carrito();
        carrito.Agregar("café", 120, 2);
        carrito.Agregar("tostada", 250, 1);

        Assert.Equal(3, carrito.Unidades);
    }
}
```

Resultado de `dotnet test`:

```text
Con error: 0, Superado: 3, Omitido: 0, Total: 3
```

Las tres pruebas pasan. Cada una tiene un nombre que se lee como una frase y una sola comprobación principal.

## Cuando una prueba falla

Una prueba que falla es **buena noticia**: te dice dónde está el problema. Aquí se ha escrito a propósito una comprobación con un valor esperado equivocado (120 en vez de 240):

`Carrito.cs`:

```csharp
public class Carrito
{
    private readonly List<(string Producto, int Precio, int Cantidad)> lineas = [];

    // Los precios van en céntimos
    public void Agregar(string producto, int precio, int cantidad)
    {
        if (cantidad <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(cantidad), "la cantidad debe ser positiva");
        }
        lineas.Add((producto, precio, cantidad));
    }

    public int Total() => lineas.Sum(l => l.Precio * l.Cantidad);

    public int Unidades => lineas.Sum(l => l.Cantidad);
}
```

`CarritoFalloTests.cs`:

```csharp
using Xunit;

public class CarritoFalloTests
{
    [Fact]
    public void ElTotalEstaMalCalculadoAPropositoEnLaPrueba()
    {
        var carrito = new Carrito();
        carrito.Agregar("café", 120, 2);

        Assert.Equal(120, carrito.Total());      // el esperado correcto sería 240
    }
}
```

Resultado de `dotnet test`:

```text
Con error CarritoFalloTests.ElTotalEstaMalCalculadoAPropositoEnLaPrueba
    Assert.Equal() Failure: Values differ
    Expected: 120
    Actual:   240
Con error: 1, Superado: 0, Omitido: 0, Total: 1
```

El mensaje dice **qué esperaba** (`Expected`) y **qué obtuvo** (`Actual`). Lee siempre los dos: la mitad de las veces el fallo está en la prueba y no en el código.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Poner `Assert.Equal(obtenido, esperado)` al revés | El esperado va primero: los mensajes de fallo salen al revés si no |
| Varias ideas en una sola prueba | Una prueba, una idea |
| Pruebas que dependen de otra o del orden | Cada prueba prepara lo suyo (xUnit ya crea una instancia nueva por prueba) |
| Nombres como `Prueba1` | Nombres que digan qué se comprueba |
| Pruebas sin ninguna comprobación | Toda prueba termina en un `Assert` |
| Probar la implementación en vez del comportamiento | Comprueba qué devuelve o qué cambia, no cómo lo hace por dentro |

## Para practicar

Los ejercicios [A4.1 a A4.4](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
