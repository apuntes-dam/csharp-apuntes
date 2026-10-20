# A5.B Patrones de comportamiento

Los patrones de **comportamiento** resuelven cómo se reparten el trabajo y se comunican los objetos.

## Estrategia: cambiar el algoritmo

Una tienda calcula el precio final con un descuento, pero el descuento puede ser de varios tipos: ninguno, un porcentaje, una cantidad fija. Escribir un `if` o `switch` con todos los tipos obliga a **modificar** la tienda cada vez que aparece uno nuevo. El patrón **estrategia** saca cada algoritmo a su propia pieza, todas con la misma forma, y la tienda recibe **la que debe usar**. Es el principio abierto/cerrado de [6.B](../../u06/t-solid-1.md) en acción.

```csharp
var palabras = new List<string> { "kiwi", "pera", "uva", "manzana" };

// 1. Con interfaz: cada estrategia es una clase
Console.WriteLine(new Tienda(new SinDescuento()).Cobrar(1000));
Console.WriteLine(new Tienda(new Porcentaje(20)).Cobrar(1000));
Console.WriteLine(new Tienda(new Fijo(150)).Cobrar(1000));

// 2. Con funciones: la estrategia es un Func (casi siempre es más corto)
Func<List<string>, List<string>> porLongitud = l => l.OrderBy(p => p.Length).ToList();
Func<List<string>, List<string>> alfabetico = l => l.Order(StringComparer.Ordinal).ToList();
Console.WriteLine($"por longitud: {string.Join(", ", Ordenar(palabras, porLongitud))}");
Console.WriteLine($"alfabético: {string.Join(", ", Ordenar(palabras, alfabetico))}");

List<string> Ordenar(List<string> lista, Func<List<string>, List<string>> estrategia) => estrategia(lista);

interface IDescuento
{
    int Aplicar(int precio);
}

class SinDescuento : IDescuento
{
    public int Aplicar(int precio) => precio;
}

class Porcentaje(int tanto) : IDescuento
{
    public int Aplicar(int precio) => precio - precio * tanto / 100;
}

class Fijo(int cantidad) : IDescuento
{
    public int Aplicar(int precio) => Math.Max(0, precio - cantidad);
}

class Tienda(IDescuento descuento)
{
    public int Cobrar(int precio) => descuento.Aplicar(precio);
}
```

Salida:

```text
1000
800
850
por longitud: uva, kiwi, pera, manzana
alfabético: kiwi, manzana, pera, uva
```

Hay dos maneras de escribirla:

* **Con una interfaz** (`IDescuento`): una clase por estrategia. Conviene si la estrategia tiene datos propios o varias operaciones.
* **Con una función** (`Func<...>`): si la estrategia es una sola operación, una lambda es mucho más corta. Es la aplicación directa de [A1.A](../a1/t-funciones.md): pasar el comportamiento como un valor. En C# apenas hay que escribir clases de estrategia sencillas.

## Observador: avisar sin conocer

Un sensor de temperatura necesita avisar a una pantalla, a una alarma, a un registro... Si el sensor llamara directamente a cada uno, tendría que **conocerlos**, y cada novedad le obligaría a cambiar. Con el patrón **observador**, el sensor mantiene una lista de interesados y les avisa a todos cuando cambia, **sin saber quiénes son**.

C# trae este patrón **dentro del lenguaje**: son los **eventos** (`event`). Los usas todos los días sin pensar en ello (el `Click` de un botón de la [unidad 7](../../u07/t-gui.md) es exactamente un observador):

```csharp
var sensor = new Sensor();

// Suscribirse con +=: el sensor no conoce a quien le escucha
sensor.Cambio += t => Console.WriteLine($"pantalla: {t} °C");
Action<int> alarma = t =>
{
    if (t > 30)
    {
        Console.WriteLine($"alarma: {t} °C");
    }
};
sensor.Cambio += alarma;

sensor.Fijar(25);
sensor.Fijar(32);

sensor.Cambio -= alarma;      // darse de baja con -=
Console.WriteLine("(la alarma se da de baja)");
sensor.Fijar(35);

class Sensor
{
    // event: desde fuera solo se puede suscribir (+=) o darse de baja (-=), no invocarlo ni borrar a los demás
    public event Action<int>? Cambio;

    public void Fijar(int grados) => Cambio?.Invoke(grados);
}
```

Salida:

```text
pantalla: 25 °C
pantalla: 32 °C
alarma: 32 °C
(la alarma se da de baja)
pantalla: 35 °C
```

* Quien quiere enterarse se **suscribe** con `+=`, y se da de baja con `-=` (hay que guardar la lambda en una variable para poder quitarla después).
* Se avisa por el **orden de suscripción**: primero la pantalla, luego la alarma.
* La palabra `event` protege al sensor: desde fuera solo se puede suscribir o darse de baja; nadie más puede **invocar** el evento ni borrar a los demás suscriptores.
* `Cambio?.Invoke(...)` evita un error si no hay nadie suscrito (el evento es `null`).

!!! warning "Las suscripciones que no se quitan"
    El que se suscribe a un evento queda **referenciado** por el emisor mientras no se dé de baja. Si un objeto de vida corta (una ventana) se suscribe a otro de vida larga (un servicio) y nunca se da de baja, el recolector de basura no puede liberarlo: una fuga de memoria muy común. Date de baja (`-=`) cuando termines.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Un `switch` que crece con cada variante del algoritmo | Estrategia: una pieza por variante |
| Una clase de estrategia con un solo método que podría ser una lambda | Usar `Func<...>` |
| El emisor conoce a todos los receptores | Observador: lista de suscriptores o un `event` |
| Invocar el evento sin comprobar que hay suscriptores | `evento?.Invoke(...)` |
| No darse de baja de un evento | `-=`, o hacer el suscriptor de vida igual de larga |
| Modificar la lista de suscriptores mientras se avisa | Avisar sobre una copia, o no cambiar suscripciones dentro del manejador |

## Para practicar

Los ejercicios [A5.4 y A5.5](ejercicios.md) usan estos patrones. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
