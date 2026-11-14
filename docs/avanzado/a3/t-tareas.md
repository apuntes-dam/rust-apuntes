# A3.A Tareas a la vez

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5, en especial errores y structs. Aquí las esperas son de decenas de milisegundos para que las salidas sean siempre las mismas.

Un programa que hace una cosa tras otra pierde mucho tiempo **esperando**: una respuesta de la red, un archivo, un temporizador. La **concurrencia** consiste en aprovechar esa espera, o repartir el trabajo, entre varios **hilos**.

## El modelo de Rust: `std::thread`

A diferencia de otros lenguajes, la biblioteca estándar de Rust no trae `async`/`await` (eso lo añaden bibliotecas como *Tokio*, fuera del alcance de esta unidad): el camino directo son **hilos del sistema operativo**:

| Pieza | Qué es |
|---|---|
| **`thread::spawn(closure)`** | Lanza un hilo nuevo que ejecuta la closure; devuelve un **`JoinHandle`** |
| **`handle.join()`** | Espera a que ese hilo termine; devuelve un `Result` con lo que la closure devolvió (o un error, si el hilo entró en *panic*) |
| **`thread::sleep(duracion)`** | Para el hilo actual durante ese tiempo (**bloquea** el hilo, a diferencia de un `await`) |

Para que una closure pueda ejecutarse en otro hilo, Rust exige que **no** capture referencias a datos que podrían desaparecer antes de tiempo: casi siempre hace falta `move`, para que el hilo sea dueño de lo que necesita. El compilador te avisa en tiempo de compilación si algo no es seguro entre hilos: no hay que esperar a ejecutar para descubrirlo.

## Un ejemplo: tres descargas a la vez

Tres «descargas» con una pausa cada una. Primero una tras otra y luego a la vez:

```rust
use std::thread;
use std::time::Instant;
use std::time::Duration;

fn descargar(nombre: &str, ms: u64) {
    thread::sleep(Duration::from_millis(ms));
    println!("termina {}", nombre);
}

fn main() {
    // Una tras otra
    let inicio = Instant::now();
    descargar("uno", 200);
    descargar("dos", 50);
    descargar("tres", 100);
    let secuencial = inicio.elapsed().as_millis() >= 340;

    println!("--- ahora a la vez ---");

    // Las tres a la vez: se lanzan los tres hilos y luego se espera a todos
    let inicio2 = Instant::now();
    let h1 = thread::spawn(|| descargar("uno", 200));
    let h2 = thread::spawn(|| descargar("dos", 50));
    let h3 = thread::spawn(|| descargar("tres", 100));
    h1.join().unwrap();
    h2.join().unwrap();
    h3.join().unwrap();
    let simultaneo = inicio2.elapsed().as_millis() < 340;

    println!("una tras otra tarda 350 ms o más: {}", secuencial);
    println!("a la vez tarda menos de 340 ms: {}", simultaneo);
}
```

Salida:

```text
termina uno
termina dos
termina tres
--- ahora a la vez ---
termina dos
termina tres
termina uno
una tras otra tarda 350 ms o más: true
a la vez tarda menos de 340 ms: true
```

* Llamando a `descargar` **una tras otra**, el tiempo se suma (200 + 50 + 100 = 350 ms).
* Con `thread::spawn`, se **lanzan los tres hilos** y después se espera a todos con `.join()`: el tiempo total es el del hilo **más lento** (200 ms). Se ve en el orden en que terminan: `dos`, `tres`, `uno`, que no es el orden en que se lanzaron.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar `.join()` | El hilo sigue en marcha, pero el programa puede terminar antes de que acabe: casi siempre quieres esperarlo |
| Capturar una referencia que no sobrevive al hilo | Usa `move` para que el hilo sea dueño de los datos |
| Pensar que `thread::sleep` libera el hilo para otra cosa | **Bloquea** el hilo entero; para miles de esperas a la vez con pocos hilos, se necesita una biblioteca async (fuera de esta unidad) |
| Lanzar muchísimos hilos para tareas muy pequeñas | Cada hilo tiene su coste; para repartir un cálculo entre núcleos, hay *crates* como *rayon* pensadas para eso |

## Para practicar

Los ejercicios [A3.1 y A3.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
