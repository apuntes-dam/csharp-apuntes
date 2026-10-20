# 4.B Encapsulamiento

**Encapsular** es **ocultar el estado interno** de un objeto y dejar solo unas operaciones controladas para usarlo. Así el objeto se encarga de que sus datos siempre tengan sentido: un precio nunca es negativo, un stock nunca baja de cero.

Sin encapsulamiento, cualquiera podría escribir `producto.Precio = -5` y el error aparecería mucho más tarde, lejos de donde se produjo. Con él, el fallo salta **justo al intentar el cambio incorrecto**.

## Visibilidad

C# tiene `private` (solo la clase), `protected` (la clase y sus hijas), `internal` (el proyecto) y `public`. Lo que se escribe **sin modificador dentro de una clase es `private`**. Lo habitual es que los datos que no deban tocarse desde fuera sean `private` o tengan un **`private set`**.

## Acceso controlado y validación

```csharp
var p = new Producto("Cuaderno", 3, 10);
Console.WriteLine($"{p.Nombre}: {p.Precio} € x {p.Stock} = {p.ValorStock} €");
p.Vender(4);
Console.WriteLine($"tras vender 4, quedan {p.Stock}");
try
{
    p.Vender(20);
}
catch (InvalidOperationException e)
{
    Console.WriteLine($"Error: {e.Message}");
}
try
{
    p.Precio = -1;
}
catch (ArgumentException e)
{
    Console.WriteLine($"Error: {e.Message}");
}
p.Precio = 7;
Console.WriteLine($"nuevo valor del stock: {p.ValorStock} €");
try
{
    new Producto("Roto", -2, 1);
}
catch (ArgumentException e)
{
    Console.WriteLine($"Error: {e.Message}");
}

class Producto
{
    private int _precio;                    // el campo que guarda el valor

    public string Nombre { get; }
    public int Stock { get; private set; }  // se lee desde fuera, pero solo cambia desde dentro

    public int Precio
    {
        get => _precio;
        set
        {
            if (value < 0)
            {
                throw new ArgumentException("el precio no puede ser negativo");
            }
            _precio = value;
        }
    }

    public int ValorStock => Precio * Stock;   // propiedad calculada: no guarda nada

    public Producto(string nombre, int precio, int stock)
    {
        Nombre = nombre;
        Precio = precio;       // pasa por el set: se valida igual que desde fuera
        Stock = stock;
    }

    public void Vender(int cantidad)
    {
        if (cantidad > Stock)
        {
            throw new InvalidOperationException("no hay stock suficiente");
        }
        Stock -= cantidad;
    }
}
```

Salida:

```text
Cuaderno: 3 € x 10 = 30 €
tras vender 4, quedan 6
Error: no hay stock suficiente
Error: el precio no puede ser negativo
nuevo valor del stock: 42 €
Error: el precio no puede ser negativo
```

Qué hace cada pieza:

* **Atributos protegidos** (`Precio`, `Stock`): no se pueden cambiar desde fuera sin pasar por el código de la clase.
* **Validar al crear y al modificar**: el constructor asigna `Precio = precio`, que pasa por el `set`, así que **una sola comprobación** cubre las dos puertas de entrada. Se lanza una **excepción** si el valor no es válido (ver [2.3 Excepciones](../u02/02-excepciones.md)).
* **Solo lectura**: el stock se puede consultar, pero **solo** `Vender` lo modifica.
* **Propiedad calculada**: `ValorStock` no se guarda; se calcula cada vez a partir de `Precio` y `Stock`, así nunca queda desactualizada.

Una propiedad `{ get; set; }` sin más código ya es un *getter* y un *setter* automáticos. Cuando necesitas validar, escribes el `set` tú y guardas el valor en un **campo privado** (`_precio`).

!!! tip "El nuevo `field` de C# 14"
    Desde C# 14 (.NET 10) no hace falta declarar el campo privado a mano: dentro de un `get` o un `set`, la palabra `field` se refiere al valor guardado.

```csharp
var p = new Producto { Precio = 5 };
Console.WriteLine(p.Precio);
try
{
    p.Precio = -1;
}
catch (ArgumentException e)
{
    Console.WriteLine($"Error: {e.Message}");
}

class Producto
{
    public int Precio
    {
        get;
        set
        {
            if (value < 0) throw new ArgumentException("el precio no puede ser negativo");
            field = value;
        }
    }
}
```

Salida:

```text
5
Error: el precio no puede ser negativo
```

!!! warning "Valida antes de modificar"
    Fíjate en `Vender`: primero comprueba que hay stock y **solo después** resta. Si la comprobación falla, el objeto queda como estaba. Modificar primero y validar después deja objetos a medias.

### Propiedades de solo escritura al crear: `init` y `required`

Para datos que solo se fijan al crear el objeto y no cambian después, existe `init`. Con `required`, el compilador **obliga** a darles valor:

```csharp
var u = new Usuario { Nombre = "Ana", Edad = 20 };
Console.WriteLine($"{u.Nombre}, {u.Edad}");

class Usuario
{
    public required string Nombre { get; init; }
    public int Edad { get; init; }
}
```

Salida:

```text
Ana, 20
```

`u.Nombre = "Eva"` después de crear el objeto **no compila** (es `init`), y `new Usuario { Edad = 20 }` tampoco (falta el `required`).

## Igualdad: identidad frente a valor

Dos objetos pueden ser **el mismo** (la misma referencia) o ser **iguales** (tener el mismo contenido). Por defecto, `==` en una clase solo reconoce lo primero; para que dos puntos con las mismas coordenadas sean iguales hay que decírselo.

```csharp
Console.WriteLine($"sin igualdad por valor: {(new PuntoSimple(1, 2) == new PuntoSimple(1, 2) ? "sí" : "no")}");
Console.WriteLine($"con igualdad por valor: {(new Punto(1, 2) == new Punto(1, 2) ? "sí" : "no")}");

var conjunto = new HashSet<Punto> { new(1, 2), new(1, 2), new(3, 4) };
Console.WriteLine($"puntos distintos en el conjunto: {conjunto.Count}");
Console.WriteLine(new Punto(1, 2));

class PuntoSimple(int x, int y)
{
    public int X { get; } = x;
    public int Y { get; } = y;
}

record Punto(int X, int Y);
```

Salida:

```text
sin igualdad por valor: no
con igualdad por valor: sí
puntos distintos en el conjunto: 2
Punto { X = 1, Y = 2 }
```

Un **`record`** genera solo `Equals`, `GetHashCode`, `ToString` y la copia con cambios (`p with { X = 5 }`), comparando todas sus propiedades. En una clase normal, `==` compara referencias.

Por la misma razón, en un **conjunto** o como **clave de un mapa**, la igualdad por valor decide si dos objetos son «el mismo elemento» (ver [3.3 Conjuntos](../u03/03-conjuntos.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Campos públicos modificables desde cualquier sitio | Hazlos privados y expón propiedades o métodos |
| Un `set` que acepta cualquier valor | Valida y lanza una excepción si no es correcto |
| Ofrecer `set` para todo «por si acaso» | Solo lo que de verdad deba poder cambiar: lo demás, `get` o `init` |
| Validar en el constructor pero no en el `set` (o al revés) | Que el constructor pase por el `set`, como en el ejemplo |
| Sobrescribir `Equals` sin `GetHashCode` (o al revés) | Siempre juntos y con los mismos campos; o usar un `record` |

## Para practicar

Los ejercicios de validación y atributos de solo lectura están en [U4.2 · POO I](poo-1.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
