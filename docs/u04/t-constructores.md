# 4.C Constructores y enumerados

## Más de una forma de crear un objeto

A veces un objeto se puede crear de varias maneras: con unos valores por defecto, con datos propios o a partir de una «receta» ya preparada. Cada lenguaje lo resuelve a su manera.

C# permite **varios constructores** con distintos parámetros (**sobrecarga**) y **parámetros opcionales** con valor por defecto (`string nombre = "Margarita"`), que se pueden pasar **por nombre** (`new Pizza(Tamano.Mediana, extras: 2)`). Para «recetas» con otro significado se usan **métodos de fábrica estáticos** (`Pizza.CuatroQuesos(...)`).

```csharp
var a = new Pizza(Tamano.Mediana);
var b = Pizza.CuatroQuesos(Tamano.Grande);
var c = new Pizza(Tamano.Pequena, "Barbacoa");
Console.WriteLine(a);
Console.WriteLine(b);
Console.WriteLine(c);
a.AnadirExtra();
a.AnadirExtra(2);
Console.WriteLine(a);
Console.WriteLine($"total del pedido: {a.Precio() + b.Precio() + c.Precio()} €");
Console.WriteLine($"tamaños: {string.Join(", ", Enum.GetValues<Tamano>().Select(t => t.Etiqueta()))}");

enum Tamano { Pequena, Mediana, Grande }

static class TamanoExtensiones
{
    public static string Etiqueta(this Tamano t) => t switch
    {
        Tamano.Pequena => "Pequeña",
        Tamano.Mediana => "Mediana",
        Tamano.Grande => "Grande",
        _ => throw new ArgumentOutOfRangeException(nameof(t))
    };

    public static int PrecioBase(this Tamano t) => t switch
    {
        Tamano.Pequena => 6,
        Tamano.Mediana => 8,
        Tamano.Grande => 10,
        _ => throw new ArgumentOutOfRangeException(nameof(t))
    };
}

class Pizza
{
    public Tamano Tam { get; }
    public string Nombre { get; }
    public int Extras { get; private set; }

    public Pizza(Tamano tam, string nombre = "Margarita", int extras = 0)
    {
        Tam = tam;
        Nombre = nombre;
        Extras = extras;
    }

    public void AnadirExtra(int cantidad = 1)
    {
        Extras += cantidad;
    }

    public int Precio() => Tam.PrecioBase() + Extras;

    public override string ToString() => $"Pizza({Nombre}, {Tam.Etiqueta()}, {Extras} extras, {Precio()} €)";

    public static Pizza CuatroQuesos(Tamano tam) => new(tam, "Cuatro quesos", 3);
}
```

Salida:

```text
Pizza(Margarita, Mediana, 0 extras, 8 €)
Pizza(Cuatro quesos, Grande, 3 extras, 13 €)
Pizza(Barbacoa, Pequeña, 0 extras, 6 €)
Pizza(Margarita, Mediana, 3 extras, 11 €)
total del pedido: 30 €
tamaños: Pequeña, Mediana, Grande
```

En este ejemplo hay tres formas de obtener una pizza: **solo con el tamaño** (el nombre por defecto es «Margarita»), **con nombre propio** (`Barbacoa`) y la receta ya preparada con un **método de fábrica** (`CuatroQuesos`, que además trae tres extras). Después, `AnadirExtra` se llama **con y sin argumento**.

Para un método que se puede llamar con o sin argumento, como `AnadirExtra()` y `AnadirExtra(2)`, se usa **un solo método** con un parámetro opcional (`int cantidad = 1`).

### Varios constructores: sobrecarga y `: this(...)`

Cuando los constructores se parecen, que **todos acaben en uno solo** evita repetir reglas. Se encadenan con `: this(...)`:

```csharp
Console.WriteLine(new Cuenta("Ana"));
Console.WriteLine(new Cuenta("Luis", 250));
Console.WriteLine(new Cuenta("Eva", 100, "EUR"));
try
{
    new Cuenta("Pedro", -5);
}
catch (ArgumentException e)
{
    Console.WriteLine($"Error: {e.Message}");
}

class Cuenta
{
    public string Titular { get; }
    public int Saldo { get; }
    public string Moneda { get; }

    public Cuenta(string titular) : this(titular, 0) { }

    public Cuenta(string titular, int saldo) : this(titular, saldo, "EUR") { }

    public Cuenta(string titular, int saldo, string moneda)    // el único que valida
    {
        if (saldo < 0)
        {
            throw new ArgumentException("el saldo inicial no puede ser negativo");
        }
        Titular = titular;
        Saldo = saldo;
        Moneda = moneda;
    }

    public override string ToString() => $"{Titular}: {Saldo} {Moneda}";
}
```

Salida:

```text
Ana: 0 EUR
Luis: 250 EUR
Eva: 100 EUR
Error: el saldo inicial no puede ser negativo
```

!!! tip "Un solo sitio para las reglas"
    Procura que **todos los constructores acaben pasando por el mismo código** (el último, al que los demás llaman con `: this(...)`): así cualquier regla que añadas más adelante se escribe **una sola vez**.

## Enumerados

Un **enumerado** es un tipo con un **conjunto fijo y cerrado de valores**: los días de la semana, los estados de un pedido, los tamaños de una pizza. Es mejor que usar números o textos sueltos porque el compilador impide valores inventados (`Tamano.Gigante` no existe) y el código se lee solo.

Un `enum` de C# es solo una lista de nombres (por debajo, números enteros): **no puede tener campos ni métodos propios**, a diferencia de los de Kotlin o Java. Lo habitual es añadir los datos con **métodos de extensión** (la clase `TamanoExtensiones` del ejemplo) o con una `switch`. `Enum.GetValues<Tamano>()` devuelve todos los valores en orden.

En el ejemplo, cada tamaño tiene **datos asociados** (la etiqueta que se muestra y el precio base), de modo que el resto del programa no necesita un `if` por cada tamaño: pregunta el dato al propio valor (`tam.PrecioBase()`).

!!! warning "Un enumerado acepta cualquier número"
    `(Tamano)7` compila y crea un valor que no existe. Por eso las `switch` sobre enumerados llevan un caso final (`_ => throw ...`), como en el ejemplo.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Repetir las comprobaciones en cada constructor | Un constructor principal y los demás que lo llaman con `: this(...)` |
| Dos constructores casi iguales con distinto orden de parámetros | Usa parámetros opcionales o un método de fábrica con nombre claro |
| Usar números o textos para representar categorías (`tipo = 1`) | Un enumerado |
| Cadenas de `if`/`else` según el valor de un enumerado | Pon el dato o el comportamiento detrás de un método de extensión |
| Esperar que un `enum` tenga campos como en Kotlin | En C# se resuelve con métodos de extensión o con un `record` por valor |

## Para practicar

Los constructores, los enumerados y la sobrecarga se practican en [U4.6 · Prueba](prueba.md). Compara cómo se escribe en otro lenguaje con [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
