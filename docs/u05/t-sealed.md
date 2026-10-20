# 5.D Jerarquías cerradas y clases de datos

## El problema: un conjunto de variantes conocido y cerrado

A veces los distintos tipos **no se van a ampliar**: un pago es en efectivo, con tarjeta o por Bizum, y no hay una cuarta posibilidad que inventar desde fuera. Con una jerarquía normal, quien use el tipo base nunca está seguro de haber cubierto todos los casos.

Una **jerarquía cerrada** le dice al compilador **exactamente cuáles son los subtipos posibles**. Así, cuando se hace algo distinto según el subtipo, el compilador puede **comprobar que no falte ninguno**.

En C# esto tiene una limitación que conviene conocer: **hasta C# 14 (la versión de .NET 10) el compilador no comprueba que un `switch` cubra todas las variantes de una jerarquía de clases**, como sí hace Kotlin o Java. Hay propuestas de tipos «union» para versiones posteriores. Mientras tanto se usan **`record`** (clases de datos) en una jerarquía, y el `switch` termina con un caso `_ => throw ...` por si se olvida una variante.

```csharp
var pagos = new List<Pago> { new Efectivo(50), new Tarjeta(100, "1234"), new Bizum(30, "600123456") };
int total = 0;
foreach (Pago pago in pagos)
{
    int c = Comision(pago);
    total += c;
    Console.WriteLine($"{Describir(pago)} -> comisión {c} €");
}
Console.WriteLine($"total de comisiones: {total} €");

string Describir(Pago pago) => pago switch
{
    Efectivo e => $"Efectivo de {e.Importe} €",
    Tarjeta t => $"Tarjeta ****{t.Ultimos4} de {t.Importe} €",
    Bizum b => $"Bizum al {b.Telefono} de {b.Importe} €",
    _ => throw new ArgumentOutOfRangeException(nameof(pago), "variante de pago desconocida")
};

int Comision(Pago pago) => pago switch
{
    Efectivo => 0,
    Tarjeta t => t.Importe * 2 / 100,
    Bizum => 1,
    _ => throw new ArgumentOutOfRangeException(nameof(pago), "variante de pago desconocida")
};

abstract record Pago(int Importe);

sealed record Efectivo(int Importe) : Pago(Importe);

sealed record Tarjeta(int Importe, string Ultimos4) : Pago(Importe);

sealed record Bizum(int Importe, string Telefono) : Pago(Importe);
```

Salida:

```text
Efectivo de 50 € -> comisión 0 €
Tarjeta ****1234 de 100 € -> comisión 2 €
Bizum al 600123456 de 30 € -> comisión 1 €
total de comisiones: 3 €
```

Observa:

* Cada variante lleva **datos distintos** (`Ultimos4` solo existe en la tarjeta, `Telefono` solo en Bizum). Por eso aquí no vale un enumerado.
* Las variantes son **`record`**, que solo guardan información y comparan por valor (ver [4.B](../u04/t-encapsulamiento.md)). Cada una es `sealed`: de ellas ya no se puede heredar.
* El `switch` usa **patrones de tipo** (`Tarjeta t => ...`): comprueba el tipo y, a la vez, da la variable ya convertida para leer sus datos.
* El comportamiento **está fuera** de las clases (en las funciones), no dentro de ellas. Se elige así cuando las variantes son solo datos y las operaciones se van añadiendo.

### Qué pasa si falta un caso

Como el compilador no avisa, el fallo aparece **al ejecutar**. Añadamos un pago nuevo y olvidemos tratarlo en `Describir`:

```csharp
var pagos = new List<Pago> { new Efectivo(50), new Cripto(10, "BTC") };
foreach (Pago pago in pagos)
{
    try
    {
        Console.WriteLine(Describir(pago));
    }
    catch (ArgumentOutOfRangeException e)
    {
        Console.WriteLine($"Error en tiempo de ejecución: {e.ParamName}");
    }
}

string Describir(Pago pago) => pago switch
{
    Efectivo e => $"Efectivo de {e.Importe} €",
    _ => throw new ArgumentOutOfRangeException(nameof(pago))
};

abstract record Pago(int Importe);
sealed record Efectivo(int Importe) : Pago(Importe);
sealed record Cripto(int Importe, string Moneda) : Pago(Importe);
```

Salida:

```text
Efectivo de 50 €
Error en tiempo de ejecución: pago
```

El `_ => throw` evita que el error pase en silencio, pero el compilador no lo detecta antes. Por eso, en C#, **al añadir una variante hay que buscar a mano todos los `switch`** sobre ese tipo (o cubrirlos con pruebas).

### Cerrar de verdad la jerarquía

Con `sealed` en cada variante, nadie hereda *de ellas*, pero **cualquiera puede crear otra variante nueva de `Pago`**. Para impedirlo, se pone el constructor de la base `private` y se escriben las variantes **dentro** de la base, de modo que solo ellas puedan heredar:

```csharp
Pago p = new Pago.Tarjeta(100, "1234");
Console.WriteLine(p switch
{
    Pago.Efectivo e => $"Efectivo de {e.Importe} €",
    Pago.Tarjeta t => $"Tarjeta ****{t.Ultimos4} de {t.Importe} €",
    _ => "otro"
});

abstract record Pago
{
    private Pago() { }        // el único constructor es privado: solo las variantes de dentro pueden heredar

    public sealed record Efectivo(int Importe) : Pago;

    public sealed record Tarjeta(int Importe, string Ultimos4) : Pago;
}
```

Salida:

```text
Tarjeta ****1234 de 100 €
```

Así, `record Otro : Pago;` **no compila** fuera de `Pago` (error CS0122: el constructor de la base es privado e inaccesible). Fíjate en que hay que escribir `record`: una `class` ni siquiera puede heredar de un `record`. Es más complejo y tampoco hace que el `switch` sea exhaustivo, así que muchas veces compensa más la versión sencilla.

## ¿Enumerado, jerarquía cerrada, abstracta o interfaz?

| Situación | Usa |
|---|---|
| Valores fijos y sencillos, a lo sumo con datos iguales para todos (días, tamaños, estados) | **Enumerado** |
| Variantes cerradas, **cada una con datos distintos** (pagos, resultados, formas de un mensaje) | **Jerarquía de `record` sellados** |
| Familia abierta con **código común** | Clase abstracta |
| Capacidad que clases distintas pueden ofrecer | Interfaz |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Poner un `_ =>` que **calla** las variantes nuevas | Haz que lance una excepción, como en los ejemplos, y revisa los `switch` al añadir una variante |
| Usar una jerarquía cerrada cuando otras personas deben poder añadir variantes | Interfaz o clase abstracta (jerarquía abierta) |
| Meter la lógica de cada caso en una cadena de `if` y `is` | Un `switch` con patrones |
| Usar un enumerado cuando cada variante necesita datos propios | Jerarquía de `record` |
| Esperar de C# la comprobación exhaustiva de Kotlin | No existe hasta C# 14: cúbrelo con excepciones y pruebas |

## Para practicar

El ejercicio de la biblioteca de [U5.1](herencia.md) usa clases de datos y una jerarquía cerrada para los usuarios. La comparación entre lenguajes de todos estos tipos de clase está en [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
