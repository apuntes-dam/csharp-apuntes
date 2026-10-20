# 6.A Diseñar una jerarquía de clases

En la [unidad 5](../u05/index.md) viste **cómo se escribe** la herencia, las clases abstractas y las interfaces. Aquí se trata de **decidir bien**: qué clases hacen falta, qué va en la base y qué en cada hija, y cómo se organizan los constructores para que el diseño aguante cambios.

## Un ejemplo completo: un hotel

```csharp
var hotel = new Hotel();
hotel.Agregar(new Individual(101));
hotel.Agregar(new Doble(201, desayuno: true));
hotel.Agregar(new Suite(301, extras: 2));
foreach (Habitacion h in hotel.Todas)
{
    Console.WriteLine(h.Describir());
}

hotel.Reservar(201);
Console.WriteLine($"tras reservar la 201: {hotel.Todas[1].Describir()}");
try
{
    hotel.Reservar(201);
}
catch (InvalidOperationException e)
{
    Console.WriteLine($"Error: {e.Message}");
}
try
{
    hotel.Reservar(999);
}
catch (ArgumentException e)
{
    Console.WriteLine($"Error: {e.Message}");
}
Console.WriteLine($"libres: {string.Join(", ", hotel.Libres().Select(h => h.Numero))}");
Console.WriteLine($"precio medio: {hotel.PrecioMedio()} €");

abstract class Habitacion
{
    public int Numero { get; }
    public bool Libre { get; set; } = true;

    protected Habitacion(int numero)
    {
        Numero = numero;
    }

    public abstract string Tipo();

    public abstract int Precio();

    public string Describir() => $"{Numero} {Tipo()}: {Precio()} €/noche{(Libre ? "" : " (ocupada)")}";
}

class Individual : Habitacion
{
    public Individual(int numero) : base(numero) { }

    public override string Tipo() => "Individual";

    public override int Precio() => 50;
}

class Doble : Habitacion
{
    private readonly bool _desayuno;

    public Doble(int numero, bool desayuno = false) : base(numero)
    {
        _desayuno = desayuno;
    }

    public override string Tipo() => _desayuno ? "Doble con desayuno" : "Doble";

    public override int Precio() => _desayuno ? 90 : 80;
}

class Suite : Habitacion
{
    private readonly int _extras;

    public Suite(int numero, int extras = 0) : base(numero)
    {
        _extras = extras;
    }

    public override string Tipo() => "Suite";

    public override int Precio() => 150 + 20 * _extras;
}

class Hotel
{
    private readonly List<Habitacion> _habitaciones = new();

    public IReadOnlyList<Habitacion> Todas => _habitaciones.AsReadOnly();

    public void Agregar(Habitacion habitacion)
    {
        _habitaciones.Add(habitacion);
    }

    public void Reservar(int numero)
    {
        Habitacion habitacion = _habitaciones.FirstOrDefault(h => h.Numero == numero)
            ?? throw new ArgumentException($"la habitación {numero} no existe");
        if (!habitacion.Libre)
        {
            throw new InvalidOperationException($"la habitación {numero} ya está ocupada");
        }
        habitacion.Libre = false;
    }

    public List<Habitacion> Libres() => _habitaciones.Where(h => h.Libre).ToList();

    public int PrecioMedio()
    {
        int suma = 0;
        foreach (Habitacion h in _habitaciones)
        {
            suma += h.Precio();
        }
        return suma / _habitaciones.Count;
    }
}
```

Salida:

```text
101 Individual: 50 €/noche
201 Doble con desayuno: 90 €/noche
301 Suite: 190 €/noche
tras reservar la 201: 201 Doble con desayuno: 90 €/noche (ocupada)
Error: la habitación 201 ya está ocupada
Error: la habitación 999 no existe
libres: 101, 301
precio medio: 110 €
```

Cómo se ha decidido el diseño:

* **La base lleva lo que es común a todas.** `Habitacion` tiene el `Numero` y si está `Libre`, que valen para cualquier habitación, y un método `Describir()` ya escrito.
* **Cada hija define solo lo que cambia.** El `Tipo()` y el `Precio()` son **abstractos**: cada habitación sabe el suyo. `Doble` y `Suite` añaden sus propios datos (`desayuno`, `extras`).
* **`Describir()` usa los métodos abstractos.** Es el patrón del [método plantilla](../u05/t-abstractas.md): la base fija el esquema, las hijas rellenan los huecos.
* **El hotel solo conoce `Habitacion`.** Si se añade un tipo nuevo, `Hotel` no cambia: es polimorfismo y composición («un hotel **tiene** habitaciones»).
* **Las reglas, dentro de su sitio.** Reservar comprueba que la habitación exista y esté libre, y lanza una excepción **antes** de modificar nada. Además, `Todas` devuelve la lista como `IReadOnlyList`: quien la recibe puede recorrerla, pero **no puede añadir ni quitar** habitaciones.
* **Los constructores reparten el trabajo.** La base inicializa el número; cada hija pasa ese dato hacia arriba con `base(numero)` y se queda con el suyo.

## Modificadores que ayudan a diseñar

| Necesito... | En C# |
|---|---|
| Clase que no se puede instanciar | `abstract class` |
| Clase de la que no se puede heredar | `sealed class` (por defecto **sí** se puede heredar) |
| Método que las hijas pueden sobrescribir | `virtual` en la base y `override` en la hija |
| Método que las hijas **deben** implementar | `abstract` |
| Impedir que una hija siga sobrescribiendo | `sealed override` |
| Solo contrato | `interface` |
| Jerarquía cerrada | `sealed` en las variantes (sin comprobación exhaustiva: ver [5.D](../u05/t-sealed.md)) |
| Visible para las subclases | `protected` |
| Valor que no cambia tras crear el objeto | `readonly` (campo) o propiedad solo con `get` o `init` |

## Preguntas antes de crear una jerarquía

| Pregunta | Si la respuesta es... |
|---|---|
| ¿La relación es «es un»? | No → composición o interfaz, no herencia |
| ¿Hay código o datos comunes? | Sí → clase base (abstracta si no tiene sentido por sí sola) |
| ¿Solo hay que fijar un contrato? | Sí → interfaz |
| ¿Lo que cambia es un cálculo o una regla? | Valorar pasar una **función** (ver [4.E](../u04/t-modelar.md)) |
| ¿Las variantes son un conjunto cerrado con datos distintos? | Jerarquía de `record` sellados (ver [5.D](../u05/t-sealed.md)) |
| ¿Se podrá usar cualquier hija donde se pida la base sin sorpresas? | Si no → hay que rediseñar (ver [6.C](t-solid-2.md), principio L) |

## Buenas prácticas

* **Jerarquías cortas y anchas** (una base y varias hijas) mejor que largas y estrechas.
* **La base, estable.** Cuanto más se cambia, más hijas se rompen. Si una base cambia a menudo, probablemente tiene demasiadas responsabilidades.
* **Cierra lo que no necesites abrir** (`sealed`): permitir heredar es una decisión. En C# las clases son heredables por defecto, así que **hay que acordarse de sellarlas**.
* **No expongas tus colecciones internas**: ofrece métodos o una vista de solo lectura.
* **Prefiere la composición** cuando dudes entre heredar o tener un atributo.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Una clase base con atributos que a algunas hijas no les sirven | Mover esos atributos a las hijas que los usan |
| Hijas que sobrescriben un método solo para «no hacer nada» o lanzar un error | Señal de que la jerarquía está mal (ver 6.C) |
| Repetir el mismo código en varias hijas | Subirlo a la base |
| Una base con métodos que solo usa una hija | Bajarlos a esa hija |

## Para practicar

Haz los ejercicios de [U6.1 · Jerarquía de clases](jerarquia.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
