# P1.2 · Primeros programas

<div class="ej-gate" data-unit="u01" data-nombre="U1 · Primer programa"></div>

Traduce algoritmos sencillos a código. Para cada ejercicio anota **entradas, proceso y salidas** antes de programar. Prueba con valores normales, cero y negativos cuando tenga sentido.

## Ejercicio 1.2.1

Pide el nombre del usuario y salúdalo.

```text
Escribe tu nombre: Juan
Hola, Juan.
```

<details class="sol" data-key="p1-2/1.2.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("Escribe tu nombre: ");
string? nombre = Console.ReadLine();
Console.WriteLine($"Hola, {nombre}.");
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 1.2.2

Pide las horas trabajadas y el precio por hora y muestra el importe total del servicio.

```text
Horas de trabajo: 6
Coste por hora: 10
Importe total: 60
```

## Ejercicio 1.2.3

Suponiendo estas declaraciones:

```csharp
var ancho = 17;      // int
var alto = 12.0;     // double
```

Antes de ejecutarlas, adivina el **valor** y el **tipo** de cada expresión y luego compruébalo con `Console.WriteLine((expresion).GetType().Name)`:

1. `ancho / 2`
2. `ancho / 2.0`
3. `alto / 3`
4. `1 + 2 * 5`

??? success "Respuestas"
    1. `8` → `Int32` (`int / int` es división entera)
    2. `8,5` → `Double` (con coma en un equipo en español)
    3. `4` → `Double` (se muestra `4`, pero es un decimal)
    4. `11` → `Int32` (primero la multiplicación)

## Ejercicio 1.2.4

Pide una temperatura en grados Celsius, conviértela a Fahrenheit y muéstrala.

<details class="sol" data-key="p1-2/1.2.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("Grados Celsius: ");
double celsius = double.Parse(Console.ReadLine()!);
double fahrenheit = celsius * 9 / 5 + 32;
Console.WriteLine($"{celsius} °C = {fahrenheit} °F");
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 1.2.5

Pide el importe de un artículo sin IVA y el tipo de IVA (por ejemplo, 21) y muestra el precio final.

## Ejercicio 1.2.6

Pide el precio final de un artículo y, suponiendo un IVA del 10 %, muestra el IVA pagado y el importe sin IVA.

## Ejercicio 1.2.7

Pide tres números y muestra su suma.

!!! note "En C#"
    Si el usuario escribe los tres números en una sola línea (`4 5 6`), puedes leerla entera y separarla con `Console.ReadLine()!.Split(' ')`.

## Ejercicio 1.2.8

Pide una cantidad de segundos y muéstrala en horas, minutos y segundos. Por ejemplo, `3725` segundos son `1 h 2 min 5 s`.

## Ejercicio 1.2.9

Guarda dos valores en las variables `a` y `b`, pedidos por teclado. Intercambia sus valores (que `a` acabe con lo que tenía `b` y al revés) y muestra ambas variables antes y después del intercambio.

## Ejercicio 1.2.10

Muestra por pantalla el resultado de esta operación: `3 + 5 · (2 + 4)² ÷ 6`. Respeta la precedencia de los operadores y usa la potencia de tu lenguaje.

*(El enunciado original es una imagen; aquí se sustituye por una operación equivalente.)*

## Ejercicio 1.2.11

Lee un entero positivo `n` y muestra la suma de los enteros de 1 a `n`. Hazlo de dos formas: con un bucle y con la fórmula `n · (n + 1) / 2`.

## Ejercicio 1.2.12

Pide peso (kg) y estatura (m), calcula el índice de masa corporal (peso / estatura²) y muestra `Tu índice de masa corporal es X`, con X redondeado a 2 decimales.

!!! note "En C#"
    `double.Parse` y `{x:F2}` usan el idioma del equipo: en uno en español hay que escribir `1,75` (con coma) y se muestra `22,86`. Con `1.75` el punto se toma como separador de miles y el resultado sale mal sin avisar. Para usar siempre el punto, pasa `CultureInfo.InvariantCulture` (de `System.Globalization`).

<details class="sol" data-key="p1-2/1.2.12">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("Peso (kg): ");
double peso = double.Parse(Console.ReadLine()!);
Console.Write("Estatura (m): ");
double estatura = double.Parse(Console.ReadLine()!);
double imc = peso / (estatura * estatura);
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 1.2.13

Pide dos enteros `n` y `m` y muestra `la división de n entre m da un cociente c y un resto r`. Controla también la división entre cero.

<details class="sol" data-key="p1-2/1.2.13">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>Console.Write("n: ");
int n = int.Parse(Console.ReadLine()!);
Console.Write("m: ");
int m = int.Parse(Console.ReadLine()!);
if (m == 0)
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 1.2.14

Una juguetería envía payasos (112 g cada uno) y muñecas (75 g cada una). Lee cuántos de cada se han vendido y calcula el peso total del paquete.

## Ejercicio 1.2.15

Una cuenta de ahorros da un 4 % de interés anual que se suma al saldo a final de año. Lee el dinero depositado y muestra el saldo tras el primer, segundo y tercer año, redondeado a 2 decimales. Fórmula de cada año: `capital · (1 + interés)`.

!!! note "En C#"
    `double.Parse` y `{x:F2}` usan el idioma del equipo: en uno en español hay que escribir `1,75` (con coma) y se muestra `22,86`. Con `1.75` el punto se toma como separador de miles y el resultado sale mal sin avisar. Para usar siempre el punto, pasa `CultureInfo.InvariantCulture` (de `System.Globalization`).

## Ejercicio 1.2.16

Una panadería vende barras a 3,49 € (defínelo como constante). Las que no son del día tienen un 60 % de descuento. Lee cuántas barras no frescas se venden y muestra el precio habitual, el descuento aplicado y el coste total.

## Ejercicio 1.2.17

Pide un nombre y un entero `n` y muestra el nombre `n` veces, cada una en una línea.

## Ejercicio 1.2.18

Pide el nombre completo y muéstralo tres veces: todo en minúsculas, todo en mayúsculas y con la inicial de cada palabra en mayúscula. El usuario puede escribirlo con cualquier combinación de mayúsculas y minúsculas.

!!! note "En C#"
    Separa las palabras con `Split(' ')` y, en cada una, `char.ToUpper(p[0]) + p.Substring(1).ToLower()` (cuidado con las palabras vacías). También existe `CultureInfo.CurrentCulture.TextInfo.ToTitleCase(texto.ToLower())`.

## Ejercicio 1.2.19

Pide un nombre y muestra `NOMBRE tiene n letras.`, con el nombre en mayúsculas.

## Ejercicio 1.2.20

Los teléfonos de una empresa tienen el formato `prefijo-número-extensión`, por ejemplo `+34-913724710-56`. Pide uno y muestra solo el número, sin prefijo ni extensión.

## Ejercicio 1.2.21

Pide una frase y muéstrala invertida.

## Ejercicio 1.2.22

Pide una frase y una vocal, y muestra la frase con esa vocal en mayúscula.

## Ejercicio 1.2.23

Pide un correo electrónico y muestra otro con el mismo nombre de usuario (lo que va antes de la `@`) pero con el dominio `ceu.es`.

## Ejercicio 1.2.24

Pide el precio de un producto en euros con dos decimales y muestra cuántos euros y cuántos céntimos son.

!!! note "En C#"
    No obtengas los céntimos multiplicando un `double` por 100: `19.99 * 100` no da exactamente `1999`, y al convertir a `int` salen 1998. Con `decimal` (`19.99m * 100`) sí sale `1999`; o separa el texto por la coma.

## Ejercicio 1.2.25

Pide una fecha de nacimiento con formato `dd/mm/aaaa` y muestra día, mes y año. Después adáptalo para que funcione si el día o el mes se escriben con un solo dígito.

## Ejercicio 1.2.26

Pide los productos de una cesta de la compra separados por comas y muestra cada producto en una línea.

## Ejercicio 1.2.27

Pide el nombre de un producto, su precio y las unidades, y muestra una línea con el nombre, el precio unitario (6 dígitos enteros y 2 decimales), las unidades (3 dígitos) y el coste total (8 dígitos enteros y 2 decimales).

!!! note "En C#"
    En `$"{precio,9:F2}"`, `,9` reserva 9 posiciones (alineado a la derecha) y `:F2` fija dos decimales (con coma en un equipo en español; ver la nota del ejercicio 1.2.12).

## Ejercicio 1.2.28

Calcula el área de un triángulo a partir de sus tres lados (fórmula de Herón). Indica qué ocurre si las longitudes no pueden formar un triángulo.

!!! note "En C#"
    `Math.Sqrt` de un número negativo no lanza ninguna excepción: devuelve `NaN` (se comprueba con `double.IsNaN`). Compruébalo antes de calcular.

## Ejercicio 1.2.29

Genera un número aleatorio entre dos valores que introduce el usuario (ambos incluidos).

## Ejercicio 1.2.30

Determina si un número es primo. Ten en cuenta el 0, el 1 y los negativos.

## Ejercicio 1.2.31

Muestra todos los divisores de un número.

## Ejercicio 1.2.32

Muestra la serie de Fibonacci hasta un número dado.

## Herramientas útiles en C#

| Necesitas... | Usa... |
|---|---|
| Leer una línea | `Console.ReadLine()` (devuelve `string?`, que puede ser `null`) |
| Texto a número | `int.Parse(...)`, `double.Parse(...)`, `int.TryParse(...)` |
| Redondear al mostrar | `x.ToString("F2")` o `$"{x:F2}"` |
| Potencia y raíz | `Math.Pow(x, 2)`, `Math.Sqrt(x)` |
| Aleatorios | `Random.Shared.Next(a, b + 1)` (el máximo no se incluye, por eso `b + 1`) |
| Mayúsculas / minúsculas | `ToUpper()`, `ToLower()` |
| Cortar y unir texto | `Split(',')`, `Substring(a, longitud)`, `string.Join(", ", lista)` |
| Rellenar con ceros | `"7".PadLeft(3, '0')` |
| Lanzar una excepción | `throw new ...` |

!!! note "Decimales con coma"
    `ToString("F2")` y `double.Parse` usan la configuración regional del sistema: en español mostrarán y esperarán `22,86`. Para forzar el punto, pasa `CultureInfo.InvariantCulture` (de `System.Globalization`): `x.ToString("F2", CultureInfo.InvariantCulture)`.

Para ejecutar cada ejercicio, guárdalo en un archivo (por ejemplo `ejercicio.cs`) y lanza `dotnet run ejercicio.cs`, o crea un proyecto con `dotnet new console`.
