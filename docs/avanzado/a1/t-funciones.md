# A1.A Funciones como valores

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5 (variables, estructuras de control, colecciones y clases). Si alguna idea se te resiste, vuelve a la unidad correspondiente.

Hasta ahora has **llamado** funciones. Aquí das un paso más: tratar una función **como un valor**. Se puede guardar en una variable, pasar a otra función y devolver desde una función. Es la base de cosas que ya usas sin darte cuenta: ordenar con un criterio, filtrar una lista o reaccionar a un evento.

## Tipo de una función en C#

En C# una función se guarda en un **delegado**: un tipo que describe «una función con estos parámetros y este resultado». No hace falta declararlos a mano casi nunca, porque la biblioteca trae los habituales:

| Delegado | Significa | Ejemplo |
|---|---|---|
| `Func<int, int>` | Recibe un `int` y devuelve un `int` (el **último** tipo es siempre el resultado) | `n => n * 2` |
| `Func<int, int, int>` | Recibe dos `int` y devuelve un `int` | `(a, b) => a + b` |
| `Action<string>` | Recibe un `string` y **no devuelve nada** | `s => Console.WriteLine(s)` |
| `Predicate<int>` | Recibe un `int` y devuelve `bool` (equivale a `Func<int, bool>`) | `n => n % 2 == 0` |

Una **lambda** se escribe `parámetros => expresión` (o `parámetros => { ... }` con varias sentencias). Un método con nombre también se puede pasar como valor escribiendo su nombre **sin paréntesis** (se llama *grupo de métodos*). Si necesitas un tipo con nombre propio, se declara con `delegate`:

```csharp
Func<int, int, int> suma = (a, b) => a + b;         // recibe dos int y devuelve un int
Action<string> saludar = nombre => Console.WriteLine($"Hola, {nombre}");      // no devuelve nada
Predicate<int> esPar = n => n % 2 == 0;             // devuelve bool
Operacion resta = (a, b) => a - b;                  // delegado con nombre propio

Console.WriteLine(suma(2, 3));
saludar("Ana");
Console.WriteLine(esPar(4));
Console.WriteLine(resta(10, 4));
Console.WriteLine(resta.GetType().Name);

delegate int Operacion(int a, int b);
```

Salida:

```text
5
Hola, Ana
True
6
Operacion
```

## Un ejemplo con todo

El programa define una función, la pasa a otra, devuelve una función desde una función y crea dos contadores independientes:

```csharp
int Doble(int n) => n * 2;

// Recibe una función como parámetro
int AplicarDosVeces(Func<int, int> f, int x) => f(f(x));

// Devuelve una función que "recuerda" factor
Func<int, int> Multiplicador(int factor) => n => n * factor;

// La lambda captura la variable cuenta y la modifica
Func<int> CrearContador()
{
    int cuenta = 0;
    return () => ++cuenta;
}

Func<int, int> cuadrado = n => n * n;      // función anónima guardada en una variable

Console.WriteLine($"doble(4) = {Doble(4)}");
Console.WriteLine($"cuadrado(4) = {cuadrado(4)}");
Console.WriteLine($"aplicar dos veces doble a 3 = {AplicarDosVeces(Doble, 3)}");
Console.WriteLine($"multiplicador(5)(7) = {Multiplicador(5)(7)}");

var a = CrearContador();
var b = CrearContador();
Console.WriteLine($"contador A: {a()} {a()} {a()}");
Console.WriteLine($"contador B: {b()}");
```

Salida:

```text
doble(4) = 8
cuadrado(4) = 16
aplicar dos veces doble a 3 = 12
multiplicador(5)(7) = 35
contador A: 1 2 3
contador B: 1
```

Qué ocurre en cada línea:

| Línea de salida | Qué demuestra |
|---|---|
| `doble(4)` y `cuadrado(4)` | Una función con nombre y otra **anónima guardada en una variable** se llaman igual |
| `aplicar dos veces doble a 3` | Una función recibe **otra función como parámetro** (aquí, `Doble` sin paréntesis) y la usa dos veces |
| `multiplicador(5)(7)` | Una función **devuelve otra función**, que recuerda el factor `5` |
| `contador A` y `contador B` | Cada contador **recuerda su propia cuenta**: no se pisan |

## Cierres: funciones que recuerdan

Las lambdas de C# son **cierres**: capturan las **variables** del código que las rodea (no una copia de su valor) y pueden modificarlas. `CrearContador` devuelve una lambda que sigue viendo y cambiando `cuenta` aunque `CrearContador` ya haya terminado, y cada llamada a `CrearContador` crea **su propia** `cuenta`.

Como se captura la variable y no su valor, hay una trampa famosa con los bucles. Mira qué escribe cada bucle:

```csharp
var acciones = new List<Action>();
for (int i = 0; i < 3; i++)
{
    acciones.Add(() => Console.Write($"{i} "));
}
foreach (var accion in acciones)
{
    accion();
}
Console.WriteLine("  <- con for");

acciones.Clear();
foreach (int j in new[] { 0, 1, 2 })
{
    acciones.Add(() => Console.Write($"{j} "));
}
foreach (var accion in acciones)
{
    accion();
}
Console.WriteLine("  <- con foreach");
```

Salida:

```text
3 3 3   <- con for
0 1 2   <- con foreach
```

En el `for` hay **una sola** variable `i` para todo el bucle: cuando se ejecutan las acciones, el bucle ya terminó y `i` vale 3. En el `foreach`, cada vuelta crea una variable `j` **nueva**, así que cada lambda guarda la suya. Si necesitas el valor de ese momento dentro de un `for`, copia la variable a otra local dentro del cuerpo: `int copia = i;`.

!!! tip "Cuándo usarlo"
    Si necesitas «una operación que se decide más tarde» (un criterio de orden, un filtro, qué hacer al terminar), pásala como función. Es más corto y más flexible que crear una clase solo para eso.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a la función al pasarla (`Doble(3)` en vez de `Doble`) | Pasa el nombre **sin paréntesis** si quieres pasar la función, no su resultado |
| Capturar la variable de un `for` y esperar su valor de cada vuelta | Copiarla a una variable local dentro del cuerpo, o usar `foreach` |
| Esperar que cada llamada comparta o no comparta estado sin comprobarlo | Crear el cierre una vez por contador, como en el ejemplo |
| Escribir lambdas largas e ilegibles | Si pasa de una o dos líneas, ponle nombre a la función |
| Usar `Func<int, int>` donde hace falta un `Func<int>` | El **último** tipo genérico es el resultado; los demás son parámetros |

## Para practicar

Los ejercicios [A1.1 y A1.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
