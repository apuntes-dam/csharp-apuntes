# 1.2 Práctica con el lenguaje

## Estructura de un programa C#

Desde C# 9 un programa puede ser solo una lista de instrucciones, sin clase ni `Main`. Se llaman **instrucciones de nivel superior** (*top-level statements*):

```csharp
// Programa completo: se guarda como Program.cs
Console.WriteLine("Hola, C#");
Console.WriteLine(Cuadrado(5));      // 25

int Cuadrado(int n) => n * n;        // una función local
```

La forma clásica, que sigue siendo válida y es la que ves en libros y proyectos antiguos, mete todo en una clase con un método `Main`:

```csharp
using System;

class Programa
{
    static int Cuadrado(int n) => n * n;

    static void Main()
    {
        Console.WriteLine("Hola, C#");
        Console.WriteLine(Cuadrado(5));   // 25
    }
}
```

Reglas básicas: cada instrucción termina en `;`, los bloques van entre `{ }` y C# **distingue mayúsculas de minúsculas** (`Console` no es `console`). Las directivas `using` indican qué espacios de nombres usas; en los proyectos actuales los más comunes (`System`, `System.Linq`, `System.Collections.Generic`...) ya están incluidos de forma implícita.

## Variables

```csharp
string nombre = "Ana";        // tipo explícito
int edad = 20;
edad = 21;                    // se puede cambiar
var altura = 1.68;            // el compilador deduce el tipo: double
double peso;                  // declarada, sin valor todavía
peso = 62.5;                  // hay que darle valor antes de leerla
Console.WriteLine($"{nombre} tiene {edad} años y mide {altura}");
```

`var` **no** significa «cualquier tipo»: el tipo se fija al compilar según el valor inicial. C# es de tipado estático.

!!! note "Variables que pueden valer `null`"
    En los proyectos nuevos, los tipos de referencia (como `string`) **no admiten `null`** salvo que lo indiques con `?`: `string? apodo = null;`. Para los números se hace igual con `int? edad = null;`. Se da un valor por defecto con `??` y se accede con seguridad con `?.`.

## Constantes

| Forma | Cuándo se asigna | Ejemplo |
|---|---|---|
| `const` | En compilación, para valores que no cambian nunca | `const double Iva = 0.21;` |
| `readonly` | Una vez, en la creación del objeto (en una clase) | `readonly DateTime inicio = DateTime.Now;` |

Una `const` solo puede ser un número, un texto o un valor simple fijo. Por convención se escriben con la primera letra en mayúscula.

## Literales

`42` (`int`), `42L` (`long`), `3.14` (`double`), `3.14f` (`float`), `3.14m` (`decimal`), `'a'` (`char`), `"hola"` (`string`) y `true` (`bool`). Se pueden separar los miles con guion bajo: `1_000_000`.

## Operadores

```csharp
Console.WriteLine(7 + 2);        // 9
Console.WriteLine(7 / 2);        // 3    (int / int = división entera)
Console.WriteLine(7 / 2.0);      // 3,5  (con el separador decimal de tu idioma)
Console.WriteLine(7 % 2);        // 1
Console.WriteLine(3 > 2 && 2 > 1);   // True
int x = 5;
x += 2;                          // 7
x++;                             // 8
Console.WriteLine(x);
string? apodo = null;
Console.WriteLine(apodo ?? "sin apodo");   // sin apodo
Console.WriteLine(x > 5 ? "grande" : "pequeño");   // operador condicional
```

Grupos: aritméticos (`+ - * / %`), relacionales (`== != < > <= >=`), lógicos (`&& || !`), asignación (`= += -= ++ --`) y los propios de C#: `??` (valor por defecto si es `null`), `?.` (acceso seguro) y `? :` (condicional).

!!! tip "Comparar textos"
    En C# `==` entre `string` compara el **contenido**, como en Kotlin y Python, y a diferencia de Java.

## Entrada y salida

```csharp
Console.Write("¿Cómo te llamas? ");
string? nombre = Console.ReadLine();
Console.WriteLine($"Hola, {nombre}");               // interpolación de texto
Console.WriteLine($"Tienes {20 + 1} años");          // admite expresiones entre llaves
```

`Console.ReadLine()` devuelve `string?`, porque puede ser `null` si no hay más datos de entrada (por ejemplo, al acabar un archivo redirigido). `Write` no añade salto de línea y `WriteLine` sí.

## Comentarios

```csharp
// Comentario de una línea
/* Comentario
   de varias líneas */
/// <summary>Comentario de documentación XML: lo lee el IDE</summary>
```
