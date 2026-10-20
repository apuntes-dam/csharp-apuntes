# 5.B Polimorfismo y clases abstractas

## Polimorfismo

**Polimorfismo** significa «muchas formas»: una misma llamada se comporta de forma distinta según el **objeto real** sobre el que se hace. En el ejemplo de la [herencia](t-herencia.md), `a.Presentarse()` imprimía algo diferente para un perro, un gato y un animal genérico.

Para entenderlo hay que distinguir dos tipos:

* El **tipo declarado** de la variable, que es lo que ve el compilador (`Animal`).
* El **tipo real** del objeto, que se decide al crearlo (`Perro`, `Gato`...).

Al llamar a un método **sobrescrito** (`virtual` + `override`), se ejecuta la versión del **tipo real**. Gracias a eso se puede escribir código que trabaja con el tipo general y funciona con todos los casos presentes y **futuros**: si mañana se añade `Loro`, el bucle que recorre los animales no cambia.

## Clases abstractas

A veces una clase base solo tiene sentido **como molde**: un «instrumento» genérico no suena a nada, pero sí una guitarra o un tambor. Una **clase abstracta** es una clase que **no se puede instanciar** y que obliga a sus subclases a completar lo que falta.

Se marca la clase con **`abstract`** y cada método sin cuerpo con **`abstract`** (`public abstract string Sonido();`). No hace falta `virtual`: lo abstracto ya está pensado para heredarse, y la hija lo implementa con `override`. No se puede hacer `new` de una clase abstracta.

```csharp
var orquesta = new List<Instrumento> { new Guitarra(), new Tambor(), new Piano() };
foreach (Instrumento instrumento in orquesta)
{
    Console.WriteLine(instrumento.Tocar());
}
Console.WriteLine($"en la orquesta hay {orquesta.Count} instrumentos");

abstract class Instrumento
{
    public string Nombre { get; }

    protected Instrumento(string nombre)
    {
        Nombre = nombre;
    }

    public abstract string Sonido();                      // abstracto: cada subclase lo define

    public string Tocar() => $"{Nombre}: {Sonido()}";     // concreto: usa el abstracto
}

class Guitarra : Instrumento
{
    public Guitarra() : base("Guitarra") { }
    public override string Sonido() => "rasgueo de cuerdas";
}

class Tambor : Instrumento
{
    public Tambor() : base("Tambor") { }
    public override string Sonido() => "¡pum pum!";
}

class Piano : Instrumento
{
    public Piano() : base("Piano") { }
    public override string Sonido() => "notas de teclado";
}
```

Salida:

```text
Guitarra: rasgueo de cuerdas
Tambor: ¡pum pum!
Piano: notas de teclado
en la orquesta hay 3 instrumentos
```

Fíjate en el diseño:

* `Sonido()` es **abstracto**: la clase base sabe que todo instrumento suena, pero no *cómo*, así que lo deja sin escribir.
* `Tocar()` es **concreto** y ya está escrito, pero **usa** `Sonido()`. Es una idea muy útil: la base fija **el esquema** y cada subclase rellena el hueco. Se la conoce como método plantilla.
* El constructor de la base es `protected`: solo lo pueden llamar las hijas (con `base(...)`), porque nadie puede crear un `Instrumento` suelto.
* El código que recorre la orquesta solo conoce `Instrumento`; no le importa cuántos tipos hay.

!!! tip "¿Cuándo hacer una clase abstracta?"
    Cuando la clase base **comparte código o datos** (aquí, el nombre y `Tocar`) y además **obliga** a las hijas a definir algo. Si solo hay que fijar un contrato, sin código común, una [interfaz](t-interfaces.md) es más flexible.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Intentar crear un objeto de una clase abstracta | Crear un objeto de una subclase concreta |
| Que una subclase olvide implementar un método abstracto | El compilador lo avisa: hay que implementarlo todo (o que la hija también sea `abstract`) |
| Poner demasiado en la clase abstracta | Solo lo que es **común** a todas las hijas |
| Usar herencia solo para compartir código | Valorar composición o funciones auxiliares |

## Para practicar

Haz los ejercicios de clases abstractas y polimorfismo de [U5.1](herencia.md) (los primeros). Y para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
