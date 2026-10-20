# 1.6 Bucles: `for`, `foreach` y `while`

Un bucle repite un bloque de código. En C# hay cuatro formas: `for`, `foreach`, `while` y `do-while`.

## `for`: cuando sabes cuántas veces

```csharp
for (int i = 1; i <= 5; i++)
{
    Console.WriteLine($"Número: {i}");
}
```

| Parte | Qué hace |
|---|---|
| `int i = 1` | **Inicio**: se ejecuta una vez, antes de empezar |
| `i <= 5` | **Condición**: se comprueba antes de cada vuelta; si es falsa, el bucle termina |
| `i++` | **Avance**: se ejecuta al final de cada vuelta |
| `{ ... }` | Lo que se repite |

## `foreach`: recorrer una colección

```csharp
var nombres = new List<string> { "Ana", "Luis", "Eva" };
foreach (string nombre in nombres)
{
    Console.WriteLine(nombre);
}
```

`foreach` recorre **todos los elementos** de una colección, un array o un texto, sin que te ocupes del índice. No se puede cambiar la colección mientras se recorre con `foreach`.

## Rangos de números

C# no tiene una forma abreviada como `1..5` para el bucle, pero se pueden generar números con `Enumerable.Range(inicio, cantidad)`:

```csharp
foreach (int n in Enumerable.Range(1, 5))     // 1, 2, 3, 4, 5  (empieza en 1 y da 5 valores)
{
    Console.WriteLine(n);
}
```

## Variantes de `for`

```csharp
for (int i = 5; i >= 1; i--) Console.WriteLine(i);     // descendente
for (int i = 1; i <= 10; i += 2) Console.WriteLine(i); // de dos en dos
for (int i = 1; i < 5; i++) Console.WriteLine(i);      // sin llegar a 5
```

Para recorrer con índice, sigue siendo lo más claro un `for` sobre la longitud:

```csharp
var nombres = new List<string> { "Ana", "Luis", "Eva" };
for (int i = 0; i < nombres.Count; i++)
{
    Console.WriteLine($"{i}: {nombres[i]}");
}
```

## Comparación rápida

| Característica | `for` | `foreach` |
|---|---|---|
| Cuándo | Sabes cuántas vueltas, o necesitas el índice | Recorrer todos los elementos |
| `break` / `continue` | Sí | Sí |
| Modificar la colección mientras recorre | Posible con cuidado | No (lanza una excepción) |
| Legibilidad | Más ruido (inicio, condición, avance) | Muy clara |

## `while` y `do-while`

```csharp
int cuenta = 3;
while (cuenta > 0)          // comprueba antes
{
    Console.WriteLine(cuenta--);
}

int intentos = 0;
do                          // se ejecuta al menos una vez
{
    intentos++;
    Console.WriteLine($"Intento {intentos}");
} while (intentos < 3);
```

## Cortar un bucle con una variable de control

Una forma sencilla de parar un bucle sin `break`: una variable booleana (una *bandera*) que el bucle consulta en su condición. Cuando pasa lo que buscas, la pones en `false`.

```csharp
bool activo = true;
int i = 1;

while (activo)
{
    Console.WriteLine(i);
    if (i == 5)
    {
        activo = false;    // se cumple la condición: el bucle se detiene
    }
    i++;
}
```

??? note "Opcional: `break` y `continue`"
    Existen y funcionan igual que en Java, pero no son imprescindibles: casi siempre se puede escribir lo mismo con una variable de control.

    ```csharp
    for (int i = 1; i <= 10; i++)
    {
        if (i == 3) continue;   // salta esta vuelta
        if (i == 6) break;      // sale del bucle
        Console.WriteLine(i);   // 1, 2, 4, 5
    }
    ```
