# 1.5 Entorno: instalar, compilar y ejecutar

## Qué hace falta

| Pieza | Para qué |
|---|---|
| **SDK de .NET** | Compilador, bibliotecas y la herramienta `dotnet` |
| **Un editor** | Visual Studio Code con la extensión *C# Dev Kit*, Visual Studio (Windows) o JetBrains Rider |
| **NuGet** | Gestor de paquetes (viene con el SDK) |

El SDK se descarga desde [dotnet.microsoft.com](https://dotnet.microsoft.com/download) y se instala en Windows, macOS y Linux. En Windows también se puede instalar con `winget install Microsoft.DotNet.SDK.10`.

## Comprobar la instalación

```bash
dotnet --version
dotnet --list-sdks
```

## Crear y ejecutar un proyecto

```bash
dotnet new console -n Hola
cd Hola
dotnet run
```

`dotnet new console` crea una carpeta con dos cosas importantes:

| Archivo | Qué es |
|---|---|
| `Hola.csproj` | La descripción del proyecto: versión de .NET, opciones y paquetes |
| `Program.cs` | Tu código; el programa empieza aquí |

Y después de compilar aparecen las carpetas `bin/` (el resultado) y `obj/` (archivos temporales), que no se suben a Git.

## Ejecutar un solo archivo

Desde .NET 10 se puede ejecutar un archivo `.cs` suelto, sin proyecto, igual que en Java con `java Archivo.java`:

```bash
dotnet run hola.cs
```

Es lo más cómodo para probar un ejercicio. La primera ejecución tarda unos segundos porque compila el programa.

## Compilar y publicar

```bash
dotnet build                  # compila el proyecto
dotnet run                    # compila y ejecuta
dotnet publish -c Release     # genera la versión final para distribuir
```

## Paquetes de NuGet

```bash
dotnet add package Newtonsoft.Json
```

Añade la librería al `.csproj` y la descarga de NuGet. Ya la puedes usar con `using`.

!!! note "Errores frecuentes"
    * `dotnet` no encontrado: falta el SDK o no está en el `PATH`; reinicia el terminal tras instalarlo.
    * `Console.ReadLine()` devuelve `null` si no hay entrada; en un editor en línea sin teclado, escribe los datos en el código.
    * Varias versiones del SDK: un archivo `global.json` fija cuál usa un proyecto.
