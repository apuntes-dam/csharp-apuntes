# A5.C Estructura y cuándo usar patrones

## Decorador: añadir sin modificar

Un café puede llevar leche, canela, sirope... Si cada combinación fuera una clase (`CafeConLecheYCanela`...), habría una explosión de clases. El patrón **decorador** envuelve un objeto en otro que **tiene su misma interfaz**, añade algo y delega el resto. Se pueden apilar tantos como se quiera, en el orden que se quiera:

```csharp
ICafe cafe = new CafeSimple();
Console.WriteLine($"{cafe.Descripcion}: {cafe.Precio} céntimos");

cafe = new ConLeche(cafe);
Console.WriteLine($"{cafe.Descripcion}: {cafe.Precio} céntimos");

cafe = new ConCanela(new ConLeche(new CafeSimple()));
Console.WriteLine($"{cafe.Descripcion}: {cafe.Precio} céntimos");

// Un decorador es, a la vez, un ICafe y tiene un ICafe dentro
interface ICafe
{
    string Descripcion { get; }
    int Precio { get; }
}

class CafeSimple : ICafe
{
    public string Descripcion => "café";
    public int Precio => 120;
}

class ConLeche(ICafe interior) : ICafe
{
    public string Descripcion => interior.Descripcion + " con leche";
    public int Precio => interior.Precio + 30;
}

class ConCanela(ICafe interior) : ICafe
{
    public string Descripcion => interior.Descripcion + " y canela";
    public int Precio => interior.Precio + 10;
}
```

Salida:

```text
café: 120 céntimos
café con leche: 150 céntimos
café con leche y canela: 160 céntimos
```

Cada decorador **es un `ICafe` y contiene un `ICafe`**. Por eso se pueden encadenar, y el orden cambia el resultado (el ejercicio A5.6 lo muestra con texto). No hay que tocar `CafeSimple` ni inventar clases para cada mezcla. Si lo has visto en el mundo real: los `Stream` de .NET se decoran así (`new StreamReader(new BufferedStream(new FileStream(...)))`).

## Otros dos que conviene conocer

**Adaptador.** Conecta dos interfaces incompatibles, cuando no puedes cambiar ninguna de las dos (por ejemplo, una biblioteca de terceros):

```csharp
// Código que no puedes cambiar (una biblioteca): mide en millas
var antiguo = new MedidorMillas();

// Tu código espera kilómetros
IMedidor medidor = new AdaptadorKilometros(antiguo);
Console.WriteLine($"{medidor.Kilometros(10)} km");

interface IMedidor
{
    double Kilometros(double recorrido);
}

class MedidorMillas
{
    public double Millas(double recorrido) => recorrido;
}

// El adaptador traduce una interfaz a la otra
class AdaptadorKilometros(MedidorMillas millas) : IMedidor
{
    public double Kilometros(double recorrido) => Math.Round(millas.Millas(recorrido) * 1.609344, 2);
}
```

Salida:

```text
16,09 km
```

**Fachada.** Ofrece una interfaz **sencilla** delante de un subsistema complicado: una clase `Pedidos` con un método `Comprar(...)` que por dentro coordina el almacén, el cobro y el envío. Quien la usa no necesita conocer esas piezas.

## ¿Qué patrón uso?

| Tengo este problema... | Patrón |
|---|---|
| Necesito **una sola** instancia de algo | Singleton (mejor: inyección de dependencias) |
| Quien crea el objeto no debe saber su clase concreta | Fábrica |
| El objeto tiene muchas piezas opcionales | Builder (o `record` con `init` y `required`) |
| Hay **varias formas** de hacer lo mismo y quiero elegir | Estrategia |
| Varios objetos deben **enterarse** de un cambio | Observador (`event`) |
| Quiero **añadir** comportamiento sin tocar la clase ni heredar | Decorador |
| Debo **conectar** dos interfaces que no encajan | Adaptador |
| Quiero **simplificar** el uso de algo complejo | Fachada |

## Cuándo NO usar un patrón

Los patrones resuelven problemas **que existen**. Aplicarlos «por si acaso» hace el código más largo y más difícil de leer sin ganar nada:

* **Si no hay variantes, no hay estrategia.** Un `if` con dos casos que no van a crecer no necesita tres clases.
* **Si hay una sola clase, no hay fábrica.**
* **Primero el problema, luego el patrón.** Escribe lo más simple que funcione; cuando veas el dolor (cambios repetidos en muchos sitios), busca el patrón que lo alivia.
* En C# muchos patrones **ya están en el lenguaje**: el observador son los eventos, la estrategia sencilla es un `Func`, el iterador es `yield return`, el singleton lo gestiona la inyección de dependencias.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Aplicar patrones por costumbre, aunque no haya problema | Empezar simple y refactorizar cuando duela |
| Llamar «patrón» a cualquier clase con un nombre elegante | Un patrón es una estructura con un problema concreto detrás |
| Decoradores que no respetan la interfaz común | El decorador debe ser intercambiable con lo que decora |
| Mezclar demasiados patrones en una pieza pequeña | Un patrón por problema |
| Copiar la versión de Java cuando el lenguaje ya trae algo mejor | Mirar si C# lo resuelve con un `event`, un `Func` o `required` |

## Para practicar

El ejercicio [A5.6](ejercicios.md) usa el decorador. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
