# 4.D Colecciones de objetos y miembros estáticos

## Objetos que contienen objetos

Los objetos se pueden **combinar**: un curso **tiene** alumnos, una biblioteca **tiene** libros, una persona **tiene** cuentas. Es la relación de **composición** («tiene un»), y se resuelve guardando los objetos dentro de otro, normalmente en una **lista**.

```text
Curso ◆────── 0..n ──────▶ Alumno
(guarda una lista)         (nombre, nota)
```

```csharp
var curso = new Curso();
curso.Matricular(new Alumno("Ana", 4));
curso.Matricular(new Alumno("Marta", 8));
curso.Matricular(new Alumno("Luis", 6));
Console.WriteLine($"Matriculados: {curso.Total}");
Console.WriteLine($"Media del curso: {curso.Media()}");
Console.WriteLine($"Mejor alumno: {curso.Mejor()}");
Console.WriteLine($"Aprobados: {string.Join(", ", curso.Aprobados().Select(a => a.Nombre))}");
try
{
    new Alumno("Error", 11);
}
catch (ArgumentException e)
{
    Console.WriteLine($"Error: {e.Message}");
}
Console.WriteLine($"Alumnos creados: {Alumno.Creados}");

class Alumno
{
    public string Nombre { get; }
    public int Nota { get; }
    public bool Aprobado => Nota >= 5;

    public static int Creados { get; private set; }

    public Alumno(string nombre, int nota)
    {
        if (!NotaValida(nota))
        {
            throw new ArgumentException("nota fuera de rango");
        }
        Nombre = nombre;
        Nota = nota;
        Creados++;
    }

    public static bool NotaValida(int nota) => nota >= 0 && nota <= 10;

    public override string ToString() => $"{Nombre} ({Nota})";
}

class Curso
{
    private readonly List<Alumno> _alumnos = new();

    public int Total => _alumnos.Count;

    public void Matricular(Alumno alumno)
    {
        _alumnos.Add(alumno);
    }

    public double Media()
    {
        int suma = 0;
        foreach (Alumno a in _alumnos)
        {
            suma += a.Nota;
        }
        return (double)suma / _alumnos.Count;
    }

    public Alumno Mejor()
    {
        Alumno mejor = _alumnos[0];
        foreach (Alumno a in _alumnos)
        {
            if (a.Nota > mejor.Nota)
            {
                mejor = a;
            }
        }
        return mejor;
    }

    public List<Alumno> Aprobados() => _alumnos.Where(a => a.Aprobado).ToList();
}
```

Salida:

```text
Matriculados: 3
Media del curso: 6
Mejor alumno: Marta (8)
Aprobados: Marta, Luis
Error: nota fuera de rango
Alumnos creados: 3
```

Qué conviene copiar de este diseño:

* **El curso decide qué se puede hacer con su lista.** Ofrece `Matricular`, `Media`, `Mejor` y `Aprobados`, pero **no entrega la lista** para que cualquiera la modifique. La lista es **`private`**: desde fuera solo se puede matricular o consultar.
* **Cada clase se ocupa de lo suyo.** `Alumno` sabe si está aprobado; `Curso` sabe calcular la media y buscar al mejor. El programa principal solo **orquesta**.
* **Los recorridos son los de siempre** (acumular una suma, buscar un máximo, filtrar), solo que ahora recorren objetos y consultan sus propiedades.
* **Las reglas del objeto, en el objeto**: la nota se valida en el constructor de `Alumno`, así un alumno con nota 11 no llega a existir (y no se cuenta en `Creados`).

!!! tip "Devolver la lista sin regalarla"
    Si hace falta que otros **lean** la lista, devuélvela como `IReadOnlyList<Alumno>` (`public IReadOnlyList<Alumno> Todos => _alumnos;`): se puede recorrer y consultar, pero no tiene `Add` ni `Remove`.

## Miembros estáticos

Hasta ahora, cada atributo y cada método pertenecía a **un objeto concreto**. Un miembro **estático** pertenece a la **clase**, y es el mismo para todos los objetos. En el ejemplo hay dos:

* `Creados`: un **contador compartido** que cuenta cuántos alumnos se han creado (no tiene sentido guardarlo en cada alumno).
* `NotaValida`: una **función de utilidad** que no necesita ningún alumno para funcionar.

En C# se marcan con **`static`** y se usan con el nombre de la clase: `Alumno.Creados`, `Alumno.NotaValida(5)`. Una **clase estática** (`static class Utilidades`) solo contiene miembros estáticos y no se puede instanciar; es lo que son `Math` o `Console`.

!!! warning "No abuses de lo estático"
    Un atributo estático es **estado global**: cualquiera puede cambiarlo y los resultados dependen de lo que haya ocurrido antes en el programa, lo que complica las pruebas. Úsalo para contadores sencillos, constantes y utilidades; no para guardar los datos de la aplicación.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Devolver la lista interna para que otros la modifiquen | Ofrece métodos concretos, o devuelve `IReadOnlyList<T>` o una copia |
| Buscar el máximo empezando con un valor «inventado» (como `0`) | Empieza con el primer elemento (y comprueba que la lista no esté vacía) |
| Dividir por el tamaño de una lista vacía | Comprueba antes que hay elementos |
| Poner datos de objetos concretos en un atributo estático | Un atributo estático es para lo que es común a **todos** |
| Dividir dos `int` y esperar decimales | `(double)suma / total`: convierte antes de dividir |

## Para practicar

Las colecciones de objetos y las relaciones entre clases se practican en [U4.3 · POO II](poo-2.md). [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) muestra las mismas ideas en otro lenguaje.
