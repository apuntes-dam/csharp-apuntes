# 2.3 Excepciones: cuando algo sale mal

Una **excepción** es un aviso de que ha ocurrido un error **mientras el programa se ejecuta** (convertir «abc» en número, dividir entre cero, abrir un archivo que no existe). Si nadie la controla, el programa **se detiene** y muestra un mensaje con la pila de llamadas.

Las excepciones permiten separar dos cosas: el camino normal del programa y qué hacer cuando algo falla.

## Capturar: `try`, `catch` y `finally`

```csharp
void Procesar(string texto)
{
    try
    {
        int n = int.Parse(texto);
        Console.WriteLine($"OK: {n} -> doble {n * 2}");
    }
    catch (FormatException)
    {
        Console.WriteLine($"Error: «{texto}» no es un número entero");
    }
    finally
    {
        Console.WriteLine($"-- procesado {texto}");
    }
}

foreach (string texto in new[] { "12", "abc", "7" })
{
    Procesar(texto);
}
```

Salida:

```text
OK: 12 -> doble 24
-- procesado 12
Error: «abc» no es un número entero
-- procesado abc
OK: 7 -> doble 14
-- procesado 7
```

* El bloque **`try`** contiene el código que puede fallar. Si todo va bien, se ejecuta entero.
* Si ocurre una excepción de ese tipo, el programa **salta** al bloque `catch` y el resto del `try` no se ejecuta.
* El bloque **`finally`** se ejecuta **siempre**, haya o no excepción: es el sitio para cerrar archivos o liberar recursos.

En C# se captura con `catch (Tipo e)`; si no necesitas la variable, basta `catch (Tipo)`. Como en Kotlin, **no hay excepciones comprobadas**: nadie te obliga a capturarlas. Para convertir texto sin excepciones existe `int.TryParse`, que devuelve `false` si no es un número.

Se pueden poner **varios `catch`**, del más concreto al más general; entra solo el primero que encaja:

```csharp
foreach (string texto in new[] { "10", "abc", "", "99999999999" })
{
    try
    {
        int n = int.Parse(texto);
        Console.WriteLine($"{n}");
    }
    catch (FormatException)
    {
        Console.WriteLine($"«{texto}» no es un número");
    }
    catch (OverflowException)
    {
        Console.WriteLine($"«{texto}» es demasiado grande para un int");
    }
}
```

Salida:

```text
10
«abc» no es un número
«» no es un número
«99999999999» es demasiado grande para un int
```

!!! tip "Captura el tipo concreto"
    Captura la excepción **más concreta posible**. Una captura genérica (`catch (Exception)`) esconde errores que no esperabas y complica encontrarlos.

## Lanzar y crear excepciones

Tú también puedes avisar de un error: eso es **lanzar** una excepción con **`throw`**. Se hace cuando un método recibe datos que no puede aceptar o cuando una operación no se puede completar. Una excepción propia es una clase que hereda de `Exception` (las clases se explican a fondo en la unidad 4).

```csharp
var cuenta = new Cuenta(100);
try
{
    cuenta.Retirar(30);
    Console.WriteLine($"Saldo: {cuenta.Saldo}");
    cuenta.Retirar(200);
}
catch (SaldoInsuficiente e)
{
    Console.WriteLine($"Error: {e.Message}, faltan {e.Falta} €");
}
try
{
    cuenta.Retirar(-5);
}
catch (ArgumentException e)
{
    Console.WriteLine($"Error: {e.Message}");
}
Console.WriteLine($"Saldo final: {cuenta.Saldo}");

class SaldoInsuficiente : Exception
{
    public int Falta { get; }
    public SaldoInsuficiente(int falta) : base("saldo insuficiente")
    {
        Falta = falta;
    }
}

class Cuenta
{
    public int Saldo { get; private set; }
    public Cuenta(int saldo) { Saldo = saldo; }

    public void Retirar(int cantidad)
    {
        if (cantidad <= 0)
        {
            throw new ArgumentException("la cantidad debe ser positiva");
        }
        if (cantidad > Saldo)
        {
            throw new SaldoInsuficiente(cantidad - Saldo);
        }
        Saldo -= cantidad;
    }
}
```

Salida:

```text
Saldo: 70
Error: saldo insuficiente, faltan 130 €
Error: la cantidad debe ser positiva
Saldo final: 70
```

Para los argumentos inválidos, .NET trae excepciones ya hechas: `ArgumentException`, `ArgumentNullException`, `ArgumentOutOfRangeException`... y para el estado del objeto, `InvalidOperationException`.

Fíjate en el orden: primero se comprueba que los datos son válidos y solo después se modifica el saldo. Así, si hay un error, el objeto **queda como estaba**.

!!! warning "Volver a lanzar: `throw;`, no `throw e;`"
    Si capturas una excepción y quieres **volver a lanzarla**, escribe solo `throw;`. Con `throw e;` se pierde la pila de llamadas original y es mucho más difícil saber dónde empezó el fallo.

```csharp
try
{
    try
    {
        int.Parse("abc");
    }
    catch (FormatException)
    {
        Console.WriteLine("La entrada no es correcta");
        throw;                      // vuelve a lanzar la MISMA excepción
    }
}
catch (FormatException e)
{
    Console.WriteLine($"Capturada fuera: {e.GetType().Name}");
}
```

Salida:

```text
La entrada no es correcta
Capturada fuera: FormatException
```

## Un patrón útil: pedir un dato hasta que sea válido

Los ejercicios te piden a menudo que el programa no se rompa con lo que escriba el usuario. El patrón es un bucle que **repite la petición** hasta que el dato es correcto:

```csharp
while (true)
{
    Console.WriteLine("Escribe un número positivo:");
    string linea = Console.ReadLine() ?? "";
    try
    {
        int n = int.Parse(linea.Trim());
        if (n <= 0)
        {
            throw new FormatException("no es positivo");
        }
        Console.WriteLine($"Has escrito {n}");
        break;
    }
    catch (FormatException)
    {
        Console.WriteLine("Eso no es un número positivo");
    }
}
```

Salida:

```text
Escribe un número positivo:
Eso no es un número positivo
Escribe un número positivo:
Eso no es un número positivo
Escribe un número positivo:
Has escrito 42
```

Observa que un número negativo **no** es una excepción de la conversión, así que se lanza a mano para tratarlo igual que un texto no numérico. Con `TryParse` se puede escribir lo mismo sin excepciones:

```csharp
if (int.TryParse(linea, out int n) && n > 0) { /* dato válido */ }
```

## Cuándo usar una excepción

| Situación | Qué hacer |
|---|---|
| El usuario puede equivocarse al escribir | Captura la excepción (o usa `TryParse`) y vuelve a pedir el dato |
| Un método recibe un argumento imposible | **Lánzala**: es un fallo de quien lo llama |
| Algo previsible (una lista vacía) | Compruébalo con un `if`: no abuses de las excepciones |
| Error que no sabes arreglar | Déjalo subir: que lo capture quien sepa qué hacer |

## Errores frecuentes

* **Capturar y no hacer nada.** Un bloque `catch` vacío esconde el problema. Como mínimo, muestra o registra el error.
* **Capturar demasiado.** Pon en el `try` solo las líneas que pueden fallar.
* **Modificar el estado antes de validar**, de forma que un error deja el objeto a medias.
* **Olvidar que `Console.ReadLine()` puede devolver `null`**: `int.Parse(null)` lanza `ArgumentNullException`, no `FormatException`.

## Para practicar

Haz los [ejercicios 2.3 de excepciones](excepciones.md). El objetivo es que **ninguna excepción llegue al programa principal** y lo aborte.
