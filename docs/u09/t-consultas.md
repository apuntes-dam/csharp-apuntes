# 9.B Consultas, modificaciones y borrados

## Las cuatro sentencias básicas

| Operación | Sentencia | Ejemplo |
|---|---|---|
| **Leer** | `SELECT` | `SELECT titulo FROM libros WHERE anio < 1700` |
| **Insertar** | `INSERT` | `INSERT INTO autores (nombre) VALUES ('Cervantes')` |
| **Modificar** | `UPDATE` | `UPDATE libros SET stock = 10 WHERE id = 1` |
| **Borrar** | `DELETE` | `DELETE FROM libros WHERE id = 4` |

!!! danger "Sin `WHERE`, afecta a TODAS las filas"
    `UPDATE libros SET stock = 0` deja **todos** los libros sin stock, y `DELETE FROM libros` **vacía la tabla entera**. Escribe siempre el `WHERE` primero y, antes de ejecutar un `UPDATE` o un `DELETE`, prueba su `WHERE` con un `SELECT` para ver qué filas afectará.

## Consultar: `SELECT`

Un `SELECT` se construye por partes, siempre en este orden:

```sql
SELECT   columnas            -- qué quiero ver
FROM     tabla               -- de dónde
JOIN     otra ON condición   -- (opcional) uniendo con otra tabla
WHERE    condición           -- (opcional) qué filas
GROUP BY columna             -- (opcional) agrupar
ORDER BY columna             -- (opcional) ordenar
```

| Idea | Ejemplo |
|---|---|
| Filtrar | `WHERE anio < 1700 AND stock > 0` |
| Ordenar | `ORDER BY anio DESC` (descendente) |
| Unir tablas (**JOIN**) | `FROM libros l JOIN autores a ON a.id = l.id_autor` |
| Contar, sumar, promediar | `COUNT(*)`, `SUM(stock)`, `AVG(anio)`, `MIN(...)`, `MAX(...)` |
| Agrupar | `GROUP BY a.nombre` (una fila por autor) |

Un **`JOIN`** combina filas de dos tablas **relacionadas por una clave**: aquí, cada libro con el nombre de su autor. Sin el `JOIN` solo verías el número `id_autor`.

## Un ejemplo completo

```csharp
#:package Microsoft.Data.Sqlite@10.0.0
#:package SQLitePCLRaw.lib.e_sqlite3@2.1.14

using Microsoft.Data.Sqlite;

using var con = new SqliteConnection("Data Source=:memory:");
con.Open();
Preparar(con);

Console.WriteLine("antes de 1700:");
using (var cmd = con.CreateCommand())
{
    cmd.CommandText = "SELECT a.nombre, l.titulo FROM libros l JOIN autores a ON a.id = l.id_autor "
                    + "WHERE l.anio < @limite ORDER BY l.anio";
    cmd.Parameters.AddWithValue("@limite", 1700);
    using var lector = cmd.ExecuteReader();
    while (lector.Read())
    {
        Console.WriteLine($"  {lector.GetString(0)} — {lector.GetString(1)}");
    }
}

Console.WriteLine("libros por autor:");
using (var cmd = con.CreateCommand())
{
    cmd.CommandText = "SELECT a.nombre, COUNT(*) FROM libros l JOIN autores a ON a.id = l.id_autor "
                    + "GROUP BY a.nombre ORDER BY a.nombre";
    using var lector = cmd.ExecuteReader();
    while (lector.Read())
    {
        Console.WriteLine($"  {lector.GetString(0)}: {lector.GetInt32(1)}");
    }
}

Console.WriteLine($"stock total: {Escalar(con, "SELECT SUM(stock) FROM libros")}");

using (var cmd = con.CreateCommand())
{
    cmd.CommandText = "UPDATE libros SET stock = @stock WHERE titulo = @titulo";
    cmd.Parameters.AddWithValue("@stock", 10);
    cmd.Parameters.AddWithValue("@titulo", "Don Quijote");
    Console.WriteLine($"filas modificadas: {cmd.ExecuteNonQuery()}");
}
Console.WriteLine($"stock total tras reponer: {Escalar(con, "SELECT SUM(stock) FROM libros")}");

try
{
    Ejecutar(con, "DELETE FROM autores WHERE id = 1");
}
catch (SqliteException)
{
    Console.WriteLine("borrar el autor 1: no se puede, tiene libros");
}

using (var cmd = con.CreateCommand())
{
    cmd.CommandText = "DELETE FROM libros WHERE id = @id";
    cmd.Parameters.AddWithValue("@id", 4);
    Console.WriteLine($"filas borradas: {cmd.ExecuteNonQuery()}");
}
Console.WriteLine($"libros que quedan: {Escalar(con, "SELECT COUNT(*) FROM libros")}");

static object? Escalar(SqliteConnection con, string sql)
{
    using var cmd = con.CreateCommand();
    cmd.CommandText = sql;
    return cmd.ExecuteScalar();
}

static void Preparar(SqliteConnection con)
{
    Ejecutar(con, "CREATE TABLE autores (id INTEGER PRIMARY KEY AUTOINCREMENT, nombre TEXT NOT NULL)");
    Ejecutar(con, "CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, anio INTEGER, "
                + "stock INTEGER NOT NULL, id_autor INTEGER NOT NULL REFERENCES autores(id))");
    Ejecutar(con, "INSERT INTO autores (nombre) VALUES ('Cervantes'), ('García Márquez')");
    Ejecutar(con, "INSERT INTO libros (titulo, anio, stock, id_autor) VALUES "
                + "('Don Quijote', 1605, 3, 1), ('Novelas ejemplares', 1613, 2, 1), "
                + "('Cien años de soledad', 1967, 5, 2), ('El coronel no tiene quien le escriba', 1961, 1, 2)");
}

static void Ejecutar(SqliteConnection con, string sql)
{
    using var cmd = con.CreateCommand();
    cmd.CommandText = sql;
    cmd.ExecuteNonQuery();
}
```

Salida:

```text
antes de 1700:
  Cervantes — Don Quijote
  Cervantes — Novelas ejemplares
libros por autor:
  Cervantes: 2
  García Márquez: 2
stock total: 11
filas modificadas: 1
stock total tras reponer: 18
borrar el autor 1: no se puede, tiene libros
filas borradas: 1
libros que quedan: 3
```

Qué enseña este programa:

* **Consulta con unión y parámetro**: los libros anteriores a 1700 con el nombre de su autor, ordenados por año.
* **Agrupación**: cuántos libros tiene cada autor (`GROUP BY` con `COUNT`).
* **Un valor suelto**: la suma del stock con `SUM`.
* **`UPDATE`**: repone el stock de un libro; el programa muestra **cuántas filas** se han modificado (si fuera 0, el `WHERE` no encontró nada).
* **Una restricción que protege los datos**: borrar el autor 1 **falla**, porque todavía tiene libros. La base de datos impide dejar libros apuntando a un autor que ya no existe.
* **`DELETE`**: borra un libro concreto con su `WHERE`.

## Leer los resultados en C&#35;

| Necesito... | En C# |
|---|---|
| Ejecutar una consulta y recorrer las filas | `using var lector = cmd.ExecuteReader();` y `while (lector.Read()) { ... }` |
| Leer una columna de una fila | `lector.GetString(0)`, `lector.GetInt32(1)`, o por nombre: `lector.GetString(lector.GetOrdinal("titulo"))`. **Las posiciones empiezan en 0** (en JDBC, el equivalente de Java, empiezan en 1) |
| Sentencias que modifican (`INSERT`, `UPDATE`, `DELETE`) | `cmd.ExecuteNonQuery()` |
| Cuántas filas se han modificado | es lo que **devuelve** `ExecuteNonQuery()` |
| Un único valor | `cmd.ExecuteScalar()` (devuelve `object?`: `COUNT` y `SUM` dan un `long`) |
| Un valor que puede ser `NULL` | `lector.IsDBNull(i)` antes de leerlo: `GetString` sobre un `NULL` lanza una excepción |

## Borrar datos relacionados

`Microsoft.Data.Sqlite` **activa las claves foráneas** al abrir cada conexión, por eso el ejemplo anterior rechaza el borrado del autor con libros (el error es una `SqliteException`). No es lo normal en SQLite en general, que las deja desactivadas salvo que se pida con `PRAGMA foreign_keys = ON`. Se puede desactivar a propósito con `Foreign Keys=False` en la cadena de conexión, y entonces la base de datos acepta datos incoherentes:

```csharp
#:package Microsoft.Data.Sqlite@10.0.0
#:package SQLitePCLRaw.lib.e_sqlite3@2.1.14

using Microsoft.Data.Sqlite;

using var con = new SqliteConnection("Data Source=:memory:;Foreign Keys=False");
con.Open();
Ejecutar(con, "CREATE TABLE autores (id INTEGER PRIMARY KEY, nombre TEXT)");
Ejecutar(con, "CREATE TABLE libros (id INTEGER PRIMARY KEY, titulo TEXT, id_autor INTEGER REFERENCES autores(id))");
Ejecutar(con, "INSERT INTO libros (titulo, id_autor) VALUES ('Fantasma', 99)");
Console.WriteLine("con Foreign Keys=False, un libro con un autor que no existe SÍ se inserta");

static void Ejecutar(SqliteConnection con, string sql)
{
    using var cmd = con.CreateCommand();
    cmd.CommandText = sql;
    cmd.ExecuteNonQuery();
}
```

Salida:

```text
con Foreign Keys=False, un libro con un autor que no existe SÍ se inserta
```

Si intentas borrar una fila a la que **otras filas apuntan**, la base de datos rechaza la operación. Hay tres formas de resolverlo:

| Estrategia | Cómo | Cuándo |
|---|---|---|
| **Borrar primero lo dependiente** | `DELETE` de los libros del autor y después `DELETE` del autor | Lo más claro y explícito |
| **Borrado en cascada** | Declarar la clave con `ON DELETE CASCADE`: al borrar el autor se borran solos sus libros | Cuando los datos hijos no tienen sentido sin el padre. **Peligroso** si se usa sin pensar |
| **No permitir borrar** | Dejar el error y avisar al usuario | Cuando el borrado indica un fallo de lógica |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| `UPDATE` o `DELETE` sin `WHERE` | Escribir el `WHERE` antes que nada y probarlo con un `SELECT` |
| Esperar un orden sin `ORDER BY` | Sin él, el orden **no está garantizado** |
| Leer una columna con el índice equivocado | En ADO.NET empiezan en **0** |
| Leer un `NULL` con `GetString` o `GetInt32` | Comprobar antes con `IsDBNull` |
| Intentar borrar un registro del que dependen otros | Borrar antes lo dependiente, o usar cascada con cuidado |
| Pegar valores dentro del SQL | Parámetros con nombre (ver [9.C](t-buenas-practicas.md)) |

## Para practicar

Haz los ejercicios de [U9.2 · Consultas, borrados y modificaciones](consultas.md). Fíjate en las estrategias de borrado de la tabla anterior cuando llegues al ejercicio de las eliminaciones. [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) compara el código de cada lenguaje.
