# 9.A Conectar y crear tablas

!!! tip "¿Quieres aprender SQL a fondo?"
    Esta unidad explica **lo mínimo de SQL** para entender los ejemplos. Para estudiarlo con calma (consultas, `JOIN`, agrupaciones, diseño de tablas, transacciones...) y con ejercicios, tienes la web de [SQL y bases de datos](https://apuntes-dam.github.io/sql-apuntes/).

## Qué es una base de datos relacional

Una **base de datos relacional** guarda la información en **tablas**: cada tabla tiene **columnas** (los datos que se guardan) y **filas** (cada elemento guardado). Las tablas se **relacionan** entre sí mediante claves, y todo se maneja con un lenguaje común, **SQL**.

| Concepto | Qué es | Ejemplo |
|---|---|---|
| **Tabla** | Un conjunto de datos del mismo tipo | `libros` |
| **Fila** (registro) | Un elemento guardado | el libro «Don Quijote» |
| **Columna** (campo) | Un dato de cada fila, con un tipo | `titulo`, `anio`, `stock` |
| **Clave primaria** (*primary key*) | Columna que **identifica de forma única** cada fila | `id` |
| **Clave foránea** (*foreign key*) | Columna que **apunta a la clave primaria de otra tabla** | `libros.id_autor` → `autores.id` |

En los ejemplos de esta unidad hay tres tablas relacionadas: un **autor** tiene muchos **libros** y un **libro** puede tener muchos **préstamos**.

```text
autores (id, nombre)  1 ──── N  libros (id, titulo, anio, stock, id_autor)  1 ──── N  prestamos (id, id_libro, socio)
```

## Motores de bases de datos

| Motor | Cómo funciona | Cuándo se usa |
|---|---|---|
| **SQLite** | Un **archivo** (o la memoria); no hay servidor ni instalación | Aprender, apps móviles y de escritorio, pruebas |
| **SQL Server**, **MySQL**, **PostgreSQL** | **Servidor** al que se conecta por red con usuario y contraseña | Aplicaciones reales con muchos usuarios |

!!! note "Qué motor usan estos ejemplos"
    Todos los ejemplos usan **SQLite**, porque no necesita instalar nada. Los ejercicios del módulo mencionan **H2**, que es una base de datos de Java y no existe en .NET: usa SQLite en su lugar. El código es casi igual con otro motor: cambia el paquete y la clase de la conexión (por ejemplo, `Npgsql` para PostgreSQL), pero las ideas y la mayor parte del SQL son las mismas. Las sentencias sí varían un poco entre motores: la columna que se numera sola es `AUTOINCREMENT` en SQLite y `AUTO_INCREMENT` en MySQL.

## ADO.NET: la forma de hablar con una base de datos en C&#35;

.NET accede a las bases de datos con **ADO.NET**, una familia de clases comunes a todos los motores (como JDBC en Java). Cada motor aporta su propio **paquete** (el *proveedor*), que se descarga de NuGet:

| Motor | Paquete | Clase de la conexión |
|---|---|---|
| SQLite | `Microsoft.Data.Sqlite` | `SqliteConnection` |
| SQL Server | `Microsoft.Data.SqlClient` | `SqlConnection` |
| PostgreSQL | `Npgsql` | `NpgsqlConnection` |
| MySQL | `MySqlConnector` | `MySqlConnection` |

Las piezas son siempre las mismas, con distinto prefijo según el motor:

| Pieza | Para qué sirve |
|---|---|
| **Conexión** (`SqliteConnection`) | La conexión con la base de datos; se abre con `Open()` y se cierra con `using` |
| **Comando** (`SqliteCommand`) | Una sentencia SQL en `CommandText`, con sus parámetros |
| **Lector** (`SqliteDataReader`) | Recorre las filas de una consulta con `Read()` |

Para usar un paquete en un archivo suelto, se declara en la primera línea con `#:package`. La primera vez que se ejecuta, `dotnet run` lo descarga (hace falta internet); en un proyecto normal se añade con `dotnet add package Microsoft.Data.Sqlite`. Los ejemplos de esta unidad empiezan así:

```csharp
#:package Microsoft.Data.Sqlite@10.0.0
#:package SQLitePCLRaw.lib.e_sqlite3@2.1.14
```

La segunda línea fija la versión del motor nativo de SQLite que se incluye: sin ella, la versión por defecto trae un aviso de seguridad de NuGet. En proyectos grandes los datos se suelen manejar con un **ORM** como *Entity Framework Core*, que genera el SQL por ti; esta unidad trabaja con SQL directo para entender qué ocurre debajo.

## Conectar, crear las tablas e insertar datos

```csharp
#:package Microsoft.Data.Sqlite@10.0.0
#:package SQLitePCLRaw.lib.e_sqlite3@2.1.14

using Microsoft.Data.Sqlite;

try
{
    using (var con = new SqliteConnection("Data Source=biblioteca.db"))
    {
        con.Open();
        Console.WriteLine("conectado a la base de datos");
        Ejecutar(con, "CREATE TABLE autores (id INTEGER PRIMARY KEY AUTOINCREMENT, nombre TEXT NOT NULL)");
        Ejecutar(con, "CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, "
                    + "anio INTEGER, stock INTEGER NOT NULL, id_autor INTEGER NOT NULL REFERENCES autores(id))");
        Ejecutar(con, "CREATE TABLE prestamos (id INTEGER PRIMARY KEY AUTOINCREMENT, "
                    + "id_libro INTEGER NOT NULL REFERENCES libros(id), socio TEXT NOT NULL)");
        Console.WriteLine("tablas creadas");

        string[] autores = ["Cervantes", "García Márquez"];
        foreach (string nombre in autores)
        {
            using var cmd = con.CreateCommand();
            cmd.CommandText = "INSERT INTO autores (nombre) VALUES (@nombre)";
            cmd.Parameters.AddWithValue("@nombre", nombre);
            cmd.ExecuteNonQuery();
        }
        Console.WriteLine($"autores insertados: {autores.Length}");

        (string Titulo, int Anio, int Stock, int IdAutor)[] libros =
        [
            ("Don Quijote", 1605, 3, 1),
            ("Novelas ejemplares", 1613, 2, 1),
            ("Cien años de soledad", 1967, 5, 2),
            ("El coronel no tiene quien le escriba", 1961, 1, 2),
        ];
        foreach (var libro in libros)
        {
            using var cmd = con.CreateCommand();
            cmd.CommandText = "INSERT INTO libros (titulo, anio, stock, id_autor) VALUES (@titulo, @anio, @stock, @autor)";
            cmd.Parameters.AddWithValue("@titulo", libro.Titulo);
            cmd.Parameters.AddWithValue("@anio", libro.Anio);
            cmd.Parameters.AddWithValue("@stock", libro.Stock);
            cmd.Parameters.AddWithValue("@autor", libro.IdAutor);
            cmd.ExecuteNonQuery();
        }
        Console.WriteLine($"libros insertados: {libros.Length}");

        using (var cmd = con.CreateCommand())
        {
            cmd.CommandText = "SELECT COUNT(*) FROM libros";
            Console.WriteLine($"libros en la base de datos: {cmd.ExecuteScalar()}");
        }

        try
        {
            Ejecutar(con, "INSERT INTO editoriales (nombre) VALUES ('X')");
        }
        catch (SqliteException)
        {
            Console.WriteLine("Error controlado: la sentencia no es válida");
        }
    }
}
catch (SqliteException e)
{
    Console.WriteLine($"Error de base de datos: {e.Message}");
}
Console.WriteLine("conexión cerrada");

SqliteConnection.ClearAllPools();      // suelta el archivo: sin esto, Windows no deja borrarlo
File.Delete("biblioteca.db");
Console.WriteLine($"archivo biblioteca.db borrado: {!File.Exists("biblioteca.db")}");

static void Ejecutar(SqliteConnection con, string sql)
{
    using var cmd = con.CreateCommand();
    cmd.CommandText = sql;
    cmd.ExecuteNonQuery();
}
```

Salida:

```text
conectado a la base de datos
tablas creadas
autores insertados: 2
libros insertados: 4
libros en la base de datos: 4
Error controlado: la sentencia no es válida
conexión cerrada
archivo biblioteca.db borrado: True
```

Qué hace cada paso:

1. **Conecta** con la base de datos (el archivo `biblioteca.db`, que se crea si no existe). `Data Source=biblioteca.db` es la **cadena de conexión**.
2. **Crea las tres tablas** con `CREATE TABLE`, indicando para cada columna su tipo y sus restricciones.
3. **Inserta datos** con `INSERT` y **parámetros** (`@nombre`): los valores no se pegan dentro del texto SQL, se pasan aparte con `Parameters.AddWithValue`. Esto es lo que se explica en [9.C](t-buenas-practicas.md) y es **obligatorio** cuando el dato viene del usuario. En SQLite para .NET los parámetros **tienen nombre** (`@nombre`); el `?` sin nombre de otros lenguajes no vale.
4. **Cuenta** las filas con una consulta que devuelve un único valor (`ExecuteScalar`).
5. **Provoca un error a propósito** (insertar en una tabla que no existe) y lo **captura**: el programa no se detiene y muestra un mensaje claro. Los errores de SQL son una **`SqliteException`**.
6. **Cierra la conexión** con el `using` **siempre**, haya habido error o no.

!!! warning "Cerrar la conexión no suelta el archivo"
    `Microsoft.Data.Sqlite` mantiene un *pool* de conexiones: al cerrar una conexión, la real queda guardada para reutilizarla, y el archivo sigue **abierto**. Por eso, para borrar `biblioteca.db` después de cerrar, el ejemplo llama a `SqliteConnection.ClearAllPools()` (sin esa línea, `File.Delete` falla en Windows con una `IOException`: «el proceso no tiene acceso al archivo porque está siendo utilizado»). Otra forma es desactivar el pool en la cadena de conexión: `Data Source=biblioteca.db;Pooling=False`.

## Crear tablas: tipos y restricciones

```sql
CREATE TABLE libros (
  id       INTEGER PRIMARY KEY AUTOINCREMENT,   -- identifica cada fila; se numera solo
  titulo   TEXT    NOT NULL,                    -- obligatorio
  anio     INTEGER,                             -- puede quedar vacío (NULL)
  stock    INTEGER NOT NULL,
  id_autor INTEGER NOT NULL REFERENCES autores(id)   -- clave foránea
);
```

| Restricción | Significa |
|---|---|
| `PRIMARY KEY` | Identifica la fila; no se repite ni puede ser nula |
| `NOT NULL` | La columna no puede quedar vacía |
| `UNIQUE` | No puede haber dos filas con el mismo valor (por ejemplo, un correo) |
| `REFERENCES tabla(columna)` | Clave foránea: el valor **debe existir** en la otra tabla |
| `DEFAULT valor` | Valor que se usa si no se indica ninguno |

| Tipo en SQLite | Para qué | Tipo de C# al leerlo | Equivalente habitual en otros motores |
|---|---|---|---|
| `INTEGER` | Números enteros | `int` o `long` (`GetInt32`, `GetInt64`) | `INT`, `BIGINT` |
| `TEXT` | Texto | `string` (`GetString`) | `VARCHAR(n)`, `TEXT` |
| `REAL` | Números decimales | `double` (`GetDouble`) | `DOUBLE`, `FLOAT` |
| `BLOB` | Datos binarios | `byte[]` | `BLOB`, `BYTEA` |

!!! tip "Dinero y decimales"
    En C# el tipo para dinero es `decimal`, que guarda decimales **exactos**. SQLite no tiene un tipo `DECIMAL` real: lo guardaría como `REAL` (coma flotante) y pueden aparecer errores de redondeo, como en `0.1 + 0.2`. Otros motores sí ofrecen `DECIMAL(10,2)`. Con SQLite, una alternativa muy usada es guardar el importe **en céntimos**, como entero.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| No cerrar la conexión | `using` |
| Pegar el valor dentro del SQL | Usar parámetros con nombre (`@valor`) |
| Usar `?` como parámetro | En Microsoft.Data.Sqlite los parámetros tienen nombre: `@titulo` |
| Intentar borrar el archivo de la base de datos justo después de cerrar | `SqliteConnection.ClearAllPools()` o `Pooling=False` |
| Crear una tabla que ya existe | Crear las tablas una vez, o usar `CREATE TABLE IF NOT EXISTS` |
| Insertar con claves foráneas que no existen | Insertar primero la tabla «padre» (los autores) y luego la «hija» (los libros) |
| No tener internet la primera vez | `#:package` descarga el paquete de NuGet al ejecutar por primera vez |

## Para practicar

Haz los ejercicios de [U9.1 · Conexión y creación de tablas](conexion.md). [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) compara cómo se escribe en cada lenguaje.
