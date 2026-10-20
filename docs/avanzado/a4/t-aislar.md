# A4.C Aislar lo que no controlas

Una buena prueba da **siempre el mismo resultado**. Pero hay código que depende de cosas que no controlas: **la hora actual**, un número aleatorio, internet, un archivo, una base de datos, el envío de un correo. Si tu función llama directamente a `DateTime.Now`, su resultado cambia con la hora y no puedes escribir una prueba fiable; si envía un correo de verdad, cada prueba molestaría a alguien.

## La solución: pedir las dependencias desde fuera

En vez de que la función **vaya a buscar** el reloj, el correo o la base de datos, se los **dan** ya hechos (esto es la *inversión de dependencias* de la [unidad 6](../../u06/t-solid-2.md)). Se declara una **interfaz** con lo que necesitas (`IReloj`, `IEnviador`) y la clase la recibe por el constructor. En producción se le pasa la implementación real; en las pruebas, una **falsa**.

## Dobles de prueba

Un **doble de prueba** es un objeto falso que sustituye a la dependencia real. Hay varios tipos:

| Doble | Para qué sirve | Ejemplo |
|---|---|---|
| **Stub** | Devuelve valores fijos | Un reloj que siempre dice que son las 8 |
| **Fake** | Una versión simple pero funcional | Una base de datos en memoria |
| **Spy** | Apunta cómo se le usó, para comprobarlo | Un enviador que guarda lo que le pidieron enviar |
| **Mock** | Falso configurado con lo que *debe* recibir; falla si no ocurre | Generado con una biblioteca como Moq o NSubstitute |

Se pueden escribir a mano, como aquí, o generarlos con bibliotecas (**Moq**, **NSubstitute**); para el tiempo, .NET trae la clase `TimeProvider` y existe un `FakeTimeProvider` para pruebas. Escribirlos a mano primero ayuda a entender qué hacen:

`Servicios.cs`:

```csharp
// Mal acoplada sería: public string Saludo() => DateTime.Now.Hour < 12 ? ...   (depende del reloj real)
// Bien: el reloj se pide desde fuera, como una interfaz

public interface IReloj
{
    int Hora { get; }
}

public class RelojReal : IReloj
{
    public int Hora => DateTime.Now.Hour;
}

public class Saludador(IReloj reloj)
{
    public string Saludo()
    {
        int hora = reloj.Hora;
        if (hora < 12) return "buenos días";
        if (hora < 20) return "buenas tardes";
        return "buenas noches";
    }
}

public interface IEnviador
{
    void Enviar(string destino, string mensaje);
}

public class Avisador(IEnviador enviador)
{
    public void AvisarSiHayDeuda(string cliente, int deuda)
    {
        if (deuda > 0)
        {
            enviador.Enviar(cliente, $"debes {deuda} €");
        }
    }
}
```

`AislarTests.cs`:

```csharp
using Xunit;

// Un STUB: devuelve lo que le digas
class RelojFalso(int hora) : IReloj
{
    public int Hora { get; } = hora;
}

// Un SPY: apunta lo que le pidieron para poder comprobarlo después
class EnviadorEspia : IEnviador
{
    public List<(string Destino, string Mensaje)> Enviados { get; } = [];

    public void Enviar(string destino, string mensaje) => Enviados.Add((destino, mensaje));
}

public class AislarTests
{
    [Theory]
    [InlineData(8, "buenos días")]
    [InlineData(15, "buenas tardes")]
    [InlineData(22, "buenas noches")]
    public void ElSaludoDependeDeLaHora(int hora, string esperado)
    {
        var saludador = new Saludador(new RelojFalso(hora));      // el reloj es nuestro

        Assert.Equal(esperado, saludador.Saludo());
    }

    [Fact]
    public void SiHayDeudaSeEnviaUnAviso()
    {
        var espia = new EnviadorEspia();

        new Avisador(espia).AvisarSiHayDeuda("ana", 30);

        Assert.Single(espia.Enviados);
        Assert.Equal(("ana", "debes 30 €"), espia.Enviados[0]);
    }

    [Fact]
    public void SinDeudaNoSeEnviaNada()
    {
        var espia = new EnviadorEspia();

        new Avisador(espia).AvisarSiHayDeuda("ana", 0);

        Assert.Empty(espia.Enviados);
    }
}
```

Resultado de `dotnet test`:

```text
Con error: 0, Superado: 5, Omitido: 0, Total: 5
```

Las cinco pruebas (tres del saludo y dos del aviso) pasan sin depender de la hora real ni enviar nada. Cuando el parámetro es una sola operación, basta con pasar **una función** en vez de una interfaz: un `Func<int>` para el reloj (`() => 8`) es el doble más sencillo posible, y es lo que usa el ejercicio A4.6.

## Qué probar y qué no

* **Prueba tu lógica**, no la de la biblioteca. No hace falta probar que `List.Add` añade.
* Cuanto más aislada esté una prueba, más rápida y fiable es; una prueba que usa red o base de datos de verdad es una prueba de **integración** y se escribe aparte, en menos cantidad.
* No abuses de los dobles: si para probar una función necesitas cinco, quizá la función hace demasiado ([6.B Responsabilidad única](../../u06/t-solid-1.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a `DateTime.Now`, `Random` o a internet dentro de la lógica | Pedirlos desde fuera (interfaz, `Func` o `TimeProvider`) |
| Pruebas que a veces pasan y a veces no | Eliminar la dependencia no controlada con un doble |
| Un mock que repite línea por línea la implementación | Comprobar el resultado o el efecto, no cada llamada interna |
| Crear la dependencia real dentro de la clase (`new RelojReal()`) | Recibirla por el constructor |
| Usar pruebas con datos de verdad como si fueran unitarias | Separarlas en un proyecto de integración |

## Para practicar

El ejercicio [A4.6](ejercicios.md) usa un reloj falso. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
