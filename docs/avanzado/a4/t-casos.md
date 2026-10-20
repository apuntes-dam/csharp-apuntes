# A4.B Errores y casos límite

## Probar que algo falla

Un programa correcto no solo da el resultado bueno con datos buenos: también **falla de la forma prevista** con datos malos. Eso también se prueba. En xUnit, `Assert.Throws<T>` comprueba que un trozo de código lanza **exactamente** esa excepción (una subclase no vale; para aceptarla, `Assert.ThrowsAny<T>`), y devuelve la excepción por si quieres mirar su mensaje:

```csharp
var error = Assert.Throws<ArgumentOutOfRangeException>(() => Edades.Categoria(-1));
Assert.Contains("negativa", error.Message);
```

Fíjate en que el código que debe fallar va **dentro de una lambda**: si lo escribieras suelto, la excepción saltaría antes de que `Assert.Throws` pudiera atraparla.

## Casos límite

Los errores se esconden en los **bordes**: el primer y último valor de cada tramo, el cero, el vacío, el negativo, el máximo. Una función que clasifica edades en tramos (`niño` hasta 12, `adolescente` hasta 17...) no se prueba con una edad por tramo, sino con **los valores de cada frontera**: 12 y 13, 17 y 18, 64 y 65. Un `<` en vez de un `<=` solo se descubre ahí.

| Tipo de dato | Casos límite habituales |
|---|---|
| Números | `0`, `1`, `-1`, el mínimo y el máximo permitidos, uno más allá |
| Texto | Vacío, un solo carácter, con espacios, con mayúsculas, con tildes |
| Colecciones | Vacía, un elemento, muchos, con repetidos |
| Intervalos | Los dos extremos y los valores justo fuera |

## Varios casos con una tabla

Repetir un método de prueba para cada valor es tedioso. En xUnit, **`[Theory]`** convierte un método con parámetros en una prueba que se ejecuta una vez por cada `[InlineData(...)]`, y el informe muestra cada caso por separado. Si un solo caso falla, sabes exactamente cuál.

## El ejemplo

`Edades.cs`:

```csharp
public static class Edades
{
    public static string Categoria(int edad)
    {
        if (edad < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(edad), "la edad no puede ser negativa");
        }
        if (edad <= 12) return "niño";
        if (edad <= 17) return "adolescente";
        if (edad <= 64) return "adulto";
        return "mayor";
    }
}
```

`EdadesTests.cs`:

```csharp
using Xunit;

public class EdadesTests
{
    // Probar que algo FALLA: el error esperado es parte del comportamiento
    [Fact]
    public void UnaEdadNegativaLanzaError()
    {
        var error = Assert.Throws<ArgumentOutOfRangeException>(() => Edades.Categoria(-1));
        Assert.Contains("negativa", error.Message);
    }

    // Varios casos con una tabla: [Theory] repite el método con cada [InlineData]
    [Theory]
    [InlineData(0, "niño")]            // límite inferior
    [InlineData(12, "niño")]           // último de niño
    [InlineData(13, "adolescente")]    // primero de adolescente
    [InlineData(17, "adolescente")]
    [InlineData(18, "adulto")]
    [InlineData(64, "adulto")]
    [InlineData(65, "mayor")]
    [InlineData(120, "mayor")]
    public void LaCategoriaEnLosLimites(int edad, string esperada)
    {
        Assert.Equal(esperada, Edades.Categoria(edad));
    }
}
```

Resultado de `dotnet test`:

```text
Con error: 0, Superado: 9, Omitido: 0, Total: 9
```

El resultado cuenta **9 pruebas**: la del error y una por cada fila de la tabla. (Las filas se leen como casos independientes; xUnit ejecuta la `[Theory]` una vez por cada una.) Para tablas más grandes o datos calculados existen `[MemberData]` y `[ClassData]`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Probar solo el «camino feliz» | Añadir los errores y los límites |
| Poner el código que falla fuera de la lambda de `Assert.Throws` | Siempre dentro: `() => ...` |
| Aceptar cualquier excepción | Comprueba el tipo concreto y, si importa, el mensaje |
| Probar un valor por tramo y no las fronteras | Probar justo a cada lado de cada límite |
| Copiar y pegar la misma prueba con otros números | `[Theory]` con `[InlineData]` |

## Para practicar

Los ejercicios [A4.5 y A4.6](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
