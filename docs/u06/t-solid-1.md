# 6.B SOLID: responsabilidad única y abierto/cerrado

**SOLID** es el nombre de **cinco principios de diseño** con objetos. No son leyes ni se aplican al pie de la letra, pero resumen lo que se ha comprobado que hace el código más fácil de **entender, cambiar y probar**. Cada letra es un principio:

| Letra | Principio | En una frase |
|---|---|---|
| **S** | Responsabilidad única | Una clase, **una razón para cambiar** |
| **O** | Abierto/cerrado | Ampliar **sin modificar** lo que ya funciona |
| **L** | Sustitución de Liskov | Una hija debe poder usarse **donde se pida su base** sin sorpresas |
| **I** | Segregación de interfaces | **Contratos pequeños**, no obligar a implementar lo que no se usa |
| **D** | Inversión de dependencias | Depender de **abstracciones**, no de clases concretas |

En esta página, las dos primeras; en la [siguiente](t-solid-2.md), las otras tres. En cada ejemplo hay primero una versión **«antes»** con el problema y después una versión **«después»** ya corregida.

!!! info "Cómo leer los ejemplos"
    Los dos programas hacen **lo mismo** y producen el mismo resultado. Lo que cambia es **cómo está organizado el código**, y eso se nota cuando hay que modificarlo.

## S · Responsabilidad única

Una clase debería tener **una sola razón para cambiar**. Si una clase hace varias cosas (validar, guardar, avisar), cualquier cambio en una de ellas obliga a tocar la clase y puede romper las demás.

```csharp
var correos = new[] { "ana@correo.es", "correo-malo" };

var malo = new RegistroMalo();
foreach (string c in correos)
{
    Console.WriteLine($"[antes] {c} -> {malo.Registrar(c)}");
}

var repositorio = new RepositorioUsuarios();
var avisador = new Avisador();
var servicio = new ServicioRegistro(new ValidadorCorreo(), repositorio, avisador);
foreach (string c in correos)
{
    Console.WriteLine($"[después] {c} -> {servicio.Registrar(c)}");
}
Console.WriteLine($"usuarios guardados (después): {repositorio.Total}");
Console.WriteLine($"avisos enviados (después): {string.Join(", ", avisador.Enviados)}");

// ANTES: una sola clase valida, guarda y avisa (tres responsabilidades)
class RegistroMalo
{
    public List<string> Usuarios { get; } = new();
    public List<string> Enviados { get; } = new();

    public string Registrar(string correo)
    {
        if (!correo.Contains('@'))
        {
            return "correo no válido";                    // validar
        }
        Usuarios.Add(correo);                             // guardar
        Enviados.Add($"Bienvenido, {correo}");            // avisar
        return "registrado";
    }
}

// DESPUÉS: cada clase tiene una sola razón para cambiar
class ValidadorCorreo
{
    public bool EsValido(string correo) => correo.Contains('@');
}

class RepositorioUsuarios
{
    private readonly List<string> _correos = new();

    public void Guardar(string correo)
    {
        _correos.Add(correo);
    }

    public int Total => _correos.Count;
}

class Avisador
{
    public List<string> Enviados { get; } = new();

    public void Bienvenida(string correo)
    {
        Enviados.Add($"Bienvenido, {correo}");
    }
}

class ServicioRegistro
{
    private readonly ValidadorCorreo _validador;
    private readonly RepositorioUsuarios _repositorio;
    private readonly Avisador _avisador;

    public ServicioRegistro(ValidadorCorreo validador, RepositorioUsuarios repositorio, Avisador avisador)
    {
        _validador = validador;
        _repositorio = repositorio;
        _avisador = avisador;
    }

    public string Registrar(string correo)
    {
        if (!_validador.EsValido(correo))
        {
            return "correo no válido";
        }
        _repositorio.Guardar(correo);
        _avisador.Bienvenida(correo);
        return "registrado";
    }
}
```

Salida:

```text
[antes] ana@correo.es -> registrado
[antes] correo-malo -> correo no válido
[después] ana@correo.es -> registrado
[después] correo-malo -> correo no válido
usuarios guardados (después): 1
avisos enviados (después): Bienvenido, ana@correo.es
```

* **Antes:** `RegistroMalo` tiene tres responsabilidades. Si cambia la regla de validación, **o** el sitio donde se guardan los usuarios, **o** el texto del aviso, hay que editar la misma clase.
* **Después:** cada clase hace una cosa (`ValidadorCorreo`, `RepositorioUsuarios`, `Avisador`) y `ServicioRegistro` las **coordina**. Un cambio afecta a una sola clase, y cada pieza se puede probar por separado.

**Cómo detectarlo:** si para describir una clase necesitas decir «y» («valida **y** guarda **y** avisa»), probablemente son varias clases.

## O · Abierto/cerrado

El código debería estar **abierto a la ampliación** (se pueden añadir comportamientos nuevos) pero **cerrado a la modificación** (no hace falta tocar lo que ya funciona). Cada vez que se edita código que ya estaba probado, se arriesga a romperlo.

```csharp
var datos = new List<string> { "uno", "dos", "tres" };

foreach (string formato in new[] { "txt", "csv", "json" })
{
    try
    {
        Console.WriteLine($"[antes] {formato}: {Exportar(formato, datos)}");
    }
    catch (ArgumentException e)
    {
        Console.WriteLine($"[antes] {formato}: Error: {e.Message}");
    }
}

var exportadores = new List<IExportador> { new ExportadorTxt(), new ExportadorCsv(), new ExportadorJson() };
foreach (IExportador e in exportadores)
{
    Console.WriteLine($"[después] {e.Nombre}: {e.Exportar(datos)}");
}

// ANTES: añadir un formato obliga a EDITAR esta función
string Exportar(string formato, List<string> datos) => formato switch
{
    "txt" => string.Join(" ", datos),
    "csv" => string.Join(",", datos),
    _ => throw new ArgumentException($"formato desconocido: {formato}")
};

// DESPUÉS: añadir un formato es añadir una clase; lo que ya existe no cambia
interface IExportador
{
    string Nombre { get; }
    string Exportar(List<string> datos);
}

class ExportadorTxt : IExportador
{
    public string Nombre => "txt";
    public string Exportar(List<string> datos) => string.Join(" ", datos);
}

class ExportadorCsv : IExportador
{
    public string Nombre => "csv";
    public string Exportar(List<string> datos) => string.Join(",", datos);
}

// formato nuevo, escrito sin tocar el código anterior
class ExportadorJson : IExportador
{
    public string Nombre => "json";
    public string Exportar(List<string> datos) => "[" + string.Join(",", datos.Select(d => $"\"{d}\"")) + "]";
}
```

Salida:

```text
[antes] txt: uno dos tres
[antes] csv: uno,dos,tres
[antes] json: Error: formato desconocido: json
[después] txt: uno dos tres
[después] csv: uno,dos,tres
[después] json: ["uno","dos","tres"]
```

* **Antes:** para añadir el formato `json` hay que **editar** la función `Exportar`. Y así con cada formato nuevo: el `switch` no deja de crecer.
* **Después:** cada formato es una clase que cumple un contrato (`IExportador`). Para añadir `json` se **escribe una clase nueva** y las demás **no se tocan**.

**Cómo detectarlo:** una cadena de `if`/`else if` o un `switch` que cambia según un «tipo» y que hay que ampliar cada vez que aparece uno nuevo. La solución casi siempre combina una **interfaz** ([5.C](../u05/t-interfaces.md)) con **polimorfismo**.

!!! warning "No te pases"
    No hace falta una interfaz por cada `if`. Con dos o tres casos que probablemente no cambiarán, un `if` sencillo es perfectamente válido. Aplica el principio cuando notes que **de verdad** se añaden variantes con frecuencia.

## Para practicar

Haz los ejercicios S y O de [U6.2 · SOLID](solid.md). Y la comparación entre lenguajes de interfaces y funciones está en [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
