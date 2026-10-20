# 7.D Interfaces gráficas

Hasta ahora los programas **leían** de la consola y **escribían** en ella, en un orden fijo. Una **interfaz gráfica** (GUI) cambia la idea: el programa muestra una **ventana** y **espera** a que el usuario haga algo (escribir, pulsar, elegir). Se dice que está **dirigido por eventos**.

## Ideas comunes a todas las librerías

| Idea | Qué es |
|---|---|
| **Componente** (*widget* o *control*) | Cada elemento de la ventana: texto, campo de entrada, botón, lista, imagen |
| **Contenedor y diseño** (*layout*) | Cómo se colocan los componentes: en columna, en fila, en rejilla |
| **Evento** | Algo que ocurre: una pulsación, una tecla, un clic |
| **Controlador de eventos** (*handler* o *callback*) | El código que se ejecuta cuando ocurre el evento |
| **Estado** | Los datos que cambian y que determinan lo que se ve (el texto escrito, el resultado) |
| **Bucle de eventos** | El ciclo interno que espera eventos y los reparte; se pone en marcha al abrir la ventana |

El programa no controla el orden: **reacciona**. Por eso cada botón lleva asociado su controlador.

## C# tiene varias librerías

En C# hay varias, y la elección depende de dónde quieras que se ejecute el programa:

| Librería | Para qué | Notas |
|---|---|---|
| **Windows Forms** | Aplicaciones de escritorio **solo en Windows** | La más sencilla y la clásica. Es la que se usa aquí |
| **WPF** | Escritorio **solo en Windows**, con interfaces descritas en **XAML** | Más potente en diseño y enlace de datos |
| **.NET MAUI** | Móvil (Android, iOS) y escritorio | Una sola base de código, también con XAML |
| **Avalonia** | Escritorio en Windows, Linux y macOS | Parecida a WPF, de la comunidad |

Windows Forms funciona como Swing en Java o Tkinter en Python: es **imperativa**. Tú **creas objetos** (`Form` para la ventana, `Button`, `Label`, `TextBox`...), los **colocas** dentro de un contenedor con un *layout* y les **añades un controlador** al evento (`Click`) que se ejecuta cuando ocurre algo. Cambias lo que se ve modificando sus propiedades (`Resultado.Text = ...`).

## Un ejemplo: la propina

La ventana tiene un campo para escribir un importe, un botón «Calcular» y un texto con el resultado (el 10 % del importe). Si lo escrito no es un número entero o es negativo, muestra un mensaje de error. Este programa necesita **Windows** y las tres primeras líneas (`#:property`), que le dicen a `dotnet run` que use la plataforma de escritorio de Windows:

```csharp
#:property TargetFramework=net10.0-windows
#:property UseWindowsForms=true
#:property PublishAot=false

using System.Windows.Forms;

Application.EnableVisualStyles();
Application.Run(new VentanaPropina());

class VentanaPropina : Form
{
    public static string CalcularPropina(string texto)
    {
        if (!int.TryParse(texto.Trim(), out int importe))
        {
            return "Escribe un número entero";
        }
        if (importe < 0)
        {
            return "El importe no puede ser negativo";
        }
        return $"Propina: {importe / 10} €";
    }

    public readonly TextBox Entrada = new() { Width = 120 };
    public readonly Button Boton = new() { Text = "Calcular", AutoSize = true };
    public readonly Label Resultado = new() { Text = " ", AutoSize = true };

    public VentanaPropina()
    {
        Text = "Propina";
        var panel = new FlowLayoutPanel { Dock = DockStyle.Fill, FlowDirection = FlowDirection.TopDown, Padding = new Padding(10) };
        panel.Controls.Add(new Label { Text = "Importe (€):", AutoSize = true });
        panel.Controls.Add(Entrada);
        panel.Controls.Add(Boton);
        panel.Controls.Add(Resultado);
        Controls.Add(panel);

        Boton.Click += (origen, evento) => Resultado.Text = CalcularPropina(Entrada.Text);
    }
}
```

Se ejecuta con `dotnet run propina.cs`. Hay una decisión de diseño importante: **`CalcularPropina` es una función aparte**, que recibe un texto y devuelve otro, sin saber nada de ventanas. La ventana solo se ocupa de **recoger** el texto, **llamar** a la función y **mostrar** el resultado. Esa separación entre **lógica** e **interfaz** permite probar la lógica sin abrir nada y cambiar de librería sin reescribirla.

## Probar la ventana desde el código

Como los componentes son objetos, una prueba puede **escribir en el campo y simular una pulsación** con `PerformClick()`, sin que nadie toque el ratón. Un detalle: `PerformClick()` solo hace algo si la ventana está **mostrada**, así que la prueba llama a `Show()` (la ventana aparece un instante). Es el mismo código de la ventana, con otro arranque:

```csharp
#:property TargetFramework=net10.0-windows
#:property UseWindowsForms=true
#:property PublishAot=false

using System.Windows.Forms;

var ventana = new VentanaPropina();
ventana.Show();      // PerformClick solo actúa con la ventana mostrada
foreach (string texto in new[] { "50", "abc", "-5" })
{
    ventana.Entrada.Text = texto;
    ventana.Boton.PerformClick();
    Console.WriteLine($"{texto} -> {ventana.Resultado.Text}");
}
ventana.Dispose();

// ... la clase VentanaPropina de arriba, sin cambios ...
```

Resultado de los tres casos:

```text
50 -> Propina: 5 €
abc -> Escribe un número entero
-5 -> El importe no puede ser negativo
```

He ejecutado esta prueba con Windows Forms en .NET 10 y devuelve exactamente esos tres resultados. No he comprobado el aspecto de la ventana con el ratón: pruébala tú con `dotnet run`.

## Tres cuidados importantes

* **Nunca bloquees la ventana.** Mientras el controlador de un evento está trabajando, la ventana **no responde**. Las tareas largas (descargas, cálculos pesados, lecturas de archivos grandes) deben hacerse **fuera del hilo de la interfaz**; en C#, con `async`/`await` y `Task.Run`.
* **Valida lo que escribe el usuario.** Todo lo que llega de un campo es **texto**: conviértelo con `int.TryParse` y prevé que falle, como hace `CalcularPropina`.
* **Cambia la interfaz desde el hilo correcto.** Los controles solo se tocan desde el hilo de la interfaz. Si otro hilo necesita cambiar un texto, tiene que pedírselo con `control.Invoke(...)`; hacerlo directamente lanza una excepción.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| El programa no arranca en Linux o macOS | Windows Forms es solo para Windows: usa Avalonia o MAUI |
| Olvidar `#:property UseWindowsForms=true` | Sin ella, `System.Windows.Forms` no existe |
| Escribir la lógica dentro del controlador del botón | Ponerla en una función aparte |
| Congelar la ventana con una tarea larga | Hacerla fuera del hilo de la interfaz |
| Fiarse de que el usuario escribirá un número | Validar y mostrar un mensaje claro |
| Componentes que se salen o se superponen | Usar un contenedor con diseño (`FlowLayoutPanel`, `TableLayoutPanel`) en vez de posiciones fijas |

## Para practicar

Haz los ejercicios de [U7.4 · Interfaces gráficas](gui.md): una ventana con botón, un contador con estado, un formulario con validación y una lista de tareas. Para ver cómo se hace en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
