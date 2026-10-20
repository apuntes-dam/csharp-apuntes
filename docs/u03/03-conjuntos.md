# 3.3 Conjuntos

Un **conjunto** (`HashSet<T>`) guarda valores **sin repetir** y responde muy rápido a «¿está este valor?». No tiene posiciones: no se puede pedir «el tercero».

## Operaciones

```csharp
var a = new HashSet<int> { 1, 2, 3, 4 };
var b = new HashSet<int> { 3, 4, 5 };

Console.WriteLine($"añadir 2 (ya estaba): {a.Add(2)}");
Console.WriteLine($"añadir 9 (nuevo):     {a.Add(9)}");
Console.WriteLine($"¿está el 3? {a.Contains(3)}");
a.Remove(9);

// operaciones con conjuntos, SIN modificar a ni b
Console.WriteLine("unión:        " + string.Join(", ", a.Union(b).OrderBy(x => x)));
Console.WriteLine("intersección: " + string.Join(", ", a.Intersect(b).OrderBy(x => x)));
Console.WriteLine("solo en a:    " + string.Join(", ", a.Except(b).OrderBy(x => x)));
Console.WriteLine("solo en b:    " + string.Join(", ", b.Except(a).OrderBy(x => x)));

// versiones que MODIFICAN el conjunto
var c = new HashSet<int>(a);
c.IntersectWith(b);
Console.WriteLine("c tras IntersectWith: " + string.Join(", ", c.OrderBy(x => x)));
```

Salida:

```text
añadir 2 (ya estaba): False
añadir 9 (nuevo):     True
¿está el 3? True
unión:        1, 2, 3, 4, 5
intersección: 3, 4
solo en a:    1, 2
solo en b:    5
c tras IntersectWith: 3, 4
```

* `Add` devuelve `true` si el valor era nuevo y `false` si ya estaba.
* `Union`, `Intersect` y `Except` (de LINQ) devuelven un resultado nuevo. `UnionWith`, `IntersectWith` y `ExceptWith` **modifican** el conjunto.
* `.OrderBy(x => x)` solo está ahí para que la salida salga ordenada: un `HashSet` no garantiza el orden.

## Quitar repetidos

```csharp
var frutas = new List<string> { "pera", "uva", "pera", "manzana", "uva" };

var sinRepetir = frutas.Distinct().ToList();           // conserva el orden
Console.WriteLine(string.Join(", ", sinRepetir));

var conjunto = new HashSet<string>(frutas);
Console.WriteLine($"{frutas.Count} elementos, {conjunto.Count} distintos");
```

Salida:

```text
pera, uva, manzana
5 elementos, 3 distintos
```

## ¿Lista o conjunto?

| Necesitas... | Usa... |
|---|---|
| Orden y posiciones, o permitir repetidos | Lista |
| Que no haya repetidos y preguntar «¿está?» | Conjunto |
| Unión, intersección o diferencia | Conjunto |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Pedir un elemento por posición | Un conjunto no tiene posiciones: conviértelo en lista |
| Esperar un orden | No hay orden garantizado; usa `SortedSet<T>` si lo necesitas |
| Esperar que `Union` modifique el conjunto | Devuelve uno nuevo; para modificar usa `UnionWith` |
| Un conjunto de objetos propios que no detecta repetidos | La clase debe definir su igualdad (`Equals` y `GetHashCode`, o ser un `record`) |

## Para practicar

Haz los [ejercicios 3.3 de conjuntos](conjuntos.md).
