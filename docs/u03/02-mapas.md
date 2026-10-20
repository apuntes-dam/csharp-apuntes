# 3.2 Mapas (diccionarios)

Un **mapa** (en C#, `Dictionary<TClave, TValor>`) guarda **pares clave → valor** y permite encontrar un valor a partir de su clave sin recorrer nada. Las claves **no se repiten**.

## Crear, consultar, modificar y borrar

```csharp
var edades = new Dictionary<string, int>
{
    ["Ana"] = 25,
    ["Luis"] = 30,
};

edades["Eva"] = 22;                  // añade una clave nueva
edades["Ana"] = 26;                  // si la clave existe, cambia su valor
Console.WriteLine($"Ana: {edades["Ana"]}");
Console.WriteLine($"tamaño: {edades.Count}");
Console.WriteLine($"¿está Luis? {edades.ContainsKey("Luis")}");

// consulta segura: no falla si la clave no existe
if (edades.TryGetValue("Pedro", out int edad))
{
    Console.WriteLine($"Pedro: {edad}");
}
else
{
    Console.WriteLine("Pedro no está");
}
Console.WriteLine($"con valor por defecto: {edades.GetValueOrDefault("Pedro", -1)}");

edades.Remove("Luis");
foreach (var (nombre, años) in edades)
{
    Console.WriteLine($"{nombre} -> {años}");
}
```

Salida:

```text
Ana: 26
tamaño: 3
¿está Luis? True
Pedro no está
con valor por defecto: -1
Ana -> 26
Eva -> 22
```

* `mapa[clave] = valor` añade **o reemplaza**; `mapa.Add(clave, valor)` solo añade y **lanza una excepción** si la clave ya existe.
* `mapa[clave]` lanza `KeyNotFoundException` si la clave no está. Para consultar con seguridad: `TryGetValue` o `GetValueOrDefault`.
* `foreach (var (clave, valor) in mapa)` recorre los pares.

## Contar con un mapa

Es el uso más típico: cuántas veces aparece cada cosa.

```csharp
string texto = "la casa de la montaña la casa";
var cuenta = new Dictionary<string, int>();

foreach (string palabra in texto.Split(' '))
{
    cuenta[palabra] = cuenta.GetValueOrDefault(palabra) + 1;
}

foreach (var (palabra, veces) in cuenta.OrderByDescending(p => p.Value))
{
    Console.WriteLine($"{palabra}: {veces}");
}
```

Salida:

```text
la: 3
casa: 2
de: 1
montaña: 1
```

`GetValueOrDefault(palabra)` devuelve 0 si la palabra aún no está, así que `+ 1` funciona en la primera vez y en las siguientes.

## Claves, valores y mapas de listas

```csharp
var notas = new Dictionary<string, List<int>>
{
    ["Ana"] = new() { 7, 9 },
    ["Luis"] = new() { 5 },
};
notas["Luis"].Add(8);

Console.WriteLine("claves: " + string.Join(", ", notas.Keys));
foreach (var (alumno, lista) in notas)
{
    Console.WriteLine($"{alumno}: {string.Join(", ", lista)} (media {lista.Average()})");
}

// un mapa dentro de otro puede mezclar tipos si el valor es object
var cliente = new Dictionary<string, object>
{
    ["nombre"] = "Ana",
    ["edad"] = 25,
    ["preferente"] = true,
};
Console.WriteLine($"{cliente["nombre"]}, {cliente["edad"]}, {cliente["preferente"]}");
```

Salida:

```text
claves: Ana, Luis
Ana: 7, 9 (media 8)
Luis: 5, 8 (media 6,5)
Ana, 25, True
```

## ¿Mapa, lista o conjunto?

| Necesitas... | Usa... |
|---|---|
| Valores **en orden**, por posición | Lista |
| Buscar un valor por una **clave** | Mapa |
| Saber si algo **está o no**, sin repetidos | Conjunto |
| Contar apariciones | Mapa (clave → número) |

!!! note "El orden de un `Dictionary`"
    Un `Dictionary` no garantiza ningún orden al recorrerlo. Si necesitas que las claves salgan ordenadas, ordénalas al recorrer (`OrderBy`) o usa `SortedDictionary`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Leer una clave que no existe con `mapa[clave]` | Lanza `KeyNotFoundException`: usa `TryGetValue` o `GetValueOrDefault` |
| `Add` con una clave repetida | Lanza `ArgumentException`: usa `mapa[clave] = valor` |
| Añadir claves al mapa mientras lo recorres | Lanza `InvalidOperationException`: recorre una copia (`.ToList()`) |
| Esperar un orden concreto | Ordena al recorrer o usa `SortedDictionary` |

## Para practicar

Haz los [ejercicios 3.2 de mapas](mapas.md).
