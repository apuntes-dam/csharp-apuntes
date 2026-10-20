# 6.C SOLID: Liskov, interfaces pequeñas e inversión de dependencias

Los tres principios restantes de [SOLID](t-solid-1.md). Cada ejemplo vuelve a mostrar la versión «antes» y la «después», con el mismo resultado visible.

## L · Sustitución de Liskov

Donde se pida una clase base, **tiene que poder usarse cualquiera de sus hijas** sin que el programa falle ni se comporte de forma rara. Si una hija **rompe** lo que la base prometía, la herencia está mal planteada, aunque la frase «es un» suene bien.

```csharp
var antes = new List<Documento> { new Documento("ficha", "v1"), new DocumentoSoloLectura("contrato", "texto original") };
foreach (Documento d in antes)
{
    try
    {
        d.Guardar("v2");
        Console.WriteLine($"[antes] {d.Nombre}: guardado");
    }
    catch (NotSupportedException e)
    {
        Console.WriteLine($"[antes] {d.Nombre}: Error: {e.Message}");
    }
}

// solo se pueden guardar documentos Editable: con SoloLectura ni siquiera compilaría
var editables = new List<Editable> { new Editable("ficha", "v1") };
foreach (Editable d in editables)
{
    d.Guardar("v2");
    Console.WriteLine($"[después] {d.Nombre}: guardado");
}
var contrato = new SoloLectura("contrato", "texto original");
Console.WriteLine($"[después] {contrato.Nombre} se puede leer: {contrato.Leer()}");

// ANTES: DocumentoSoloLectura "es un" Documento, pero NO se puede usar en su lugar
class Documento
{
    public string Nombre { get; }
    public string Contenido { get; protected set; }

    public Documento(string nombre, string contenido)
    {
        Nombre = nombre;
        Contenido = contenido;
    }

    public virtual void Guardar(string texto)
    {
        Contenido = texto;
    }
}

class DocumentoSoloLectura : Documento
{
    public DocumentoSoloLectura(string nombre, string contenido) : base(nombre, contenido) { }

    public override void Guardar(string texto)
    {
        throw new NotSupportedException("es de solo lectura");
    }
}

// DESPUÉS: la jerarquía refleja lo que cada clase SABE hacer
abstract class Lectura
{
    public string Nombre { get; }
    protected string Contenido { get; set; }

    protected Lectura(string nombre, string contenido)
    {
        Nombre = nombre;
        Contenido = contenido;
    }

    public string Leer() => Contenido;
}

class Editable : Lectura
{
    public Editable(string nombre, string contenido) : base(nombre, contenido) { }

    public void Guardar(string texto)
    {
        Contenido = texto;
    }
}

class SoloLectura : Lectura
{
    public SoloLectura(string nombre, string contenido) : base(nombre, contenido) { }
}
```

Salida:

```text
[antes] ficha: guardado
[antes] contrato: Error: es de solo lectura
[después] ficha: guardado
[después] contrato se puede leer: texto original
```

* **Antes:** `DocumentoSoloLectura` «es un» `Documento`, pero cuando se le pide guardar **falla**. Un bucle que guarda todos los documentos funciona con unos y se rompe con otros: la hija **no puede sustituir** a la base.
* **Después:** la jerarquía refleja lo que cada clase **sabe hacer**. Todos se pueden **leer** (`Lectura`); solo los `Editable` se pueden **guardar**. Ya no hay forma de pedir guardar a un documento de solo lectura.

**Señales de que se viola:** una hija que lanza «no soportado» (`NotSupportedException`), que deja un método vacío, o código que necesita comprobar el tipo concreto antes de usar la clase base.

## I · Segregación de interfaces

Es mejor tener **varias interfaces pequeñas** y específicas que una grande. Ninguna clase debería verse obligada a implementar métodos que no usa.

```csharp
IMaquina vieja = new ImpresoraSencillaMala();
Console.WriteLine($"[antes] sencilla imprime: {vieja.Imprimir("informe")}");
try
{
    vieja.Escanear("foto");
}
catch (NotSupportedException e)
{
    Console.WriteLine($"[antes] sencilla escanea: Error: {e.Message}");
}

var sencilla = new ImpresoraSencilla();
var multi = new Multifuncion();
Console.WriteLine($"[después] sencilla imprime: {sencilla.Imprimir("informe")}");
// sencilla.Escanear("foto");   // ya no compila: la clase no promete escanear
Console.WriteLine($"[después] multifunción escanea: {multi.Escanear("foto")}");

// ANTES: una interfaz «gorda» obliga a implementar lo que no se usa
interface IMaquina
{
    string Imprimir(string texto);
    string Escanear(string texto);
}

class ImpresoraSencillaMala : IMaquina
{
    public string Imprimir(string texto) => $"impreso: {texto}";

    public string Escanear(string texto) => throw new NotSupportedException("no soportado");
}

// DESPUÉS: contratos pequeños; cada clase cumple solo los suyos
interface IImpresora
{
    string Imprimir(string texto);
}

interface IEscaner
{
    string Escanear(string texto);
}

class ImpresoraSencilla : IImpresora
{
    public string Imprimir(string texto) => $"impreso: {texto}";
}

class Multifuncion : IImpresora, IEscaner
{
    public string Imprimir(string texto) => $"impreso: {texto}";

    public string Escanear(string texto) => $"escaneado: {texto}";
}
```

Salida:

```text
[antes] sencilla imprime: impreso: informe
[antes] sencilla escanea: Error: no soportado
[después] sencilla imprime: impreso: informe
[después] multifunción escanea: escaneado: foto
```

* **Antes:** la interfaz `IMaquina` obliga a toda impresora a «escanear», aunque no pueda: la implementación acaba lanzando un error.
* **Después:** `IImpresora` e `IEscaner` son contratos separados. La impresora sencilla cumple uno; la multifunción, los dos. Pedir escanear a una impresora sencilla **ni siquiera se puede escribir**.

**Señales de que se viola:** métodos que lanzan «no soportado» o que están vacíos solo para cumplir la interfaz.

## D · Inversión de dependencias

Las clases importantes no deberían depender de **detalles concretos** (el reloj del sistema, una base de datos, un servicio de correo), sino de **abstracciones** (un contrato) que se les entregan **desde fuera**. Así se puede cambiar el detalle o, muy importante, **sustituirlo por uno falso en las pruebas**.

```csharp
SaludadorMalo.Saludo();
Console.WriteLine("[antes] el resultado depende de la hora del equipo");
foreach (int hora in new[] { 9, 15, 22 })
{
    Console.WriteLine($"[después] {hora} h -> {new Saludador(new RelojFijo(hora)).Saludo()}");
}
string real = new Saludador(new RelojReal()).Saludo();
Console.WriteLine($"con el reloj real: {(real.Length > 0 ? "funciona" : "falla")}");

// ANTES: el Saludador crea su propio reloj: no se puede probar con una hora concreta
static class SaludadorMalo
{
    public static string Saludo()
    {
        int hora = DateTime.Now.Hour;
        if (hora < 12) return "Buenos días";
        if (hora < 20) return "Buenas tardes";
        return "Buenas noches";
    }
}

// DESPUÉS: depende de una abstracción que le dan desde fuera
interface IReloj
{
    int Hora();
}

class RelojReal : IReloj
{
    public int Hora() => DateTime.Now.Hour;
}

class RelojFijo : IReloj
{
    private readonly int _hora;

    public RelojFijo(int hora)
    {
        _hora = hora;
    }

    public int Hora() => _hora;
}

class Saludador
{
    private readonly IReloj _reloj;

    public Saludador(IReloj reloj)
    {
        _reloj = reloj;
    }

    public string Saludo()
    {
        int hora = _reloj.Hora();
        if (hora < 12) return "Buenos días";
        if (hora < 20) return "Buenas tardes";
        return "Buenas noches";
    }
}
```

Salida:

```text
[antes] el resultado depende de la hora del equipo
[después] 9 h -> Buenos días
[después] 15 h -> Buenas tardes
[después] 22 h -> Buenas noches
con el reloj real: funciona
```

* **Antes:** `SaludadorMalo` pregunta la hora al sistema. Para comprobar que a las 22 h dice «Buenas noches» habría que esperar a las 22 h: no se puede probar.
* **Después:** `Saludador` recibe un `IReloj` (un contrato). En el programa real se le da el `RelojReal`; en las pruebas, un `RelojFijo` con la hora que se quiera. El `Saludador` **no cambia**.

A esta técnica de entregar las dependencias desde fuera, normalmente por el **constructor**, se le llama **inyección de dependencias**. Fíjate en que `Saludador` no escribe `new RelojReal()` por dentro: *recibe* un `IReloj`.

!!! tip "Un objeto falso se llama «doble de prueba»"
    `RelojFijo` es un **doble de prueba** (*fake*): hace de reloj pero controlas lo que devuelve. Es la base de las pruebas automáticas serias, y solo es posible si se diseña con este principio.

!!! info "En .NET ya existe un reloj abstracto: `TimeProvider`"
    Desde .NET 8 la propia biblioteca trae la clase abstracta `TimeProvider` (`TimeProvider.System.GetLocalNow()` es el reloj real) pensada para este mismo principio. En proyectos reales se usa esa en vez de inventar un `IReloj`; aquí se escribe a mano para que se vea la idea.

## Resumen de los cinco principios

| Principio | Señal de que falla | Remedio habitual |
|---|---|---|
| **S** | Una clase hace varias cosas («y») | Dividirla; una clase coordina |
| **O** | Cadena de `if` según un tipo que crece | Interfaz + una clase por variante |
| **L** | Una hija lanza «no soportado» o necesita comprobaciones de tipo | Rehacer la jerarquía según lo que **sabe hacer** cada clase |
| **I** | Métodos vacíos o que fallan solo para cumplir la interfaz | Dividir la interfaz |
| **D** | La clase crea por dentro sus dependencias y no se puede probar | Recibir una abstracción por el constructor |

## Para practicar

Haz los ejercicios L, I y D de [U6.2 · SOLID](solid.md): en todos hay que partir de un código con el problema y refactorizarlo, como en estos ejemplos. [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) compara cómo se escriben interfaces y funciones en cada lenguaje.
