# 1.1 Un programa

Un **programa** es una secuencia de instrucciones que resuelve un problema, y el **algoritmo** son los pasos, en orden, que van de los datos de entrada al resultado.

!!! info "La teoría general está aparte"
    Qué es un algoritmo, el ciclo de desarrollo y el **pseudocódigo** son iguales en todos los lenguajes, así que están en una sola página: [Fundamentos: algoritmos y pseudocódigo](https://apuntes-dam.github.io/apuntes-lenguajes/fundamentos/). Aquí solo ves cómo se traduce a C#.

## Del pseudocódigo a C#

El algoritmo del área de un rectángulo, en pseudocódigo:

```text
ALGORITMO areaRectangulo
  LEER base
  LEER altura
  area <- base * altura
  ESCRIBIR "Área:", area
FIN
```

y su traducción a C#:

```csharp
Console.Write("Base: ");
double baseRect = double.Parse(Console.ReadLine()!);
Console.Write("Altura: ");
double altura = double.Parse(Console.ReadLine()!);

double area = baseRect * altura;
Console.WriteLine($"Área: {area}");
```

| Pseudocódigo | C# |
|---|---|
| `LEER x` | `Console.ReadLine()` (devuelve texto; se convierte con `int.Parse(...)`, `double.Parse(...)`) |
| `x <- expresión` | `x = expresión;` |
| `ESCRIBIR x` | `Console.WriteLine(x)` |
| `SI ... SINO ... FIN SI` | `if` / `else` (ver [1.2](02-lenguaje.md)) |
| `PARA` / `MIENTRAS` | `for` / `while` (ver [1.6](06-bucles.md)) |

!!! warning "Errores típicos en C#"
    Olvidar el `;` al final de cada instrucción, usar una variable sin inicializar (C# no compila), olvidar que `int / int` es división entera y no convertir el texto de `Console.ReadLine()` antes de operar. Los errores generales de diseño (olvidar un caso, bucles que no terminan…) están en [Fundamentos](https://apuntes-dam.github.io/apuntes-lenguajes/fundamentos/).
