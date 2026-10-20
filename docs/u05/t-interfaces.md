# 5.C Interfaces

Una **interfaz** es un **contrato**: una lista de métodos que una clase **promete ofrecer**, sin decir cómo. Cualquier clase que cumpla el contrato puede usarse allí donde se pida la interfaz, sin importar de qué clase venga.

La herencia responde a «**qué es** el objeto» (un pato **es un** animal). La interfaz responde a «**qué sabe hacer**» (un pato **sabe volar** y **sabe nadar**). Y eso es independiente: un avión también vuela, aunque no tenga nada que ver con un animal.

En C# se declara con **`interface`** y, por convención, su nombre **empieza por `I`** (`IVolador`, `INadador`). Las clases la cumplen poniéndola tras los dos puntos, y una clase puede cumplir **todas las que quiera**. Una interfaz puede traer métodos **con cuerpo** (la implementación por defecto, como `Aterrizar()` en el ejemplo) y propiedades abstractas.

```csharp
var voladores = new List<IVolador> { new Pato(), new Avion() };
foreach (IVolador v in voladores)
{
    Console.WriteLine($"{v.Volar()} y {v.Aterrizar()}");
}
var nadadores = new List<INadador> { new Pato(), new Pez() };
foreach (INadador n in nadadores)
{
    Console.WriteLine(n.Nadar());
}

interface INadador
{
    string Nadar();
}

interface IVolador
{
    string Volar();

    string Aterrizar() => "aterriza suavemente";      // implementación por defecto
}

class Pato : IVolador, INadador
{
    public string Volar() => "El pato bate las alas";

    public string Nadar() => "El pato chapotea";
}

class Avion : IVolador
{
    public string Volar() => "El avión despega";

    public string Aterrizar() => "aterriza con tren de aterrizaje";
}

class Pez : INadador
{
    public string Nadar() => "El pez ondula";
}
```

Salida:

```text
El pato bate las alas y aterriza suavemente
El avión despega y aterriza con tren de aterrizaje
El pato chapotea
El pez ondula
```

Qué enseña este ejemplo:

* **`Pato` cumple dos contratos** (`IVolador` e `INadador`). Una clase solo puede tener **un** padre, pero puede cumplir **muchas** interfaces.
* **`Avion` y `Pato` no se parecen en nada**, pero ambos son `IVolador`: una lista de voladores los mezcla sin problema. Es polimorfismo, ahora **sin herencia**.
* `Pato` usa tal cual el `Aterrizar()` por defecto; `Avion` lo **sustituye** con el suyo.
* Quien recibe un `IVolador` solo puede llamar a lo que **el contrato** promete, aunque el objeto real tenga más cosas.

!!! warning "El método por defecto solo se ve a través de la interfaz"
    Con `var pato = new Pato(); pato.Aterrizar();` **no compila**: el método por defecto pertenece a la interfaz, no a la clase. Hay que usar una variable del tipo de la interfaz: `IVolador v = pato; v.Aterrizar();`.

## Clase abstracta o interfaz

| | Clase abstracta | Interfaz |
|---|---|---|
| Relación | «es un» (hereda) | «sabe hacer» (cumple un contrato) |
| ¿Cuántas puede tener una clase? | **Una** | **Todas las que quiera** |
| ¿Guarda datos (campos)? | Sí | No |
| ¿Puede traer código? | Sí | Solo comportamiento por defecto |
| Cuándo usarla | Familia de clases con **código y datos comunes** | Capacidad que pueden tener **clases sin relación** |

!!! tip "Programa contra el contrato"
    Escribe métodos que reciban la interfaz (`IVolador`) y no la clase concreta (`Pato`): serán reutilizables con cualquier clase futura que cumpla el contrato. Es una de las ideas más importantes del diseño con objetos, y la base de los [principios SOLID](../u06/index.md).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Interfaces enormes con métodos que muchas clases no necesitan | Interfaces pequeñas, con **una** capacidad cada una |
| Usar una interfaz cuando hay mucho código común | Clase abstracta |
| No implementar todos los métodos (sin cuerpo por defecto) | El compilador lo avisa |
| Poner estado dentro de la interfaz | Un contrato describe **qué hacer**, no qué guardar |
| Olvidar que los miembros de una clase que implementa la interfaz deben ser `public` | Si no, el compilador da error |

## Para practicar

Haz los ejercicios de interfaces de [U5.1](herencia.md) (dispositivos y notificaciones). Compara cómo se escribe en otro lenguaje con [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
