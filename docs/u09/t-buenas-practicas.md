# 9.C Buenas prácticas: pool, seguridad, transacciones y DAO

## Pool de conexiones

Abrir una conexión con un servidor de bases de datos es **lento y costoso** (red, autenticación). Un programa que abre y cierra una conexión por cada operación desperdicia mucho tiempo. Un **pool de conexiones** mantiene varias conexiones **ya abiertas** y las **presta** a quien las necesita:

```text
programa ──pide conexión──▶  [ pool: 🔌 🔌 🔌 🔌 ]  ──▶  servidor de base de datos
         ◀──la devuelve───
```

En .NET **no hace falta montar el pool a mano**: los proveedores de ADO.NET lo traen incluido y activado por defecto. Cada vez que haces `new SqlConnection(cadena)` y `Open()`, el proveedor busca una conexión libre con **la misma cadena de conexión** en su pool; y cuando haces `Close()` o sales del `using`, la conexión **vuelve al pool** en lugar de cerrarse de verdad. Por eso se recomienda **abrir tarde y cerrar pronto**: abrir la conexión justo antes de usarla y soltarla enseguida. Se ajusta con la cadena de conexión (`Max Pool Size`, `Pooling=False`...). Con SQLite, que es un archivo local, el pool existe pero aporta poco: lo notaste en [9.A](t-conexion.md), porque mantiene el archivo abierto después de cerrar.

## Seguridad: la inyección SQL

La **inyección SQL** ocurre cuando se construye una consulta **pegando texto del usuario** dentro del SQL. Si el usuario escribe SQL en lugar de un dato normal, **cambia el significado de la consulta**: puede ver datos que no debe, saltarse un inicio de sesión o borrar tablas. Es una de las vulnerabilidades más graves y frecuentes de las aplicaciones reales.

```csharp
#:package Microsoft.Data.Sqlite@10.0.0
#:package SQLitePCLRaw.lib.e_sqlite3@2.1.14

using Microsoft.Data.Sqlite;

using var con = new SqliteConnection("Data Source=:memory:");
con.Open();
Ejecutar(con, "CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, stock INTEGER NOT NULL)");
Ejecutar(con, "INSERT INTO libros (titulo, stock) VALUES ('Don Quijote', 3), ('Novelas ejemplares', 2), "
            + "('Cien años de soledad', 5), ('El coronel no tiene quien le escriba', 1)");

string maliciosa = "x' OR '1'='1";
Console.WriteLine($"[inseguro] Don Quijote -> {BuscarInseguro(con, "Don Quijote").Count} resultado(s)");
Console.WriteLine($"[inseguro] {maliciosa} -> {BuscarInseguro(con, maliciosa).Count} resultado(s)");
Console.WriteLine($"[seguro] Don Quijote -> {BuscarSeguro(con, "Don Quijote").Count} resultado(s)");
Console.WriteLine($"[seguro] {maliciosa} -> {BuscarSeguro(con, maliciosa).Count} resultado(s)");

static List<string> BuscarInseguro(SqliteConnection con, string titulo)
{
    using var cmd = con.CreateCommand();
    cmd.CommandText = $"SELECT titulo FROM libros WHERE titulo = '{titulo}'";      // ¡NUNCA así!
    return Leer(cmd);
}

static List<string> BuscarSeguro(SqliteConnection con, string titulo)
{
    using var cmd = con.CreateCommand();
    cmd.CommandText = "SELECT titulo FROM libros WHERE titulo = @titulo";
    cmd.Parameters.AddWithValue("@titulo", titulo);
    return Leer(cmd);
}

static List<string> Leer(SqliteCommand cmd)
{
    var resultado = new List<string>();
    using var lector = cmd.ExecuteReader();
    while (lector.Read())
    {
        resultado.Add(lector.GetString(0));
    }
    return resultado;
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
[inseguro] Don Quijote -> 1 resultado(s)
[inseguro] x' OR '1'='1 -> 4 resultado(s)
[seguro] Don Quijote -> 1 resultado(s)
[seguro] x' OR '1'='1 -> 0 resultado(s)
```

La cadena `x' OR '1'='1` hace que la consulta insegura quede como:

```sql
SELECT titulo FROM libros WHERE titulo = 'x' OR '1'='1'
```

Como `'1'='1'` siempre es cierto, la consulta **devuelve todos los libros**. Con una **consulta parametrizada**, esa misma cadena se trata como un **título que no existe**, porque el valor viaja **aparte del SQL** y nunca se interpreta como código.

En ADO.NET: `cmd.CommandText = "... WHERE titulo = @titulo"; cmd.Parameters.AddWithValue("@titulo", titulo);`: los valores se asignan por nombre.

!!! danger "Regla de oro"
    **Nunca** construyas SQL concatenando o interpolando datos que no controles (todo lo que escribe el usuario, lo que llega de un formulario o de la red). Fíjate en que la cadena interpolada `$"... '{titulo}'"` es **igual de peligrosa** que la concatenación con `+`. Usa **siempre** parámetros. Un parámetro sirve para **valores**; los nombres de tablas o columnas no se pueden parametrizar: si deben variar, elígelos de una lista fija de opciones permitidas.

## Transacciones

Una **transacción** agrupa varias operaciones para que se ejecuten **como una sola**: o se hacen **todas** o no se hace **ninguna**. Es imprescindible cuando un cambio requiere varios pasos. Prestar un libro, por ejemplo, exige **registrar el préstamo** y **descontar el stock**: si lo primero se hace y lo segundo falla, la base de datos quedaría incoherente.

```csharp
#:package Microsoft.Data.Sqlite@10.0.0
#:package SQLitePCLRaw.lib.e_sqlite3@2.1.14

using Microsoft.Data.Sqlite;

using var con = new SqliteConnection("Data Source=:memory:");
con.Open();
Ejecutar(con, "CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, stock INTEGER NOT NULL)");
Ejecutar(con, "CREATE TABLE prestamos (id INTEGER PRIMARY KEY AUTOINCREMENT, id_libro INTEGER NOT NULL, socio TEXT NOT NULL)");
Ejecutar(con, "INSERT INTO libros (titulo, stock) VALUES ('Don Quijote', 3), ('Novelas ejemplares', 2), "
            + "('Cien años de soledad', 5), ('El coronel no tiene quien le escriba', 1)");

Console.WriteLine($"préstamo 1: {Prestar(con, 4, "Ana", fallarAMitad: false)}");
Console.WriteLine($"préstamo 2: {Prestar(con, 4, "Luis", fallarAMitad: false)}");
Console.WriteLine($"préstamo 3: {Prestar(con, 3, "Eva", fallarAMitad: true)}");
Console.WriteLine($"préstamos registrados: {Escalar(con, "SELECT COUNT(*) FROM prestamos")}");
Console.WriteLine($"stock del libro 4: {Stock(con, 4)}");
Console.WriteLine($"stock del libro 3: {Stock(con, 3)}");

// Registra el préstamo y descuenta el stock como UNA sola operación.
static string Prestar(SqliteConnection con, int idLibro, string socio, bool fallarAMitad)
{
    using var transaccion = con.BeginTransaction();      // empieza la transacción
    try
    {
        if (Stock(con, idLibro) < 1)
        {
            throw new PrestamoFallido("no hay stock");
        }
        using (var cmd = con.CreateCommand())
        {
            cmd.CommandText = "INSERT INTO prestamos (id_libro, socio) VALUES (@libro, @socio)";
            cmd.Parameters.AddWithValue("@libro", idLibro);
            cmd.Parameters.AddWithValue("@socio", socio);
            cmd.ExecuteNonQuery();
        }
        if (fallarAMitad)
        {
            throw new PrestamoFallido("error simulado a mitad de la operación");
        }
        using (var cmd = con.CreateCommand())
        {
            cmd.CommandText = "UPDATE libros SET stock = stock - 1 WHERE id = @libro";
            cmd.Parameters.AddWithValue("@libro", idLibro);
            cmd.ExecuteNonQuery();
        }
        transaccion.Commit();      // todo ha ido bien: se confirma
        return "correcto";
    }
    catch (PrestamoFallido e)
    {
        transaccion.Rollback();      // algo ha fallado: se deshace TODO
        return e.Message;
    }
}

static object? Escalar(SqliteConnection con, string sql)
{
    using var cmd = con.CreateCommand();
    cmd.CommandText = sql;
    return cmd.ExecuteScalar();
}

static int Stock(SqliteConnection con, int idLibro)
{
    using var cmd = con.CreateCommand();
    cmd.CommandText = "SELECT stock FROM libros WHERE id = @libro";
    cmd.Parameters.AddWithValue("@libro", idLibro);
    return Convert.ToInt32(cmd.ExecuteScalar());
}

static void Ejecutar(SqliteConnection con, string sql)
{
    using var cmd = con.CreateCommand();
    cmd.CommandText = sql;
    cmd.ExecuteNonQuery();
}

class PrestamoFallido(string mensaje) : Exception(mensaje);
```

Salida:

```text
préstamo 1: correcto
préstamo 2: no hay stock
préstamo 3: error simulado a mitad de la operación
préstamos registrados: 1
stock del libro 4: 0
stock del libro 3: 5
```

El tercer préstamo falla **a propósito, después de haber insertado el préstamo**. Gracias a la transacción, el `Rollback` **deshace también esa inserción**: al final solo hay **un** préstamo registrado y el stock del libro 3 sigue en 5. Sin transacción, habría un préstamo «fantasma» sin descontar.

En ADO.NET se empieza con `con.BeginTransaction()`, se confirma con `Commit()` y se deshace con `Rollback()`. Si la transacción se destruye (sale del `using`) sin haber hecho `Commit`, la base de datos la **deshace sola**, lo que da una red de seguridad si te olvidas del `Rollback`.

Una transacción cumple las propiedades **ACID**:

| Propiedad | Significa |
|---|---|
| **A**tomicidad | Todo o nada |
| **C**onsistencia | Los datos siguen cumpliendo las reglas (claves, restricciones) |
| **I**slamiento | Varias operaciones simultáneas no se estorban |
| **D**urabilidad | Lo confirmado queda guardado aunque falle el equipo |

## El patrón DAO

Si el SQL está **repartido por todo el programa**, cualquier cambio en las tablas obliga a buscarlo en cien sitios, y es imposible probar la lógica sin una base de datos. El patrón **DAO** (*Data Access Object*) lo soluciona: **una clase** concentra **todo** el acceso a los datos de una tabla, y el resto del programa solo habla con ella y con **objetos**, nunca con conexiones ni con SQL.

```text
  Programa  ──▶  Servicio (reglas)  ──▶  DAO (SQL)  ──▶  Base de datos
  objetos Libro        usa objetos            convierte filas ↔ objetos
```

```csharp
#:package Microsoft.Data.Sqlite@10.0.0
#:package SQLitePCLRaw.lib.e_sqlite3@2.1.14

using Microsoft.Data.Sqlite;

using var con = new SqliteConnection("Data Source=:memory:");
con.Open();
Preparar(con);
var dao = new LibroDao(con);
var catalogo = new Catalogo(dao);

Libro nuevo = dao.Guardar(new Libro(null, "Dune", 1965, 2, 3));
Console.WriteLine($"guardado: {nuevo}");
Console.WriteLine($"buscar(5): {dao.Buscar(5)?.Titulo}");
Console.WriteLine($"buscar(99): {(dao.Buscar(99) is null ? "no existe" : "existe")}");
Console.WriteLine($"del autor Cervantes: {string.Join(", ", dao.PorAutor("Cervantes").Select(l => l.Titulo))}");
catalogo.PrestarUno(5);
Console.WriteLine($"stock de Dune tras prestar uno: {dao.Buscar(5)?.Stock}");
Console.WriteLine($"total de libros: {dao.Todos().Count}");

static void Preparar(SqliteConnection con)
{
    Ejecutar(con, "CREATE TABLE autores (id INTEGER PRIMARY KEY AUTOINCREMENT, nombre TEXT NOT NULL)");
    Ejecutar(con, "CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, anio INTEGER, "
                + "stock INTEGER NOT NULL, id_autor INTEGER NOT NULL REFERENCES autores(id))");
    Ejecutar(con, "INSERT INTO autores (nombre) VALUES ('Cervantes'), ('García Márquez'), ('Frank Herbert')");
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

record Libro(int? Id, string Titulo, int Anio, int Stock, int IdAutor);

/// <summary>Único sitio del programa que conoce el SQL y la conexión.</summary>
class LibroDao(SqliteConnection con)
{
    private const string Columnas = "id, titulo, anio, stock, id_autor";

    private static Libro DeFila(SqliteDataReader r) => new(r.GetInt32(0), r.GetString(1), r.GetInt32(2), r.GetInt32(3), r.GetInt32(4));

    public Libro Guardar(Libro libro)
    {
        using var cmd = con.CreateCommand();
        cmd.CommandText = "INSERT INTO libros (titulo, anio, stock, id_autor) VALUES (@titulo, @anio, @stock, @autor) RETURNING id";
        cmd.Parameters.AddWithValue("@titulo", libro.Titulo);
        cmd.Parameters.AddWithValue("@anio", libro.Anio);
        cmd.Parameters.AddWithValue("@stock", libro.Stock);
        cmd.Parameters.AddWithValue("@autor", libro.IdAutor);
        int id = Convert.ToInt32(cmd.ExecuteScalar());
        return libro with { Id = id };
    }

    public Libro? Buscar(int id)
    {
        using var cmd = con.CreateCommand();
        cmd.CommandText = $"SELECT {Columnas} FROM libros WHERE id = @id";
        cmd.Parameters.AddWithValue("@id", id);
        using var lector = cmd.ExecuteReader();
        return lector.Read() ? DeFila(lector) : null;
    }

    public List<Libro> Todos()
    {
        using var cmd = con.CreateCommand();
        cmd.CommandText = $"SELECT {Columnas} FROM libros ORDER BY id";
        return Leer(cmd);
    }

    public List<Libro> PorAutor(string nombre)
    {
        using var cmd = con.CreateCommand();
        cmd.CommandText = "SELECT l.id, l.titulo, l.anio, l.stock, l.id_autor FROM libros l "
                        + "JOIN autores a ON a.id = l.id_autor WHERE a.nombre = @nombre ORDER BY l.anio";
        cmd.Parameters.AddWithValue("@nombre", nombre);
        return Leer(cmd);
    }

    public void CambiarStock(int id, int diferencia)
    {
        using var cmd = con.CreateCommand();
        cmd.CommandText = "UPDATE libros SET stock = stock + @diferencia WHERE id = @id";
        cmd.Parameters.AddWithValue("@diferencia", diferencia);
        cmd.Parameters.AddWithValue("@id", id);
        cmd.ExecuteNonQuery();
    }

    private static List<Libro> Leer(SqliteCommand cmd)
    {
        var lista = new List<Libro>();
        using var lector = cmd.ExecuteReader();
        while (lector.Read())
        {
            lista.Add(DeFila(lector));
        }
        return lista;
    }
}

/// <summary>Servicio: usa el DAO y no contiene SQL.</summary>
class Catalogo(LibroDao dao)
{
    public void PrestarUno(int id)
    {
        Libro libro = dao.Buscar(id) ?? throw new ArgumentException("el libro no existe");
        if (libro.Stock < 1)
        {
            throw new InvalidOperationException("no hay stock");
        }
        dao.CambiarStock(id, -1);
    }
}
```

Salida:

```text
guardado: Libro { Id = 5, Titulo = Dune, Anio = 1965, Stock = 2, IdAutor = 3 }
buscar(5): Dune
buscar(99): no existe
del autor Cervantes: Don Quijote, Novelas ejemplares
stock de Dune tras prestar uno: 1
total de libros: 5
```

Fíjate en el reparto de responsabilidades:

* **`Libro`** es un objeto de datos (un `record`), sin SQL. Al imprimirlo, un `record` enseña todos sus campos; y `libro with { Id = id }` crea una copia con otro valor, que es como el DAO le pone el identificador que ha generado la base de datos.
* **`LibroDao`** es el **único** sitio con SQL y con la conexión. Convierte filas en objetos `Libro` y al revés. Para saber el `id` que ha recibido la fila nueva usa `RETURNING id` al final del `INSERT`.
* **`Catalogo`** es el servicio: contiene las **reglas** («no se puede prestar sin stock») y usa el DAO, pero **no contiene SQL**.
* Si mañana cambia el motor de base de datos, solo se modifica el DAO.

!!! tip "Relación con SOLID"
    El DAO aplica la **responsabilidad única** (cada clase tiene una sola razón para cambiar) y, si el servicio recibe el DAO por el constructor, la **inversión de dependencias** (ver [6.B](../u06/t-solid-1.md) y [6.C](../u06/t-solid-2.md)): el servicio se podría probar con un DAO falso, sin base de datos. Para eso lo habitual es declarar una interfaz (`ILibroDao`) que implemente el DAO.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Concatenar o interpolar texto del usuario en el SQL | Parámetros con nombre, siempre |
| Varias operaciones relacionadas sin transacción | Agruparlas con `BeginTransaction` y deshacer ante cualquier error |
| Olvidar el `Rollback` cuando algo falla | Capturar el error y deshacer siempre |
| SQL esparcido por todo el programa | Una capa DAO |
| Mantener la conexión abierta todo el programa contra un servidor | Abrir tarde y cerrar pronto: el pool se encarga de reutilizar |

## Para practicar

Haz los ejercicios de [U9.3 · Pool, seguridad, transacciones y DAO](buenas-practicas.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
