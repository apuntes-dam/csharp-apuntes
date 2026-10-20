# 2.4 Depurar: encontrar los errores

**Depurar** es buscar y corregir los errores (*bugs*) de un programa. Los hay de tres clases:

| Tipo | Cuándo aparece | Ejemplo |
|---|---|---|
| **Sintaxis** | Al compilar | Falta un `;` o una llave |
| **Ejecución** | Con el programa en marcha | Dividir entre cero, convertir «abc» en número |
| **Lógicos** | Nunca avisan: el resultado es **incorrecto** | Un bucle que se queda corto |

Los primeros los marca el editor y el compilador y los segundos los muestra el programa al fallar (ver [excepciones](02-excepciones.md)). Los **errores lógicos** son los difíciles, y para esos sirve el depurador.

## Un error que no avisa

Esta función debería devolver la suma de 1 a `n`:

```csharp
int SumaHasta(int n)
{
    int suma = 0;
    for (int i = 1; i < n; i++)    // ERROR: debería ser i <= n
    {
        suma += i;
    }
    return suma;
}

Console.WriteLine($"con error: {SumaHasta(5)}");
```

Salida:

```text
con error: 10
```

El resultado esperado para `n = 5` es **15** (1 + 2 + 3 + 4 + 5), pero sale **10**. El programa no falla ni avisa: simplemente calcula mal. Seguir la ejecución a mano ayuda a verlo:

| Vuelta | `i` | `suma` después |
|---|---|---|
| 1 | 1 | 1 |
| 2 | 2 | 3 |
| 3 | 3 | 6 |
| 4 | 4 | 10 |
| (fin) | | el bucle termina sin sumar el 5 |

La corrección: `for (int i = 1; i <= n; i++)`. Con `<` el último valor queda fuera; con `<=` entra. Con esa línea el resultado es:

```csharp
int SumaHasta(int n)
{
    int suma = 0;
    for (int i = 1; i <= n; i++)
    {
        suma += i;
    }
    return suma;
}

Console.WriteLine($"corregido: {SumaHasta(5)}");
```

Salida:

```text
corregido: 15
```

## El depurador

Hacer esa tabla a mano es lento. El **depurador** ejecuta el programa **paso a paso** y te deja ver el valor de cada variable en cada momento.

1. **Punto de ruptura** (*breakpoint*): una marca en una línea. El programa se ejecuta con normalidad hasta llegar ahí y **se detiene**.
2. **Ejecutar en modo depuración**, no en modo normal.
3. **Avanzar paso a paso** y observar cómo cambian las variables.
4. **Detectar el momento en que un valor deja de ser el esperado**: ahí está el error.

| Acción | Visual Studio y VS Code | JetBrains Rider |
|---|---|---|
| Poner o quitar punto de ruptura | `F9` (o clic en el margen) | `Ctrl+F8` (o clic en el margen) |
| Iniciar depuración | `F5` | `Mayús+F9` |
| Siguiente línea (*step over*) | `F10` | `F8` |
| Entrar en la función (*step into*) | `F11` | `F7` |
| Salir de la función (*step out*) | `Mayús+F11` | `Mayús+F8` |
| Continuar hasta el siguiente punto | `F5` | `F9` |

Las ventanas **Locales** y **Automático** (Variables, en VS Code) muestran los valores, y la **ventana Inmediato** (o *Debug Console*) permite probar expresiones sin modificar el código.

!!! note "Depurar con `dotnet`"
    En Visual Studio Code, con la extensión *C# Dev Kit*, pulsa `F5` y elige el proyecto. Desde la línea de comandos, `dotnet run` ejecuta sin depurar: para depurar se necesita el editor.

### Los tres tipos de paso

* **Step over** (siguiente línea): ejecuta la línea actual **sin entrar** en las funciones que llame.
* **Step into** (entrar): si la línea llama a una función tuya, **entra** para ver qué hace por dentro.
* **Step out** (salir): termina la función actual y vuelve a quien la llamó.

Además del valor de las variables, el depurador muestra la **pila de llamadas**: qué función llamó a cuál hasta llegar al punto actual. Es muy útil para entender cómo se llegó a un error.

## Un método para depurar

1. **Reproduce** el fallo con unos datos concretos y anótalos.
2. **Sospecha** dónde puede estar: ¿qué línea es la última que sabes que funciona?
3. Pon un **punto de ruptura** un poco antes y avanza paso a paso.
4. **Compara** lo que ves con lo que esperabas.
5. **Corrige** una sola cosa y vuelve a probar con los mismos datos.

## Depurador o mensajes en pantalla

Escribir mensajes (`Console.WriteLine`) para ver valores es rápido y a veces suficiente, pero obliga a modificar el código, a ejecutarlo de nuevo cada vez y a acordarse de borrar los mensajes. El depurador permite mirar cualquier variable **sin tocar el código**. Si necesitas mensajes que solo se vean al depurar, `System.Diagnostics.Debug.WriteLine(...)` desaparece al compilar en modo *Release*.

## Errores típicos en los bucles

| Síntoma | Causa habitual |
|---|---|
| Falta el último elemento, o sobra uno | El límite está mal en un punto (`<` frente a `<=`) |
| El programa **no termina** | La variable de control nunca cambia, o la condición no puede dejar de cumplirse |
| El bucle no se ejecuta nunca | La condición es falsa desde el principio |
| El resultado sale solo con la primera vuelta | Falta actualizar un acumulador o está dentro del bloque equivocado |

## Para practicar

El ejercicio [2.4 de depuración](depurar.md) te pide implementar el algoritmo de la burbuja y **depurarlo** paso a paso.
