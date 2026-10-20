# 1.4 Pruebas

Una **prueba unitaria** comprueba automáticamente que una función devuelve lo esperado. En C# se suele usar **xUnit**, aunque también existen NUnit y MSTest, que funcionan de forma muy parecida.

## Crear el proyecto de pruebas

Se hace en dos proyectos: el que contiene tu código y otro de pruebas que lo referencia.

```bash
dotnet new classlib -n Calculadora
dotnet new xunit -n Calculadora.Tests
dotnet add Calculadora.Tests reference Calculadora
```

## La función

```csharp
// Calculadora/Operaciones.cs
namespace Calculadora;

public static class Operaciones
{
    public static int Suma(int a, int b) => a + b;

    public static double Dividir(int a, int b)
    {
        if (b == 0) throw new ArgumentException("b no puede ser 0");
        return (double)a / b;
    }
}
```

## La prueba

```csharp
// Calculadora.Tests/OperacionesTests.cs
using Calculadora;

namespace Calculadora.Tests;

public class OperacionesTests
{
    [Fact]
    public void SumaDosPositivos()
    {
        Assert.Equal(5, Operaciones.Suma(2, 3));
    }

    [Fact]
    public void DividirPorCeroLanzaExcepcion()
    {
        Assert.Throws<ArgumentException>(() => Operaciones.Dividir(1, 0));
    }

    [Theory]
    [InlineData(1, 2, 3)]
    [InlineData(0, 0, 0)]
    [InlineData(-1, 1, 0)]
    public void SumaVariosCasos(int a, int b, int esperado)
    {
        Assert.Equal(esperado, Operaciones.Suma(a, b));
    }
}
```

```bash
dotnet test
```

- `[Fact]` marca una prueba **sin parámetros**.
- `[Theory]` con `[InlineData(...)]` repite la misma prueba con varios juegos de datos.
- `Assert.Equal(esperado, obtenido)` comprueba un valor y `Assert.Throws<T>` comprueba que se lanza esa excepción.

!!! tip "Patrón AAA"
    **A**rrange (preparar), **A**ct (ejecutar), **A**ssert (comprobar).
