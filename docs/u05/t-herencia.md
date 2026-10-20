# 5.A Herencia

La **herencia** permite crear una clase nueva **a partir de otra ya existente**, aprovechando todo lo que esta tiene y añadiendo o cambiando lo que haga falta. Se usa cuando entre dos clases hay una relación **«es un»**: un perro **es un** animal; un gato **es un** animal.

| Término | Significa | Ejemplo |
|---|---|---|
| **Superclase** (o clase base, o padre) | La clase de la que se hereda | `Animal` |
| **Subclase** (o clase derivada, o hija) | La clase que hereda y la especializa | `Perro`, `Gato` |
| **Sobrescribir** (*override*) | Redefinir un método heredado para que haga otra cosa | `Hablar()` |

## Un ejemplo

```csharp
var animales = new List<Animal> { new Perro("Rex"), new Gato("Misi"), new Animal("Bicho") };
foreach (Animal a in animales)
{
    Console.WriteLine(a.Presentarse());
}
foreach (Animal a in animales)
{
    if (a is Perro perro)
    {
        Console.WriteLine(perro.TraerPelota());
    }
}

class Animal
{
    public string Nombre { get; }

    public Animal(string nombre)
    {
        Nombre = nombre;
    }

    public virtual string Hablar() => "...";

    public virtual string Presentarse() => $"{Nombre} dice: {Hablar()}";
}

class Perro : Animal
{
    public Perro(string nombre) : base(nombre) { }

    public override string Hablar() => "Guau";

    public string TraerPelota() => $"{Nombre} trae la pelota";
}

class Gato : Animal
{
    public Gato(string nombre) : base(nombre) { }

    public override string Hablar() => "Miau";

    public override string Presentarse() => $"{base.Presentarse()} (y se hace el distraído)";
}
```

Salida:

```text
Rex dice: Guau
Misi dice: Miau (y se hace el distraído)
Bicho dice: ...
Rex trae la pelota
```

Qué ocurre aquí:

* `Perro` y `Gato` **no repiten** la propiedad `Nombre` ni el método `Presentarse`: los reciben de `Animal`.
* Cada subclase **sobrescribe `Hablar()`** para dar su propio sonido; `Animal` tiene una versión genérica (`...`), y `Bicho`, que es un `Animal` normal, la usa.
* `Gato` además sobrescribe `Presentarse()` y **reutiliza** la versión del padre con `base`, añadiéndole algo.
* `Perro` añade un método que `Animal` no tiene (`TraerPelota`). Para llamarlo hay que **comprobar el tipo** antes (`a is Perro perro`, que además convierte la variable), porque la lista contiene animales de todo tipo.
* Las tres líneas se imprimen con **la misma llamada** (`a.Presentarse()`), pero el resultado depende de **qué objeto concreto** hay detrás. Eso se llama **polimorfismo** y se explica en el [siguiente apartado](t-abstractas.md).

## Cómo se escribe en C#

| Necesito... | En C# |
|---|---|
| Heredar | `class Perro : Animal`. Las clases se pueden heredar **por defecto** (para impedirlo, `sealed`) |
| Constructor del padre | Se llama tras los dos puntos: `public Perro(string nombre) : base(nombre)`. Los constructores **no se heredan** |
| Sobrescribir un método | El de la base debe ser **`virtual`** (o `abstract`) y el de la hija lleva **`override`** (obligatorio). Sin `virtual`, el método **no se puede sobrescribir** |
| Llamar a la versión del padre | `base.Presentarse()` |
| Comprobar el tipo | `a is Perro` o `a is Perro perro` (declara la variable ya convertida) |
| Impedir que hereden de ti | `sealed class Perro : Animal` |
| ¿Cuántos padres? | **Uno** (herencia simple); se completa con interfaces |

!!! warning "Si olvidas `virtual`, la hija «oculta» en vez de sobrescribir"
    Si en la base escribes `public string Hablar()` (sin `virtual`) y en la hija `public string Hablar()`, el compilador avisa con una **advertencia** (*hides inherited member*): el método de la hija **no sustituye** al del padre. Con una variable de tipo `Animal` se ejecutaría siempre el del padre. Usa siempre `virtual` + `override`.

## ¿Herencia o composición?

La herencia es muy cómoda, pero crea un vínculo fuerte: si la clase base cambia, todas las hijas se ven afectadas. Antes de heredar, pregúntate si la relación es de verdad «es un».

| Relación | Se resuelve con | Ejemplo |
|---|---|---|
| **«es un»** (un perro es un animal) | Herencia | `class Perro : Animal` |
| **«tiene un»** (un coche tiene un motor) | **Composición**: una propiedad con otro objeto (ver [4.D](../u04/t-colecciones.md)) | `Coche` con una propiedad `Motor` |
| **«sabe hacer»** (un pato sabe volar) | **Interfaz** (ver [5.C](t-interfaces.md)) | `Pato` implementa `IVolador` |

!!! warning "No heredes solo para reutilizar código"
    Si `Pila` heredara de `List<T>` solo para aprovechar `Add`, una pila **no es** una lista: acabaría ofreciendo operaciones que no tienen sentido en ella. En ese caso, la pila **tiene** una lista dentro (composición). Regla práctica: ante la duda, **composición**.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Heredar cuando la relación no es «es un» | Probar la frase «un A es un B» en voz alta |
| Jerarquías muy profundas (A → B → C → D → E) | Mantenerlas cortas; más de tres niveles suele ser mal signo |
| Llamar a un método de la subclase desde una variable del tipo base | Comprobar el tipo antes (`is`), o replantear el diseño |
| Olvidar inicializar la parte heredada en el constructor | Llamar siempre a `base(...)` con lo que necesite |
| Olvidar `virtual` en la base o `override` en la hija | El método no se sobrescribe: el compilador lo avisa con una advertencia o un error |

## Para practicar

Los ejercicios de herencia y sobrescritura están en [U5.1](herencia.md) (por ejemplo, los de artículos y vehículos). Compara cómo se escribe en otro lenguaje con [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
