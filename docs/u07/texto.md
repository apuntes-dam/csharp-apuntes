# U7.3 · Ficheros de texto

<div class="ej-gate" data-unit="u07" data-nombre="U7 · Entrada/salida y GUI"></div>

Leer y escribir texto: secuencial, línea a línea, y siempre cerrando el archivo y controlando los errores.

## Ejercicio 7.7

**Contador.** Dado un archivo de texto, muestra su número de líneas, de palabras y de caracteres.

!!! note "En C#"
    Usa `File.ReadAllLines(ruta)` o `StreamReader` con `using`; para leer por bloques, mira [7.E](t-path.md).

<details class="sol" data-key="u69/u7-3/7.7">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("Archivo: ");
string ruta = (Console.ReadLine() ?? "").Trim();
if (!File.Exists(ruta))
{
    Console.WriteLine($"El archivo \"{ruta}\" no existe");
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.8

**Copia en mayúsculas.** Lee un archivo línea a línea y crea otro con el mismo contenido en mayúsculas. Si el archivo de destino ya existe, pregunta antes de sobrescribirlo.

<details class="sol" data-key="u69/u7-3/7.8">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("Origen: ");
string origen = (Console.ReadLine() ?? "").Trim();
Console.Write("Destino: ");
string destino = (Console.ReadLine() ?? "").Trim();
if (!File.Exists(origen))
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.9

**Agenda de contactos persistente.** Guarda contactos en un archivo de texto, una línea por contacto con el formato `nombre;teléfono`. Menú: añadir, buscar por nombre, listar y salir. Al arrancar, carga el archivo (si no existe, empieza vacío); al añadir, guarda. Ignora las líneas mal formadas mostrando un aviso.

## Ejercicio 7.10

**Registro (log).** Escribe una función `registrar(mensaje)` que **añada** al final de un archivo una línea con la fecha y hora (`AAAA-MM-DD HH:MM:SS`) y el mensaje, sin borrar lo anterior. Llámala desde un programa pequeño varias veces y comprueba el archivo.

!!! note "En C#"
    `DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss", CultureInfo.InvariantCulture)` da el formato pedido. Para añadir al final del archivo, `File.AppendAllText`.

## Ejercicio 7.11

**Notas desde CSV.** Dado un archivo `notas.csv` con líneas `alumno;nota1;nota2;nota3`, calcula la nota media de cada alumno y la media de la clase, y escribe el resultado en `medias.txt`. Controla las notas que no sean números.
