# 3.4 JSON

**JSON** es un formato de texto para guardar e intercambiar datos. Lo usan casi todas las aplicaciones y servicios web. En C# viene incluido en la biblioteca estándar: `System.Text.Json`.

## Qué contiene un JSON

| Elemento | Ejemplo |
|---|---|
| Objeto (pares clave-valor) | `{"nombre": "Ana", "edad": 25}` |
| Lista | `[1, 2, 3]` |
| Texto (siempre con comillas dobles) | `"hola"` |
| Número | `42`, `3.14` |
| Booleano y nulo | `true`, `false`, `null` |

Un objeto se parece a un mapa y una lista a una lista, y se pueden anidar sin límite.

## Leer, modificar y escribir

```csharp
using System.Text.Json;
using System.Text.Json.Nodes;

string texto = """
{
    "usuarios": [
        {"id": 1, "nombre": "Juan", "edad": 30},
        {"id": 2, "nombre": "Ana", "edad": 25}
    ]
}
""";

JsonNode datos = JsonNode.Parse(texto)!;
JsonArray usuarios = datos["usuarios"]!.AsArray();

foreach (JsonNode? u in usuarios)
{
    Console.WriteLine($"ID: {u!["id"]}, Nombre: {u["nombre"]}, Edad: {u["edad"]}");
}

// modificar y añadir
usuarios[0]!["edad"] = 31;
usuarios.Add(new JsonObject { ["id"] = 3, ["nombre"] = "Eva", ["edad"] = 22 });

var opciones = new JsonSerializerOptions { WriteIndented = true };
Console.WriteLine(datos.ToJsonString(opciones));
```

Salida:

```text
ID: 1, Nombre: Juan, Edad: 30
ID: 2, Nombre: Ana, Edad: 25
{
  "usuarios": [
    {
      "id": 1,
      "nombre": "Juan",
      "edad": 31
    },
    {
      "id": 2,
      "nombre": "Ana",
      "edad": 25
    },
    {
      "id": 3,
      "nombre": "Eva",
      "edad": 22
    }
  ]
}
```

`JsonNode.Parse` convierte el texto en un árbol que se consulta como un mapa (`datos["usuarios"]`) y como una lista (`usuarios[0]`). `ToJsonString` hace el camino inverso.

!!! warning "Las tildes salen como secuencias de escape"
    Por seguridad, `System.Text.Json` escribe los caracteres que no son ASCII como secuencias `\uXXXX`. El JSON sigue siendo válido, pero se lee mal. Para escribir las tildes tal cual, usa el codificador relajado:

```csharp
using System.Text.Encodings.Web;
using System.Text.Json;
using System.Text.Json.Nodes;

var persona = new JsonObject { ["nombre"] = "José", ["ciudad"] = "Cádiz" };

Console.WriteLine(persona.ToJsonString());

var opciones = new JsonSerializerOptions { Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping };
Console.WriteLine(persona.ToJsonString(opciones));
```

Salida:

```text
{"nombre":"Jos\u00E9","ciudad":"C\u00E1diz"}
{"nombre":"José","ciudad":"Cádiz"}
```

### Con una clase: `record`

Cuando conoces la forma de los datos, es más cómodo **deserializar** el JSON directamente a objetos de una clase propia:

```csharp
#:property PublishAot=false
using System.Text.Json;

string texto = """[{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]""";

var opciones = new JsonSerializerOptions { PropertyNameCaseInsensitive = true };
List<Usuario> usuarios = JsonSerializer.Deserialize<List<Usuario>>(texto, opciones)!;

foreach (Usuario u in usuarios)
{
    Console.WriteLine(u);
}
Console.WriteLine($"mayores de 26: {usuarios.Count(u => u.Edad > 26)}");

Console.WriteLine(JsonSerializer.Serialize(usuarios[0]));

record Usuario(int Id, string Nombre, int Edad);
```

Salida:

```text
Usuario { Id = 1, Nombre = Juan, Edad = 30 }
Usuario { Id = 2, Nombre = Ana, Edad = 25 }
mayores de 26: 1
{"Id":1,"Nombre":"Juan","Edad":30}
```

!!! warning "Si ejecutas un archivo suelto: `#:property PublishAot=false`"
    La primera línea del ejemplo (`#:property PublishAot=false`) **solo hace falta al ejecutar un archivo suelto** con `dotnet run archivo.cs`. En .NET 10 esos archivos se preparan como si fueran a publicarse en modo AOT, que **prohíbe la serialización por reflexión**: sin esa línea, `JsonSerializer.Deserialize<T>` falla con *«Reflection-based serialization has been disabled»*. En un proyecto normal (`dotnet new console`) no hace falta.

Un `record` es una clase pensada para guardar datos: C# le añade solo la forma de mostrarse y de compararse. Sus propiedades se escriben con inicial mayúscula; `PropertyNameCaseInsensitive` hace que `nombre` del JSON encaje con `Nombre`.

## Trabajar con archivos y controlar los errores

Con un archivo, se lee con `File.ReadAllText` y se guarda con `File.WriteAllText`. Lo que puede fallar es **abrir** el archivo (no existe) y **leer** el JSON (está mal escrito), así que se capturan por separado:

```csharp
using System.Text.Json;
using System.Text.Json.Nodes;

void MostrarDatos(JsonNode datos)
{
    JsonArray usuarios = datos["usuarios"]!.AsArray();
    if (usuarios.Count == 0)
    {
        Console.WriteLine("No hay usuarios");
        return;
    }
    Console.WriteLine("--- Contenido Actual del JSON ---");
    foreach (JsonNode? u in usuarios)
    {
        Console.WriteLine($"ID: {u!["id"]}, Nombre: {u["nombre"]}, Edad: {u["edad"]}");
    }
    Console.WriteLine("--- Fin del Contenido ---");
}

JsonNode? Inicializar(string origen, string destino)
{
    try
    {
        string texto = File.ReadAllText(origen);
        JsonNode datos = JsonNode.Parse(texto)!;
        File.WriteAllText(destino, texto);
        Console.WriteLine($"Datos inicializados desde '{origen}' a '{destino}'");
        return datos;
    }
    catch (FileNotFoundException)
    {
        Console.WriteLine($"Error: el archivo '{origen}' no existe");
    }
    catch (JsonException e)
    {
        Console.WriteLine($"Error: '{origen}' no contiene un JSON válido ({e.GetType().Name})");
    }
    return null;
}

JsonNode? datos = Inicializar("datos_usuarios_orig.json", "datos_usuarios.json");
if (datos != null) MostrarDatos(datos);

Inicializar("no_existe.json", "salida.json");
Inicializar("roto.json", "salida.json");
```

Salida:

```text
Datos inicializados desde 'datos_usuarios_orig.json' a 'datos_usuarios.json'
--- Contenido Actual del JSON ---
ID: 1, Nombre: Juan, Edad: 30
ID: 2, Nombre: Ana, Edad: 25
--- Fin del Contenido ---
Error: el archivo 'no_existe.json' no existe
Error: 'roto.json' no contiene un JSON válido (JsonReaderException)
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| No controlar que el archivo no existe | Captura `FileNotFoundException` (o comprueba con `File.Exists`) |
| No controlar un JSON mal escrito | Captura `JsonException` (las excepciones concretas, como `JsonReaderException`, derivan de ella) |
| Comillas simples o una coma de más en el JSON | JSON solo admite comillas dobles y no admite la coma final |
| Tildes escritas como `\uXXXX` | Usa `JavaScriptEncoder.UnsafeRelaxedJsonEscaping` |
| Que el nombre de la propiedad no coincida | `PropertyNameCaseInsensitive = true` o `[JsonPropertyName("nombre")]` |

## Para practicar

Haz el [ejercicio 3.4 de JSON](json.md).
