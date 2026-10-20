# 3.5 XML

**XML** es otro formato de texto para datos, más antiguo y más detallado que JSON. Los datos van entre **etiquetas** que forman un árbol. En C# se maneja con `System.Xml.Linq` (LINQ to XML), que ya viene incluido.

## Partes de un XML

```xml
<usuarios>                         <!-- elemento raíz: solo hay uno -->
    <usuario id="1">               <!-- elemento con un atributo -->
        <nombre>Juan</nombre>      <!-- elemento con texto -->
        <edad>30</edad>
    </usuario>
</usuarios>
```

| Parte | Qué es |
|---|---|
| **Elemento** | Una etiqueta de apertura y cierre, con contenido dentro: `<nombre>Juan</nombre>` |
| **Raíz** | El elemento que contiene a todos los demás; solo puede haber uno |
| **Atributo** | Un dato dentro de la etiqueta de apertura: `id="1"` |
| **Texto** | El contenido de un elemento |

## Leer, modificar y escribir

```csharp
using System.Xml.Linq;

string texto = """
<usuarios>
    <usuario>
        <id>1</id>
        <nombre>Juan</nombre>
        <edad>30</edad>
    </usuario>
    <usuario>
        <id>2</id>
        <nombre>Ana</nombre>
        <edad>25</edad>
    </usuario>
</usuarios>
""";

XDocument doc = XDocument.Parse(texto);
XElement raiz = doc.Root!;

foreach (XElement u in raiz.Elements("usuario"))
{
    Console.WriteLine($"ID: {u.Element("id")!.Value}, Nombre: {u.Element("nombre")!.Value}, Edad: {u.Element("edad")!.Value}");
}

// modificar
raiz.Elements("usuario").First().Element("edad")!.Value = "31";

// añadir un usuario nuevo
raiz.Add(new XElement("usuario",
    new XElement("id", 3),
    new XElement("nombre", "Eva"),
    new XElement("edad", 22)));

Console.WriteLine(doc);
```

Salida:

```text
ID: 1, Nombre: Juan, Edad: 30
ID: 2, Nombre: Ana, Edad: 25
<usuarios>
  <usuario>
    <id>1</id>
    <nombre>Juan</nombre>
    <edad>31</edad>
  </usuario>
  <usuario>
    <id>2</id>
    <nombre>Ana</nombre>
    <edad>25</edad>
  </usuario>
  <usuario>
    <id>3</id>
    <nombre>Eva</nombre>
    <edad>22</edad>
  </usuario>
</usuarios>
```

* `Elements("usuario")` da los elementos hijo con ese nombre; `Element("id")` da el primero.
* `.Value` es el texto del elemento (siempre texto: para usarlo como número, `int.Parse(...)` o la conversión `(int)elemento`).
* `Add` añade un hijo, y `new XElement("nombre", contenido)` construye un elemento con su contenido.
* `Console.WriteLine(doc)` muestra el XML con sangría; `doc.Save("archivo.xml")` lo guarda.

## Crear un árbol, atributos y errores

```csharp
using System.Xml;
using System.Xml.Linq;

// un árbol vacío con un atributo
var doc = new XDocument(new XElement("catalogo"));
doc.Root!.Add(new XElement("libro", new XAttribute("id", 1), new XElement("titulo", "Dune")));
doc.Root.Add(new XElement("libro", new XAttribute("id", 2), new XElement("titulo", "Fahrenheit 451")));

foreach (XElement libro in doc.Root.Elements("libro"))
{
    int id = (int)libro.Attribute("id")!;
    Console.WriteLine($"{id}: {libro.Element("titulo")!.Value}");
}
Console.WriteLine(doc);

// un XML mal escrito
try
{
    XDocument.Parse("<a><b></a>");
}
catch (XmlException)
{
    Console.WriteLine("Error: el XML no es válido");
}
```

Salida:

```text
1: Dune
2: Fahrenheit 451
<catalogo>
  <libro id="1">
    <titulo>Dune</titulo>
  </libro>
  <libro id="2">
    <titulo>Fahrenheit 451</titulo>
  </libro>
</catalogo>
Error: el XML no es válido
```

Un XML inválido (una etiqueta sin cerrar, mal anidada...) lanza `XmlException`. Con un archivo, además, se captura `FileNotFoundException` como en el JSON:

```csharp
using System.Xml;
using System.Xml.Linq;

XDocument? Cargar(string ruta)
{
    try
    {
        return XDocument.Load(ruta);
    }
    catch (FileNotFoundException)
    {
        Console.WriteLine($"Error: el archivo '{ruta}' no existe");
    }
    catch (XmlException)
    {
        Console.WriteLine($"Error: '{ruta}' no es un XML válido");
    }
    return null;
}

XDocument? ok = Cargar("datos.xml");
Console.WriteLine($"usuarios: {ok?.Root?.Elements("usuario").Count()}");
Cargar("falta.xml");
Cargar("roto.xml");
```

Salida:

```text
usuarios: 2
Error: el archivo 'falta.xml' no existe
Error: 'roto.xml' no es un XML válido
```

## ¿JSON o XML?

| | JSON | XML |
|---|---|---|
| Tamaño | Más corto | Más largo (etiquetas de apertura y cierre) |
| Tipos | Distingue números, textos y booleanos | Todo es texto |
| Atributos y comentarios | No | Sí |
| Uso habitual | Servicios web y configuración actual | Documentos, configuración antigua y formatos como SVG o Android |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar que `.Value` es texto | Conviértelo: `int.Parse(e.Value)` o `(int)e` |
| `Element("x")` devuelve `null` si no existe | Usa `?.` o comprueba antes (`!` solo si estás seguro) |
| Más de un elemento raíz | Un XML solo tiene una raíz |
| No controlar el archivo inexistente o el XML mal formado | Captura `FileNotFoundException` y `XmlException` |

## Para practicar

Haz el [ejercicio 3.5 de XML](xml.md).
