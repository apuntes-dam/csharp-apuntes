# 2.1 Condicionales: decidir qué hacer

Un programa no ejecuta siempre las mismas líneas: **decide** según los datos. Para decidir usa una **condición**, una expresión cuyo resultado es verdadero o falso (un valor **booleano**, `bool`), y ejecuta un bloque u otro según el resultado.

## `if`, `else if` y `else`

La estructura completa tiene tres partes: lo que se hace **si** se cumple la condición, lo que se hace si **no** pero se cumple otra, y lo que se hace en **cualquier otro caso**.

```csharp
string Calificacion(int nota)
{
    if (nota < 0 || nota > 10)
    {
        return "Nota no válida";
    }
    else if (nota < 5)
    {
        return "Insuficiente";
    }
    else if (nota < 6)
    {
        return "Suficiente";
    }
    else if (nota < 7)
    {
        return "Bien";
    }
    else if (nota < 9)
    {
        return "Notable";
    }
    else if (nota < 10)
    {
        return "Sobresaliente";
    }
    else
    {
        return "Matrícula";
    }
}

foreach (int nota in new[] { 3, 5, 6, 8, 9, 10, 11 })
{
    Console.WriteLine($"{nota} -> {Calificacion(nota)}");
}
```

Salida:

```text
3 -> Insuficiente
5 -> Suficiente
6 -> Bien
8 -> Notable
9 -> Sobresaliente
10 -> Matrícula
11 -> Nota no válida
```

Las condiciones se evalúan **de arriba abajo** y se ejecuta **solo la primera rama que se cumple**; el resto se salta. Por eso el orden importa.

!!! warning "El orden de las ramas"
    Si en la función anterior pusieras primero `nota < 10`, la nota 3 entraría ahí y obtendrías «Sobresaliente». Ordena las condiciones de la más restrictiva a la más general, y deja el `else` para lo que no encaja en ninguna.

!!! tip "Casos raros primero"
    Comprobar al principio lo que no es válido (aquí, una nota fuera de 0 a 10) y devolver enseguida evita anidar condiciones y deja el resto del código más claro.

Si el bloque tiene **una sola instrucción**, las llaves son opcionales (`if (x > 0) Console.WriteLine("positivo");`), pero ponerlas siempre evita errores al añadir una segunda línea más tarde.

## Operadores de comparación y lógicos

| Comparación | Significa |
|---|---|
| `==` · `!=` | igual · distinto |
| `<` · `<=` | menor · menor o igual |
| `>` · `>=` | mayor · mayor o igual |

| Lógico | Símbolo en C# | Ejemplo |
|---|---|---|
| Y | `&&` | `edad >= 18 && carnet` |
| O | `\|\|` | `dia == 6 \|\| dia == 7` |
| NO | `!` | `!carnet` |

Las condiciones lógicas se evalúan de izquierda a derecha y **se detienen en cuanto el resultado está claro** (*cortocircuito*): en `a && b`, si `a` es falso ya no se mira `b`. Se aprovecha para escribir `x != 0 && 10 / x > 2` sin riesgo de dividir entre cero.

En C# `==` entre `string` compara el **contenido**, así que `nombre == "Ana"` funciona. Para saber si dos variables son *el mismo objeto* se usa `ReferenceEquals(a, b)`.

!!! warning "`=` no es `==`"
    Un solo `=` **asigna** un valor; dos `==` **comparan**. En C# la condición **debe ser un `bool`**: no vale un número ni un texto (`if (n)` no compila), así que `if (x = 3)` es un error de compilación. Escribe `n != 0` o `texto.Length > 0`.

!!! note "Los textos no se comparan con `<`"
    A diferencia de otros lenguajes, en C# no se puede escribir `"a" < "b"`. Para ordenar o comparar textos se usa `string.Compare(a, b)`, que devuelve un número negativo, cero o positivo, o `string.Equals(a, b, StringComparison.OrdinalIgnoreCase)` para ignorar mayúsculas.

## Una condición en una sola línea

Cuando solo hay que elegir entre dos **valores**, se puede escribir la condición dentro de la expresión con el operador condicional `? :`.

```csharp
void Mostrar(int edad, bool carnet)
{
    string tipo = edad >= 18 ? "mayor" : "menor";
    string permiso = carnet ? "con carnet" : "sin carnet";
    string conducir = edad >= 18 && carnet ? "puede conducir" : "no puede conducir";
    Console.WriteLine($"{edad} años ({tipo}), {permiso} -> {conducir}");
}

Mostrar(17, true);
Mostrar(18, false);
Mostrar(18, true);
```

Salida:

```text
17 años (menor), con carnet -> no puede conducir
18 años (mayor), sin carnet -> no puede conducir
18 años (mayor), con carnet -> puede conducir
```

Úsalo para valores sencillos; si la condición o las ramas se complican, es mejor un `if` normal.

## Selección múltiple

Cuando una misma variable se compara con muchos valores concretos, el `if`/`else if` se hace largo. Para eso existe la **selección múltiple**, con `switch`.

```csharp
string TipoDeDia(int dia)
{
    switch (dia)
    {
        case 1:
        case 2:
        case 3:
        case 4:
        case 5:
            return "laborable";
        case 6:
        case 7:
            return "fin de semana";
        default:
            return "día no válido";
    }
}

foreach (int dia in new[] { 1, 3, 6, 7, 9 })
{
    Console.WriteLine($"{dia} -> {TipoDeDia(dia)}");
}
```

Salida:

```text
1 -> laborable
3 -> laborable
6 -> fin de semana
7 -> fin de semana
9 -> día no válido
```

Cada `case` debe **terminar** con `break`, `return` u otra salida: en C# no se «cae» al siguiente caso (a diferencia de Java o C), excepto cuando se apilan etiquetas vacías, como `case 1: case 2:` arriba. El `default` recoge todo lo demás.

Y en su forma más moderna, la **expresión `switch`**, que devuelve un valor y admite rangos y combinaciones:

```csharp
string TipoDeDia(int dia) => dia switch
{
    >= 1 and <= 5 => "laborable",
    6 or 7 => "fin de semana",
    _ => "día no válido"
};

foreach (int dia in new[] { 1, 3, 6, 7, 9 })
{
    Console.WriteLine($"{dia} -> {TipoDeDia(dia)}");
}
```

Salida:

```text
1 -> laborable
3 -> laborable
6 -> fin de semana
7 -> fin de semana
9 -> día no válido
```

Las dos versiones producen la misma salida. En la expresión `switch`:

* cada caso es `patrón => resultado`, separado por comas;
* `>= 1 and <= 5` es un **patrón de rango** (con `and`, `or` y `not` se combinan);
* `_` es el caso por defecto, que cubre todo lo demás;
* si falta un caso posible, el compilador avisa.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Condiciones en mal orden, y una rama nunca se ejecuta | Prueba con un valor de cada rama y los valores **límite** (aquí 4, 5, 9 y 10) |
| Confundir `=` con `==` | Léelo en voz alta: «es igual a» |
| Olvidar el caso por defecto (`else`, `default`, `_`) | Pregúntate qué pasa con un dato inesperado |
| Condiciones repetidas con `\|\|` que podrían ser un `switch` | Si comparas la misma variable con muchos valores, usa selección múltiple |
| Olvidar el `break` o el `return` en un `case` | C# no compila: no se puede «caer» al siguiente caso |

## Para practicar

Haz los [ejercicios 2.1 de condicionales](condicionales.md). Empieza por los primeros y, cuando funcionen, prueba cada uno con un valor de cada rama y con los límites.
