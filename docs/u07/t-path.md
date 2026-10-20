# 7.E Ficheros con Path, bytes y bloques

En [7.B](t-archivos.md) y [7.C](t-texto.md) trabajaste con `File`, `Directory` y métodos cortos como `File.ReadAllLines`. Aquí se completa el cuadro con lo que suele pedirse en el módulo de Acceso a Datos:

* La clase **`Path`** y las operaciones de **`File`** y **`Directory`**, con las **excepciones concretas** de cada una.
* Los **ficheros binarios**: se guardan **bytes**, no letras.
* **Leer y escribir por bloques**: en vez de cargar todo el archivo de golpe, se trabaja con un trozo cada vez.

Todos los ejemplos de esta página se han **ejecutado** con .NET 10 en una carpeta vacía, en Windows.

## Path, File y Directory: operaciones y errores

En C#, una ruta es un simple **`string`**; la clase estática **`Path`** solo trabaja con ese texto (unir, separar, sacar el nombre) y **no toca el disco**. Que una ruta exista como texto no significa que el archivo exista. Las operaciones sobre el disco están en **`File`** y **`Directory`** (estáticas) y, si quieres un objeto con los datos de un archivo, en **`FileInfo`** y **`DirectoryInfo`**.

| Quiero… | Se escribe |
|---|---|
| Unir piezas de una ruta | `Path.Combine("datos", "nota.txt")` |
| Nombre y carpeta padre | `Path.GetFileName(ruta)` · `Path.GetDirectoryName(ruta)` (puede ser `null`) |
| Ruta absoluta | `Path.GetFullPath(ruta)` |
| Saber si existe | `File.Exists(ruta)` · `Directory.Exists(ruta)` |
| Saber si es archivo o carpeta | `File.Exists(ruta)` · `Directory.Exists(ruta)` (cada una dice solo lo suyo) |
| Tamaño en bytes | `new FileInfo(ruta).Length` (un `long`) |
| Crear una carpeta (y las que falten) | `Directory.CreateDirectory(ruta)` |
| Crear un archivo vacío | `File.WriteAllBytes(ruta, [])` (o `File.Create(ruta).Dispose()`) |
| Copiar | `File.Copy(origen, destino)` |
| Mover o cambiar de nombre | `File.Move(origen, destino)` |
| Borrar un archivo | `File.Delete(ruta)` |
| Borrar una carpeta | `Directory.Delete(ruta)` (vacía) · `Directory.Delete(ruta, recursive: true)` |

!!! info "No hay clase `Files` ni excepciones separadas para cada caso"
    En C# no existe `FileAlreadyExistsException` ni `DirectoryNotEmptyException`: los dos casos son una **`IOException`** general. Si necesitas distinguirlos, **comprueba antes** (`File.Exists`, o si la carpeta tiene contenido con `Directory.EnumerateFileSystemEntries(ruta).Any()`) y lanza o muestra tu propio mensaje.

Un recorrido completo, con `try`, `catch` y un `finally` que limpia lo que se haya creado:

```csharp
string carpeta = "datos";
string archivo = Path.Combine(carpeta, "nota.txt");
try
{
    Directory.CreateDirectory(carpeta);
    File.WriteAllText(archivo, "hola");
    Console.WriteLine($"{Path.GetFileName(archivo)} está en {Path.GetDirectoryName(archivo)}");
    Console.WriteLine($"tamaño: {new FileInfo(archivo).Length} bytes");

    string copia = Path.Combine(carpeta, "copia.txt");
    File.Copy(archivo, copia);
    string final = Path.Combine(carpeta, "final.txt");
    File.Move(copia, final);
    Console.WriteLine($"final.txt existe: {File.Exists(final)}");
}
catch (IOException e)
{
    Console.WriteLine($"Error de archivo: {e.Message}");
}
finally
{
    try
    {
        if (Directory.Exists(carpeta))
        {
            Directory.Delete(carpeta, recursive: true);
        }
    }
    catch (IOException e)
    {
        Console.WriteLine($"No se pudo limpiar: {e.Message}");
    }
}
Console.WriteLine($"quedó la carpeta: {Directory.Exists(carpeta)}");
```

Salida:

```text
nota.txt está en datos
tamaño: 4 bytes
final.txt existe: True
quedó la carpeta: False
```

El `finally` se ejecuta **siempre**, haya habido error o no. Por eso es el sitio para borrar lo que el programa creó (y como borrar también puede fallar, lleva su propio `try`). Con `Path.GetDirectoryName` sobre una ruta sin carpeta (`"nota.txt"`) se obtiene la cadena vacía, y sobre la raíz (`"C:/"`), `null`.

### Qué pasa cuando algo falla

Mira cuáles de estas operaciones fallan y con qué excepción (en cada línea se muestra el tipo de la excepción que ha saltado, o «sin error»):

```csharp
Directory.CreateDirectory("vacia");
Directory.CreateDirectory("llena");
File.WriteAllText(Path.Combine("llena", "x.txt"), "x");
File.WriteAllText("a.txt", "a");
File.WriteAllText("b.txt", "b");

Probar("copiar un archivo que no existe", () => File.Copy("falta.txt", "z.txt"));
Probar("copiar sobre uno que ya existe", () => File.Copy("a.txt", "b.txt"));
Probar("copiar sobre uno que ya existe con overwrite", () => File.Copy("a.txt", "b.txt", overwrite: true));
Probar("mover sobre uno que ya existe", () => File.Move("a.txt", "b.txt"));
Probar("leer en una carpeta que no existe", () => File.ReadAllText(Path.Combine("nada", "x.txt")));
Probar("borrar una carpeta con contenido", () => Directory.Delete("llena"));
Probar("borrar una carpeta vacía", () => Directory.Delete("vacia"));
Probar("borrar un archivo que no existe", () => File.Delete("falta.txt"));
Probar("crear una carpeta que ya existe", () => Directory.CreateDirectory("llena"));
Probar("leer una carpeta como si fuera un archivo", () => File.ReadAllText("llena"));
Probar("tamaño de un archivo que no existe", () => Console.WriteLine(new FileInfo("falta.txt").Length));

Console.WriteLine();
Console.WriteLine($"FileNotFoundException es IOException: {new FileNotFoundException() is IOException}");
Console.WriteLine($"DirectoryNotFoundException es IOException: {new DirectoryNotFoundException() is IOException}");
Console.WriteLine($"UnauthorizedAccessException es IOException: {new UnauthorizedAccessException() is IOException}");

void Probar(string que, Action accion)
{
    try
    {
        accion();
        Console.WriteLine($"{que}: sin error");
    }
    catch (Exception e)
    {
        Console.WriteLine($"{que}: {e.GetType().Name}");
    }
}
```

Salida:

```text
copiar un archivo que no existe: FileNotFoundException
copiar sobre uno que ya existe: IOException
copiar sobre uno que ya existe con overwrite: sin error
mover sobre uno que ya existe: IOException
leer en una carpeta que no existe: DirectoryNotFoundException
borrar una carpeta con contenido: IOException
borrar una carpeta vacía: sin error
borrar un archivo que no existe: sin error
crear una carpeta que ya existe: sin error
leer una carpeta como si fuera un archivo: UnauthorizedAccessException
tamaño de un archivo que no existe: FileNotFoundException

FileNotFoundException es IOException: True
DirectoryNotFoundException es IOException: True
UnauthorizedAccessException es IOException: False
```

| Excepción | Cuándo se produce |
|---|---|
| `FileNotFoundException` | El archivo no existe: `File.Copy`, `File.Move` o leerlo. Es una `IOException` |
| `DirectoryNotFoundException` | Falta la **carpeta** de la ruta: leer o escribir en `nada/x.txt`. Es una `IOException` |
| `IOException` | La general: copiar o mover sobre un archivo que ya existe, borrar una carpeta con contenido... |
| `UnauthorizedAccessException` | Sin permisos, o la ruta es una carpeta y se trata como archivo. **No** es una `IOException` |

Se capturan de la más concreta a la más general. Con `e.Message` se sabe qué ruta falló, y `e.GetType().Name` dice cuál ha sido la excepción.

!!! info "Cuándo hay error y cuándo no"
    `File.Copy` y `File.Move` **no pisan** un archivo que ya existe: lanzan `IOException`, salvo que pases `overwrite: true`. En cambio, `File.Delete` sobre un archivo que no existe **no falla**, y `Directory.CreateDirectory` sobre una carpeta que ya existe tampoco: simplemente no hacen nada.

## Ficheros binarios

En un fichero binario cada elemento es un **byte**. Se abre con un **`FileStream`** y un modo (`FileMode.Create` para escribir desde cero, `FileMode.Open` para leer), siempre dentro de un **`using`**, que cierra el archivo al terminar aunque haya un error.

* `salida.WriteByte(numero)` escribe **un** byte.
* `entrada.ReadByte()` lee un byte y lo devuelve como un `int` de 0 a 255. Cuando no queda nada, devuelve **`-1`**.

```csharp
using (var salida = new FileStream("datos.bin", FileMode.Create))
{
    foreach (int numero in new[] { 72, 111, 108, 97, 200 })
    {
        salida.WriteByte((byte)numero);
    }
}
Console.WriteLine($"tamaño: {new FileInfo("datos.bin").Length} bytes");

var leidos = new List<int>();
using (var entrada = new FileStream("datos.bin", FileMode.Open))
{
    int b;
    while ((b = entrada.ReadByte()) != -1)
    {
        leidos.Add(b);
    }
}
Console.WriteLine($"leídos: {string.Join(" ", leidos)}");
File.Delete("datos.bin");
```

Salida:

```text
tamaño: 5 bytes
leídos: 72 111 108 97 200
```

Para archivos pequeños hay atajos: `File.WriteAllBytes(ruta, bytes)` y `File.ReadAllBytes(ruta)`.

!!! info "El tipo `byte` de C# va de 0 a 255"
    A diferencia de Java, el `byte` de C# **no tiene signo**: va de 0 a 255, y 200 se ve como `200`. Si lo que quieres es un byte con signo, el tipo es `sbyte`. Además, C# te protege del desbordamiento: `salida.WriteByte(300)` **no compila** (un literal fuera de rango), y convertir una variable con `(byte)` solo **recorta** los bits sobrantes si no está activada la comprobación (`unchecked`, lo normal); con `checked` salta una `OverflowException`.

```csharp
byte b = 200;
Console.WriteLine($"un byte de C# vale: {b}");

int grande = 300;
Console.WriteLine($"(byte)300 sin comprobar: {unchecked((byte)grande)}");
try
{
    Console.WriteLine(checked((byte)grande));
}
catch (OverflowException)
{
    Console.WriteLine("(byte)300 comprobado: OverflowException");
}

File.WriteAllBytes("c.bin", new byte[] { 255, 0, 128 });
byte[] datos = File.ReadAllBytes("c.bin");
Console.WriteLine($"ReadAllBytes: {string.Join(" ", datos)}");
File.Delete("c.bin");
```

Salida:

```text
un byte de C# vale: 200
(byte)300 sin comprobar: 44
(byte)300 comprobado: OverflowException
ReadAllBytes: 255 0 128
```

### Leer y escribir por bloques

Leer byte a byte es lento con archivos grandes. Lo habitual es usar un **buffer** (un `byte[]`) y leer un bloque de golpe:

* `entrada.Read(buffer, 0, buffer.Length)` rellena el buffer y **devuelve cuántos bytes ha leído**, o **`0`** si ya no queda nada (no `-1`, como `ReadByte`).
* `salida.Write(buffer, 0, leidos)` escribe **solo** los `leidos` primeros bytes del buffer.

```csharp
File.WriteAllBytes("origen.bin", Enumerable.Range(1, 25).Select(n => (byte)n).ToArray());

byte[] buffer = new byte[10];
int bloques = 0;
long total = 0;
using (var entrada = new FileStream("origen.bin", FileMode.Open))
using (var salida = new FileStream("copia.bin", FileMode.Create))
{
    int leidos;
    while ((leidos = entrada.Read(buffer, 0, buffer.Length)) > 0)
    {
        salida.Write(buffer, 0, leidos);
        bloques++;
        total += leidos;
        Console.WriteLine($"bloque {bloques}: {leidos} bytes");
    }
}
Console.WriteLine($"{bloques} bloques, {total} bytes");
Console.WriteLine($"origen: {new FileInfo("origen.bin").Length}, copia: {new FileInfo("copia.bin").Length}");
File.Delete("origen.bin");
File.Delete("copia.bin");
```

Salida:

```text
bloque 1: 10 bytes
bloque 2: 10 bytes
bloque 3: 5 bytes
3 bloques, 25 bytes
origen: 25, copia: 25
```

El último bloque trae solo 5 bytes, aunque el buffer sea de 10. Si se escribe el buffer entero, se copia basura:

```csharp
File.WriteAllBytes("origen.bin", new byte[25]);

byte[] buffer = new byte[10];
using (var entrada = new FileStream("origen.bin", FileMode.Open))
using (var salida = new FileStream("mala.bin", FileMode.Create))
{
    while (entrada.Read(buffer, 0, buffer.Length) > 0)
    {
        salida.Write(buffer);      // escribe el buffer ENTERO, vengan 10 bytes o 5
    }
}
Console.WriteLine($"origen: {new FileInfo("origen.bin").Length} bytes, copia: {new FileInfo("mala.bin").Length} bytes");
File.Delete("origen.bin");
File.Delete("mala.bin");
```

Salida:

```text
origen: 25 bytes, copia: 30 bytes
```

!!! warning "Escribe solo lo que has leído"
    `salida.Write(buffer)` escribe **todo** el buffer. En el último bloque conserva los bytes del bloque anterior, así que la copia sale más grande y estropeada. Usa siempre `Write(buffer, 0, leidos)`.

## Ficheros de texto: caracteres, bytes y bloques

Un archivo de texto también son bytes. Para leerlo como letras hay que decir cómo se codifican, y por eso **caracteres y bytes no coinciden**: en UTF-8 las letras sin tilde ocupan 1 byte, pero la `ñ` y las vocales con tilde ocupan 2 y el `€`, 3. Además, `Length` en C# cuenta unidades de 16 bits (UTF-16), no «letras»: un emoji cuenta como 2.

```csharp
foreach (string texto in new[] { "hola", "Año 5 €", "ok 😀" })
{
    File.WriteAllText("t.txt", texto);
    Console.WriteLine($"{texto,-8} Length: {texto.Length,2}  caracteres reales: {texto.EnumerateRunes().Count(),2}  bytes en disco: {new FileInfo("t.txt").Length,2}");
}
File.Delete("t.txt");
```

Salida:

```text
hola     Length:  4  caracteres reales:  4  bytes en disco:  4
Año 5 €  Length:  7  caracteres reales:  7  bytes en disco: 10
ok 😀    Length:  5  caracteres reales:  4  bytes en disco:  7
```

`EnumerateRunes().Count()` cuenta los caracteres Unicode de verdad. Para saber cuánto ocupa en el disco hay que mirar `FileInfo.Length` o `Encoding.UTF8.GetByteCount(texto)`.

### UTF-8 con y sin BOM

En .NET moderno, `File.WriteAllText(ruta, texto)` ya escribe en **UTF-8 sin BOM**, que es lo habitual. Pero si le pasas **`Encoding.UTF8`**, escribe además una marca BOM de 3 bytes al principio del archivo. Compruébalo:

```csharp
using System.Text;

File.WriteAllText("sin.txt", "hola");
File.WriteAllText("con.txt", "hola", Encoding.UTF8);
Console.WriteLine($"por defecto: {new FileInfo("sin.txt").Length} bytes");
Console.WriteLine($"con Encoding.UTF8: {new FileInfo("con.txt").Length} bytes");
File.Delete("sin.txt");
File.Delete("con.txt");
```

Salida:

```text
por defecto: 4 bytes
con Encoding.UTF8: 7 bytes
```

Esos 3 bytes de más son la marca (`EF BB BF`), y algunos programas la ven como tres caracteres extraños al principio. Si quieres indicar la codificación sin que se añada la marca, usa `new UTF8Encoding(false)`. Al **leer**, `StreamReader` detecta el BOM y lo descarta solo.

Para leer y escribir **caracteres** se usan `StreamReader` y `StreamWriter`, con un buffer `char[]`; el patrón es el mismo que con los bytes:

* `lector.Read(buffer, 0, buffer.Length)` devuelve cuántos caracteres ha leído, o `0`.
* `new string(buffer, 0, leidos)` convierte en texto solo la parte leída.

```csharp
string contenido = "Hola, mundo.\nSegunda\nFin\n";
File.WriteAllText("leer.txt", contenido);

char[] buffer = new char[16];
int bloques = 0;
using (var lector = new StreamReader("leer.txt"))
{
    int leidos;
    while ((leidos = lector.Read(buffer, 0, buffer.Length)) > 0)
    {
        bloques++;
        string trozo = new string(buffer, 0, leidos).Replace("\n", "\\n");
        Console.WriteLine($"bloque {bloques} ({leidos} caracteres): {trozo}");
    }
}
File.Delete("leer.txt");
```

Salida:

```text
bloque 1 (16 caracteres): Hola, mundo.\nSeg
bloque 2 (9 caracteres): unda\nFin\n
```

El error clásico es el mismo que con los bytes: usar todo el buffer en el último bloque.

```csharp
File.WriteAllText("leer.txt", "Hola, mundo.\nSegunda\nFin\n");

char[] buffer = new char[16];
using (var lector = new StreamReader("leer.txt"))
{
    while (lector.Read(buffer, 0, buffer.Length) > 0)
    {
        string trozo = new string(buffer).Replace("\n", "\\n");      // ¡todo el buffer!
        Console.WriteLine($"[{trozo}]");
    }
}
File.Delete("leer.txt");
```

Salida:

```text
[Hola, mundo.\nSeg]
[unda\nFin\ndo.\nSeg]
```

El último bloque solo tenía 9 caracteres y los otros 7 del buffer conservaban lo del bloque anterior. Para escribir por bloques desde un `char[]` se usa `escritor.Write(caracteres, inicio, longitud)`:

```csharp
string texto = "Programar en C# es divertido";
char[] caracteres = texto.ToCharArray();
int tamano = 10;
int escrituras = 0;

using (var escritor = new StreamWriter("salida.txt"))
{
    for (int inicio = 0; inicio < caracteres.Length; inicio += tamano)
    {
        int longitud = Math.Min(tamano, caracteres.Length - inicio);
        escritor.Write(caracteres, inicio, longitud);
        escrituras++;
    }
}
Console.WriteLine($"{escrituras} escrituras");
Console.WriteLine($"contenido: {File.ReadAllText("salida.txt")}");
File.Delete("salida.txt");
```

Salida:

```text
3 escrituras
contenido: Programar en C# es divertido
```

!!! note "Y si solo quieres las líneas"
    Para recorrer un archivo línea a línea no hace falta un buffer: `File.ReadAllLines(ruta)` o `File.ReadLines(ruta)` (ver [7.C](t-texto.md)). Los bloques se usan cuando el archivo es grande o cuando te piden leer un número fijo de caracteres.

## Errores frecuentes

* **Olvidar el `using`**: el archivo se queda abierto y puede no guardarse todo lo escrito.
* **No comprobar el final**: `ReadByte` devuelve `-1`; `Read(buffer...)`, `0`. Sin comprobarlo, el bucle no termina o procesa datos que no existen.
* **Escribir el buffer entero** en vez de `Write(buffer, 0, leidos)` (o `new string(buffer)` en vez de `new string(buffer, 0, leidos)`).
* **Esperar que `Copy` o `Move` sobrescriban**: lanzan `IOException` salvo que indiques `overwrite: true`.
* **Buscar excepciones que C# no tiene**: no hay `FileAlreadyExistsException`; comprueba antes con `File.Exists`.
* **Creer que `Path` toca el disco**: solo manipula texto; para consultar o cambiar el disco están `File` y `Directory`.

## Para practicar

Los ejercicios [7.16 a 7.21](path.md) usan todo lo de esta página.
