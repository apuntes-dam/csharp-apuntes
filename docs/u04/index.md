# U4 · Programación orientada a objetos

Pasar del código suelto a **clases y objetos**: atributos, métodos, constructores, validación, encapsulamiento y colecciones de objetos. Termina con un proyecto personal o en grupo.

## Teoría

* [4.A Clases y objetos](t-clases.md): atributos, métodos, constructor y referencias.
* [4.B Encapsulamiento](t-encapsulamiento.md): visibilidad, validación, propiedades e igualdad.
* [4.C Constructores y enumerados](t-constructores.md): varias formas de crear un objeto y tipos cerrados.
* [4.D Colecciones y miembros estáticos](t-colecciones.md): objetos que contienen objetos y lo que pertenece a la clase.
* [4.E Del problema al programa](t-modelar.md): cómo decidir las clases y pasar comportamiento como parámetro.

## Antes de empezar: qué debes dominar

* Clase, objeto, atributo y método.
* Constructores (principal y alternativos) y valores por defecto.
* Encapsulamiento: visibilidad y propiedades de solo lectura.
* Enumerados, sobrecarga y `ToString`/igualdad.
* Colecciones de objetos y validación con excepciones.

## Ejercicios de la unidad

| Bloque | Ejercicios |
|---|---|
| [U4.1 · Repaso de las unidades 1 a 3](repaso.md) | 1 |
| [U4.2 · POO I (ejercicios 1 al 5)](poo-1.md) | 5 |
| [U4.3 · POO II (ejercicios 6 al 10)](poo-2.md) | 5 |
| [U4.4 · Robots (parte 1)](robots-1.md) | 2 |
| [U4.5 · Robots (parte 2 y reto)](robots-2.md) | 1 |
| [U4.6 · Prueba: Cafetera y Taza](prueba.md) | 2 |
| [U4.7 · Juego del ahorcado (grupos)](proyecto.md) | 1 |
| [U4.8 · Cambio de rol: explícamelo tú (grupos)](cambio-de-rol.md) | 2 |

## Equivalencias de POO en C#

| Idea | En C# |
|---|---|
| Constructor | `public Persona(string nombre, int edad) { ... }`; varios con distintos parámetros (sobrecarga) y `: this(...)` para encadenarlos |
| Valores por defecto | `public Pizza(Tamano t, string nombre = "Margarita")` (y se pueden pasar por nombre: `extras: 2`) |
| Propiedad calculada de solo lectura | `public double Imc => peso / (altura * altura);` |
| Atributo que solo cambia desde dentro | `public int Stock { get; private set; }` |
| Privado | `private` (y `protected`, `internal`, `public`) |
| Validar al crear | `if (...) throw new ArgumentException(...)` en el constructor o en el `set` |
| Herencia | `class B : A`, con `: base(...)`; para sobrescribir, `virtual` en la base y `override` en la hija |
| Clase abstracta / interfaz | `abstract class` / `interface` (por convención, con `I` delante: `IVolador`) |
| Enumerado | `enum Color { Blanco, Negro }` (sin campos ni métodos propios: se añaden con métodos de extensión) |
| Datos con igualdad por valor | `record Persona(string Nombre, int Edad);` |
| Jerarquía cerrada | `sealed` en las clases finales (sin comprobación exhaustiva en el `switch`: ver 5.D) |
| Extensión | `static List<string> Filtrar(this List<string> lista, ...)` en una clase estática |
| Representación en texto | `public override string ToString()` |
| Igualdad | `Equals` y `GetHashCode` (o un `record`) |

## Antes de pasar a los ejercicios

Cuando hayas leído y practicado la teoría, marca la casilla para **desbloquear** los ejercicios de esta unidad:

<div class="ej-check" data-unit="u04"></div>
