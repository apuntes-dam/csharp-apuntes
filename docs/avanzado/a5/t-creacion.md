# A5.A Patrones de creación

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 6 (clases, herencia, interfaces y SOLID).

Un **patrón de diseño** es una solución conocida a un problema que se repite al diseñar programas. No es código que copiar, sino una **idea con nombre**: cuando dices «aquí hay una fábrica», cualquier programador entiende la estructura sin leerla. Los patrones de **creación** resuelven cómo se crean los objetos.

## Singleton: una sola instancia

Hay cosas de las que solo debe existir **una**: la configuración de la aplicación, un registro de mensajes. Un **singleton** garantiza una única instancia y un punto de acceso global. En C# se consigue con un **constructor privado** (nadie más puede crearla) y una propiedad estática que la guarda:

```csharp
var a = Configuracion.Instancia;
var b = Configuracion.Instancia;
a.Idioma = "es";
Console.WriteLine($"b ve el cambio: {b.Idioma}");
Console.WriteLine($"misma instancia: {ReferenceEquals(a, b)}");

// Lazy<T>: la instancia se crea la primera vez que se pide, y de forma segura entre hilos
var perezoso = new Lazy<Conexion>(() => new Conexion());
Console.WriteLine($"creada antes de pedirla: {Conexion.Creadas}");
_ = perezoso.Value;
_ = perezoso.Value;
Console.WriteLine($"creadas tras pedirla dos veces: {Conexion.Creadas}");

class Configuracion
{
    // Se crea una sola vez, al usar la clase por primera vez (el CLR lo hace seguro entre hilos)
    public static Configuracion Instancia { get; } = new();

    private Configuracion() { }

    public string Idioma { get; set; } = "en";
}

class Conexion
{
    public static int Creadas { get; private set; }

    public Conexion() => Creadas++;
}
```

Salida:

```text
b ve el cambio: es
misma instancia: True
creada antes de pedirla: 0
creadas tras pedirla dos veces: 1
```

* `Configuracion.Instancia` se inicializa **una sola vez**, y el propio entorno de ejecución lo hace de forma segura entre hilos. Es la versión más sencilla.
* Si crear la instancia es caro y quizá no se use, **`Lazy<T>`** la crea la primera vez que se pide (`.Value`): el ejemplo muestra que la conexión no se crea hasta entonces y que pedirla dos veces solo la crea una.

!!! warning "Con moderación"
    Un singleton es un **estado global** disfrazado: cualquier parte del programa puede cambiarlo, y las pruebas se contagian unas a otras. En aplicaciones reales de .NET se prefiere que el contenedor de **inyección de dependencias** gestione la instancia única (`services.AddSingleton<T>()`) y se la pase a quien la necesite por el constructor.

## Fábrica: decidir qué clase crear

Cuando quien necesita un objeto no debería saber **qué clase concreta** crear (porque depende de un dato: un texto, una opción), se centraliza la decisión en una **fábrica**. Quien la usa solo conoce la interfaz:

```csharp
foreach (string tipo in new[] { "efectivo", "tarjeta", "bizum", "cheque" })
{
    try
    {
        IPago pago = FabricaPagos.Crear(tipo);
        Console.WriteLine($"{tipo}: {pago.Pagar(50)}");
    }
    catch (ArgumentException e)
    {
        Console.WriteLine(e.Message);
    }
}

interface IPago
{
    string Pagar(decimal importe);
}

class Efectivo : IPago
{
    public string Pagar(decimal importe) => $"{importe} € en efectivo";
}

class Tarjeta : IPago
{
    public string Pagar(decimal importe) => $"{importe} € con tarjeta (comisión {importe * 0.02m} €)";
}

class Bizum : IPago
{
    public string Pagar(decimal importe) => $"{importe} € por Bizum";
}

// La fábrica decide qué clase crear; quien la usa solo conoce IPago
static class FabricaPagos
{
    public static IPago Crear(string tipo) => tipo switch
    {
        "efectivo" => new Efectivo(),
        "tarjeta" => new Tarjeta(),
        "bizum" => new Bizum(),
        _ => throw new ArgumentException($"forma de pago desconocida: {tipo}"),
    };
}
```

Salida:

```text
efectivo: 50 € en efectivo
tarjeta: 50 € con tarjeta (comisión 1,00 €)
bizum: 50 € por Bizum
forma de pago desconocida: cheque
```

Si mañana se añade «cheque», solo cambia la fábrica; el código que llama a `Pagar` sigue igual. La `switch` expression con `_ => throw` rechaza lo desconocido con un mensaje claro.

## Builder: construir paso a paso

Un objeto con muchas piezas opcionales (una pizza, un correo, un pedido) genera constructores con diez parámetros imposibles de leer. El **builder** lo construye **paso a paso** con métodos encadenados y devuelve el objeto terminado. C# moderno ofrece además alternativas más cortas:

```csharp
// 1. Builder clásico: métodos encadenados y un Construir() final
var pizza = new Pizza.Builder("grande").ConQueso().ConIngrediente("jamón").ConIngrediente("piña").Construir();
Console.WriteLine(pizza);

// 2. En C# moderno, muchas veces basta un record con propiedades init y "with"
var pedido = new Pedido { Cliente = "Ana", Direccion = "Calle Sol 1" };
var urgente = pedido with { Urgente = true };
Console.WriteLine(pedido);
Console.WriteLine(urgente);

// 3. "required" obliga a rellenar lo imprescindible
// var malo = new Pedido { Cliente = "Eva" };      // error CS9035: falta Direccion

class Pizza
{
    private string tamano = "";
    private bool queso;
    private readonly List<string> ingredientes = [];

    private Pizza() { }

    public override string ToString() =>
        $"pizza {tamano}{(queso ? " con queso" : "")}, ingredientes: {(ingredientes.Count == 0 ? "ninguno" : string.Join(", ", ingredientes))}";

    public class Builder(string tamano)
    {
        private readonly Pizza pizza = new() { tamano = tamano };

        public Builder ConQueso()
        {
            pizza.queso = true;
            return this;
        }

        public Builder ConIngrediente(string ingrediente)
        {
            pizza.ingredientes.Add(ingrediente);
            return this;
        }

        public Pizza Construir() => pizza;
    }
}

record Pedido
{
    public required string Cliente { get; init; }
    public required string Direccion { get; init; }
    public bool Urgente { get; init; }
}
```

Salida:

```text
pizza grande con queso, ingredientes: jamón, piña
Pedido { Cliente = Ana, Direccion = Calle Sol 1, Urgente = False }
Pedido { Cliente = Ana, Direccion = Calle Sol 1, Urgente = True }
```

* El **builder clásico** (`Pizza.Builder`): cada método devuelve `this`, y `Construir()` entrega el objeto. Sirve si hay reglas o pasos que validar.
* Un **`record` con propiedades `init`** se rellena con un inicializador de objeto (`new Pedido { ... }`), y **`with`** crea una copia con un cambio. Muchas veces sustituye al builder.
* **`required`** obliga a dar un valor a las propiedades imprescindibles: si se olvida una, el compilador no deja compilar.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar singleton para todo lo «global» | Pasar las dependencias por el constructor; reservar el singleton para lo que de verdad es único |
| Un singleton modificable desde cualquier sitio | Hacerlo de solo lectura o proteger sus cambios |
| Una fábrica con un `switch` enorme que crece sin fin | Si crece mucho, usar un diccionario de funciones de creación |
| Un builder que permite construir un objeto incompleto | Validar en `Construir()` (o usar `required`) |
| Constructores con muchos parámetros del mismo tipo | Un builder, o un `record` con propiedades `init` |

## Para practicar

Los ejercicios [A5.1 a A5.3](ejercicios.md) usan estos patrones. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
