# 6.D Comentarios y documentación

Hay dos formas distintas de explicar el código, y conviene no mezclarlas:

* **Comentarios** (`//`): notas **para quien lea el código**, que explican cosas puntuales del interior.
* **Documentación** (comentarios especiales `///`): descripción **de las clases y métodos públicos para quien los vaya a usar**, sin necesidad de leer su código. El editor la muestra al usar el método y una herramienta la convierte en páginas web.

## Un buen comentario explica el porqué

El código ya dice **qué** hace; un comentario solo aporta valor si cuenta lo que el código **no** puede decir: **por qué** se hizo así.

| Comentario | Valoración |
|---|---|
| `i = i + 1;  // suma 1 a i` | **Sobra**: repite el código |
| `// bucle for` | **Sobra**: se ve a simple vista |
| `// la API devuelve las fechas en UTC: hay que convertir antes de comparar` | **Útil**: explica una razón que no se ve |
| `// TODO: cambiar cuando exista el servicio de pagos` | **Útil**, si se revisa y se acaba borrando |
| `// precio = precio * 1.1;` (código desactivado) | **Sobra**: bórralo; el control de versiones ya guarda el historial |

!!! tip "Antes de comentar, mejora el nombre"
    Si necesitas un comentario para explicar qué es `p`, quizá lo que falta es llamarla `precioConIva`. Un código con buenos nombres y métodos cortos se explica casi solo, y los comentarios se reservan para el **porqué**.

## Documentar clases y métodos

Una buena documentación de una clase o método público responde a cuatro preguntas: **qué hace**, **qué recibe**, **qué devuelve** y **qué errores puede dar**; y, si ayuda, un **ejemplo de uso**.

```csharp
var conversor = new Conversor();
foreach (int c in new[] { 100, 0, -40 })
{
    Console.WriteLine($"{c} °C = {conversor.CelsiusAFahrenheit(c)} °F");
}
try
{
    conversor.CelsiusAFahrenheit(-300);
}
catch (ArgumentOutOfRangeException e)
{
    Console.WriteLine($"Error: {e.ParamName}: {e.Message.Split('\n')[0]}");
}

/// <summary>
/// Convierte temperaturas entre grados Celsius y Fahrenheit.
/// </summary>
/// <example>
/// <code>new Conversor().CelsiusAFahrenheit(100)  // 212</code>
/// </example>
public class Conversor
{
    /// <summary>
    /// Devuelve la temperatura en grados Fahrenheit equivalente a <paramref name="celsius"/>.
    /// </summary>
    /// <param name="celsius">Temperatura en grados Celsius.</param>
    /// <returns>La temperatura en grados Fahrenheit.</returns>
    /// <exception cref="ArgumentOutOfRangeException">Si <paramref name="celsius"/> está por debajo del cero absoluto (-273 °C).</exception>
    public int CelsiusAFahrenheit(int celsius)
    {
        if (celsius < -273)
        {
            throw new ArgumentOutOfRangeException(nameof(celsius), "por debajo del cero absoluto");
        }
        return celsius * 9 / 5 + 32;
    }
}
```

Salida:

```text
100 °C = 212 °F
0 °C = 32 °F
-40 °C = -40 °F
Error: celsius: por debajo del cero absoluto (Parameter 'celsius')
```

## Cómo se hace en C#

| Necesito... | En C# |
|---|---|
| Comentario de documentación | `///` delante del elemento, con etiquetas **XML** |
| Etiquetas | `<summary>`, `<param name="x">`, `<returns>`, `<exception cref="Tipo">`, `<example>`, `<remarks>` |
| Referenciar otro elemento | `<see cref="Clase"/>` y, para un parámetro, `<paramref name="celsius"/>` |
| Código dentro del texto | `<c>...</c>` (en línea) o `<code>...</code>` (bloque) |
| Que el editor lo muestre | Es automático: al escribir `conversor.CelsiusAFahrenheit(` el IDE enseña el `<summary>` y los `<param>` (IntelliSense) |
| Generar el archivo de documentación | En el `.csproj`: `<GenerateDocumentationFile>true</GenerateDocumentationFile>` |
| Convertirlo en una web HTML | Con **DocFX** (herramienta de .NET) |

Al compilar con `GenerateDocumentationFile` el compilador escribe un archivo `.xml` con todos los comentarios. Esto es lo que genera `dotnet build` para el ejemplo anterior (fragmento comprobado):

```xml
<member name="T:Conversor">
    <summary>
    Convierte temperaturas entre grados Celsius y Fahrenheit.
    </summary>
    <example>
    <code>new Conversor().CelsiusAFahrenheit(100)  // 212</code>
    </example>
</member>
<member name="M:Conversor.CelsiusAFahrenheit(System.Int32)">
    <summary>
    Devuelve la temperatura en grados Fahrenheit equivalente a <paramref name="celsius"/>.
    </summary>
    <param name="celsius">Temperatura en grados Celsius.</param>
    <returns>La temperatura en grados Fahrenheit.</returns>
    <exception cref="T:System.ArgumentOutOfRangeException">Si <paramref name="celsius"/> está por debajo del cero absoluto (-273 °C).</exception>
</member>
```

El código y la generación del XML están comprobados, pero **no he generado el sitio HTML con DocFX** (necesita instalar la herramienta y configurar un proyecto de documentación), así que sigue las instrucciones de su documentación oficial.

!!! note "Si se activa la documentación, avisa de lo que falta"
    Con `GenerateDocumentationFile` el compilador emite la advertencia `CS1591` por cada elemento **público** sin documentar. Es una forma cómoda de ver qué te falta.

## Qué documentar

* **Sí:** las clases y métodos **públicos**: lo que usarán otras personas (o tú mismo dentro de un mes).
* **Sí:** las excepciones que pueden lanzar y las **condiciones** que deben cumplir los datos de entrada.
* **No:** lo evidente (`/// Devuelve el nombre.` sobre `GetNombre()`). Una documentación vacía es ruido.
* **Mantenla al día:** una documentación que dice algo distinto de lo que hace el código es peor que no tenerla.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Comentar cada línea | Solo el porqué de lo que no sea obvio |
| Documentar con frases que repiten el nombre del método | Decir algo que el nombre no diga: unidades, límites, errores |
| Dejar código antiguo comentado | Borrarlo (Git guarda el historial) |
| Cambiar el código y olvidar actualizar el comentario | Repasa los comentarios cuando modifiques lo que describen |
| Etiquetas XML mal cerradas | El compilador avisa (`CS1570`): revisa los `<` y `>` |

## Para practicar

Haz los ejercicios de [U6.3 · Comentarios y documentación](documentacion.md): documentar un código, generar la documentación y limpiar comentarios que sobran. [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) también incluye cómo se escriben los comentarios en cada lenguaje.
