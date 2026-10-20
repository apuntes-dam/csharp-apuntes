# 7.C Ficheros de texto

Un **fichero de texto** guarda caracteres legibles (notas, configuraciones, registros, CSV, JSON...). Se puede abrir con cualquier editor, a diferencia de un fichero **binario** (una imagen, un ejecutable). Leer y escribir texto es lo más habitual al trabajar con datos.

!!! tip "Leer y escribir por bloques"
    Aquí se lee y escribe el archivo entero o línea a línea. Para trabajar **por bloques** (bytes y trozos de texto) y con ficheros **binarios** mira [7.E](t-path.md).

## Crear, añadir y leer

`File.WriteAllText(ruta, texto)` **crea o sobrescribe** el archivo; `File.AppendAllText` **añade** al final. `File.ReadAllLines` devuelve un array con las líneas y `File.ReadAllText`, todo el contenido de golpe. Estos métodos abren y **cierran** el archivo por ti. Para recorrer un archivo grande sin cargarlo entero existe `File.ReadLines`, que entrega las líneas **una a una** según las pides. Para trabajar **línea a línea** con control total se usan `StreamReader` y `StreamWriter` dentro de un **`using`**, que **cierra** el archivo al terminar aunque haya un error. La codificación por defecto es **UTF-8** (sin marca BOM).

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

Qué hace el programa, paso a paso:

1. **Escribe** dos líneas y después **añade** una tercera (sin borrar las anteriores).
2. **Lee** el archivo completo como array de líneas y las muestra numeradas, contando las palabras.
3. **Intenta abrir un archivo que no existe** y avisa con un mensaje claro en lugar de dejar que el programa se detenga: salta una `FileNotFoundException`.
4. **Copia** el archivo en mayúsculas leyendo **línea a línea**, y comprueba el resultado.
5. **Borra** los archivos de prueba.

## Tres decisiones al abrir un archivo

| Decisión | Opciones |
|---|---|
| **¿Qué hago con lo que ya hay?** | **Sobrescribir** (el contenido anterior se pierde) o **añadir** al final |
| **¿Cómo lo leo?** | **Entero** de una vez (cómodo, solo para ficheros pequeños) o **línea a línea** (cualquier tamaño, gasta poca memoria) |
| **¿Con qué codificación?** | **UTF-8**, que sirve para tildes, eñes y símbolos. Si el archivo se guardó con otra, las tildes salen mal |

!!! warning "Escribir borra"
    La operación de «escribir» **vacía** el archivo si ya existía. Si no quieres perder su contenido, **añade** en lugar de escribir, o pregunta antes de sobrescribir (como piden los ejercicios).

## Cerrar siempre el archivo

Un archivo abierto ocupa recursos del sistema y puede quedar **a medias** (con datos aún sin escribir en el disco) si no se cierra. Por eso se abre dentro de una estructura que lo **cierra automáticamente**, incluso si hay un error en medio. En C#, es la sentencia **`using`**: con llaves, como en el ejemplo (se cierra al salir del bloque), o en su forma corta `using var entrada = new StreamReader(archivo);` (se cierra al salir del método o del programa). Las funciones de una sola llamada (`File.ReadAllText`, `File.WriteAllText`...) ya lo hacen por dentro.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Perder el contenido anterior al escribir | Añadir (`AppendAllText`) o preguntar antes de sobrescribir |
| Cargar en memoria un archivo enorme | `File.ReadLines` o `StreamReader`, línea a línea |
| Tildes que se ven mal (`Ã¡` en vez de `á`) | Indicar la codificación al abrir (`new StreamReader(ruta, Encoding.UTF8)`) |
| No cerrar el archivo | `using` |
| Dar por hecho que el archivo existe | Capturar `FileNotFoundException` o comprobar con `File.Exists` |
| Contar las líneas sin tener en cuenta la última línea vacía | Probar con archivos que terminen y no terminen en salto de línea |

## Para practicar

Haz los ejercicios de [U7.3 · Ficheros de texto](texto.md). [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) muestra cómo se escribe lo mismo en otro lenguaje.
