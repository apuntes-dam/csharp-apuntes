# 4.A Clases y objetos

Hasta ahora los datos (variables) y las operaciones (funciones) iban por separado. En la **programación orientada a objetos (POO)** se agrupan en una sola pieza: el **objeto**, que guarda sus propios datos y sabe hacer sus propias operaciones.

| Concepto | Qué es | Ejemplo |
|---|---|---|
| **Clase** | La **plantilla** que describe cómo son y qué hacen los objetos de un tipo | `Libro` |
| **Objeto** (o *instancia*) | Una **cosa concreta** creada a partir de la clase | «El Quijote», «Dune» |
| **Atributo** | Un dato que guarda cada objeto (su **estado**) | `Titulo`, `Paginas`, `Leidas` |
| **Método** | Una operación que sabe hacer el objeto (su **comportamiento**) | `Leer(...)`, `Progreso()` |
| **Constructor** | Código que se ejecuta al **crear** el objeto y lo deja listo | `new Libro("Dune", 400)` |

Todos los libros comparten la misma clase, pero **cada objeto tiene su propio estado**: leer páginas de un libro no cambia el otro.

## Tu primera clase

```csharp
var quijote = new Libro("El Quijote", 1000);
var dune = new Libro("Dune", 400);
dune.Leer(100);
Console.WriteLine(quijote);
Console.WriteLine(dune);
Console.WriteLine($"progreso de Dune: {dune.Progreso()}%");

var otro = dune;    // no es una copia: es el mismo objeto
otro.Leer(50);
Console.WriteLine($"tras leer 50 más: {dune}");

class Libro
{
    public string Titulo { get; }
    public int Paginas { get; }
    public int Leidas { get; private set; }

    public Libro(string titulo, int paginas)
    {
        Titulo = titulo;
        Paginas = paginas;
    }

    public void Leer(int cantidad)
    {
        Leidas += cantidad;
    }

    public int Progreso() => Leidas * 100 / Paginas;

    public override string ToString() => $"Libro({Titulo}, {Paginas} págs, leídas {Leidas})";
}
```

Salida:

```text
Libro(El Quijote, 1000 págs, leídas 0)
Libro(Dune, 400 págs, leídas 100)
progreso de Dune: 25%
tras leer 50 más: Libro(Dune, 400 págs, leídas 150)
```

Observa cuatro cosas:

1. La clase `Libro` se escribe **una vez** y se usa para crear tantos objetos como se quiera (`quijote`, `dune`).
2. Los métodos (`Leer`, `Progreso`) trabajan con los atributos **del objeto sobre el que se llaman**.
3. `ToString` define **cómo se convierte el objeto en texto**: es lo que se muestra al imprimirlo.
4. Al final, `otro = dune` **no crea un libro nuevo**: ahora `otro` y `dune` son dos nombres para el **mismo objeto**, y leer con uno cambia lo que se ve con el otro. Los objetos se manejan **por referencia**, igual que las listas del [apartado 3.1](../u03/01-listas.md).

!!! note "Las clases van al final del archivo"
    En un programa con instrucciones de nivel superior (sin `Main`), las **clases se escriben después** de las instrucciones, como en el ejemplo. En un proyecto con varios archivos, cada clase suele tener el suyo.

## Cómo se escribe en C&#35;

| Necesito... | En C# |
|---|---|
| Declarar la clase | `class Libro { ... }` |
| Atributo | Una **propiedad**: `public string Titulo { get; }` (solo lectura) · `public int Leidas { get; private set; }` (solo se cambia desde dentro) |
| Constructor | Un método con el nombre de la clase y sin tipo de retorno: `public Libro(string titulo, int paginas) { ... }` |
| Crear un objeto | `new Libro("Dune", 400)` (o `Libro dune = new("Dune", 400);`) |
| Llamar a un método | `dune.Leer(100)` |
| El propio objeto | `this` (casi siempre se puede omitir) |
| Texto del objeto | `public override string ToString() => "...";` |

En C# lo habitual es que los datos de un objeto sean **propiedades** y no variables sueltas (**campos**). Una propiedad `{ get; set; }` parece un atributo, pero por detrás puede validar, calcular o restringir quién puede cambiarla (se ve en el [apartado siguiente](t-encapsulamiento.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Confundir la **clase** con el **objeto** | La clase es el molde; cada objeto es una pieza hecha con él |
| Creer que asignar un objeto a otra variable lo copia | Es la misma referencia; para una copia hay que crearla explícitamente |
| Escribir toda la lógica en `Main` y usar la clase solo para guardar datos | Pon cada operación en el método de la clase a la que corresponde |
| Una propiedad que nunca cambia pero tiene `set` | Déjala solo con `get` (o `init`) para que el compilador lo vigile |
| Olvidar `new` al crear un objeto | `Libro dune = Libro("Dune", 400);` no compila: es `new Libro(...)` |

## Para practicar

Las primeras clases se practican en [U4.2 · POO I](poo-1.md). La [tabla de equivalencias de POO](index.md#equivalencias-de-poo-en-c) del índice de la unidad resume la sintaxis de C#.
