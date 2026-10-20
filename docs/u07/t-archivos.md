# 7.B Archivos y carpetas

Los programas guardan datos en **archivos**, organizados en **carpetas** (directorios). Saber consultar, crear, copiar, mover y borrar es la base para cualquier programa que trabaje con datos que deben sobrevivir al cierre.

## Rutas

Una **ruta** indica dónde está un archivo o carpeta.

| Tipo | Ejemplo | Significa |
|---|---|---|
| **Absoluta** | `C:/Users/ana/datos/notas.txt` (Windows) · `/home/ana/datos/notas.txt` (Linux, macOS) | Desde la raíz del sistema |
| **Relativa** | `datos/notas.txt` | Desde la **carpeta de trabajo** del programa (la carpeta desde la que se ejecuta) |

!!! tip "Une rutas con `Path.Combine`"
    Windows escribe las rutas con `\`, y Linux y macOS con `/`. En vez de pegar las piezas con `+`, usa **`Path.Combine("datos", "notas.txt")`**, que pone el separador correcto en cada sistema. En un literal de texto, la `\` es el carácter de escape: o la duplicas (`"C:\\datos"`) o usas un texto literal (`@"C:\datos"`). La barra normal `/` también funciona en Windows.

!!! tip "Qué excepción salta y qué no"
    Aquí se usan las operaciones de `File`, `Directory` y `Path`. En [7.E](t-path.md) están las **excepciones concretas** de cada una (`FileNotFoundException`, `DirectoryNotFoundException`...) y las operaciones que **no** dan error aunque no haya nada que hacer, como `File.Delete` sobre un archivo que no existe.

## Las operaciones básicas

Se usan las clases de **`System.IO`**: **`File`** (operaciones sobre archivos), **`Directory`** (sobre carpetas) y **`Path`** (para trabajar con el texto de las rutas). Todas tienen métodos **estáticos**: no se crea ningún objeto. Para consultar datos de un archivo concreto (tamaño, fecha) existe también **`FileInfo`**, y **`DirectoryInfo`** para carpetas. Las operaciones fallidas lanzan una **`IOException`** o una subclase suya.

```csharp
string carpeta = "datos";
Directory.CreateDirectory(Path.Combine(carpeta, "sub"));
Console.WriteLine($"carpeta creada: {(Directory.Exists(carpeta) ? "sí" : "no")}");

File.WriteAllText(Path.Combine(carpeta, "notas.txt"), "uno\ndos\n");
foreach (string nombre in Nombres(carpeta))
{
    string ruta = Path.Combine(carpeta, nombre);
    if (Directory.Exists(ruta))
    {
        Console.WriteLine($"{nombre}: carpeta");
    }
    else
    {
        Console.WriteLine($"{nombre}: archivo de {new FileInfo(ruta).Length} bytes");
    }
}

File.Copy(Path.Combine(carpeta, "notas.txt"), Path.Combine(carpeta, "notas_copia.txt"));
Console.WriteLine($"tras copiar: {string.Join(", ", Nombres(carpeta))}");
File.Move(Path.Combine(carpeta, "notas_copia.txt"), Path.Combine(carpeta, "resumen.txt"));
Console.WriteLine($"tras renombrar: {string.Join(", ", Nombres(carpeta))}");

Directory.Delete(carpeta, recursive: true);
Console.WriteLine($"datos existe tras borrar: {(Directory.Exists(carpeta) ? "sí" : "no")}");

List<string> Nombres(string carpeta)
{
    var lista = Directory.EnumerateFileSystemEntries(carpeta).Select(p => Path.GetFileName(p)).ToList();
    lista.Sort(StringComparer.Ordinal);
    return lista;
}
```

Salida:

```text
carpeta creada: sí
notas.txt: archivo de 8 bytes
sub: carpeta
tras copiar: notas.txt, notas_copia.txt, sub
tras renombrar: notas.txt, resumen.txt, sub
datos existe tras borrar: no
```

El programa crea una carpeta con una subcarpeta, escribe un archivo, **inspecciona** lo que hay (¿archivo o carpeta? ¿cuánto ocupa?), lo copia, lo renombra y lo borra todo al final. Fíjate en que la lista de nombres se **ordena** antes de mostrarla: el sistema no garantiza ningún orden al listar una carpeta.

| Necesito... | En C# |
|---|---|
| ¿Existe? ¿Es carpeta? | `File.Exists(r)` · `Directory.Exists(r)` |
| Tamaño en bytes | `new FileInfo(r).Length` |
| Crear carpeta (con las que falten) | `Directory.CreateDirectory(r)` |
| Listar el contenido | `Directory.EnumerateFileSystemEntries(r)` (o `GetFiles` y `GetDirectories`) |
| Copiar · renombrar o mover | `File.Copy(a, b)` · `File.Move(a, b)` |
| Borrar archivo · carpeta con todo lo que contiene | `File.Delete(r)` · `Directory.Delete(r, recursive: true)` |
| Nombre, extensión y carpeta de una ruta | `Path.GetFileName(r)` · `Path.GetExtension(r)` · `Path.GetDirectoryName(r)` |
| Ruta absoluta | `Path.GetFullPath(r)` |

## Cuando algo falla

Un archivo que no existe lanza **`FileNotFoundException`**; una carpeta que no existe, **`DirectoryNotFoundException`**; la falta de permisos, **`UnauthorizedAccessException`**; y la mayoría de los demás problemas (copiar donde ya hay un archivo, borrar una carpeta con contenido), una **`IOException`** general. Las dos primeras son subclases de `IOException`.

Los fallos son **normales** con archivos (el usuario escribe mal una ruta, el disco está lleno, otro programa tiene el archivo abierto). Un programa robusto **comprueba antes** lo que pueda (¿existe?) y **captura** lo demás para dar un mensaje claro.

!!! danger "Cuidado con lo que borras"
    `Directory.Delete(ruta, recursive: true)` elimina **todo** lo que haya dentro, sin papelera y sin vuelta atrás. Antes de borrar:

    * Comprueba que la ruta es **la que esperas** (no vacía, no la raíz de un disco, no tu carpeta de usuario).
    * Si el borrado lo decide el usuario, **pídele confirmación**.
    * Mientras pruebas, usa una carpeta de prueba aparte.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Rutas relativas que dependen de dónde se ejecute el programa | Comprobar la carpeta de trabajo (`Directory.GetCurrentDirectory()`), o usar rutas absolutas |
| Asumir un orden al listar una carpeta | Ordenar los nombres |
| Sobrescribir un archivo existente sin avisar | Comprobar si existe y preguntar antes |
| Borrar una carpeta no vacía sin `recursive` | Usar la versión recursiva, con confirmación |
| Pegar rutas con `+` y una barra escrita a mano | `Path.Combine` |
| Escribir `"C:\datos"` sin duplicar la barra | `"C:\\datos"`, `@"C:\datos"` o `/` |

## Para practicar

Haz los ejercicios de [U7.2 · Archivos y carpetas](archivos.md). Y para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
