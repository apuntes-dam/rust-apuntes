# A3.C Compartir datos con seguridad

## La condición de carrera, evitada al compilar

Cuando varios hilos **modifican el mismo dato** a la vez, pueden pisarse: `contador += 1` son en realidad tres pasos (leer, sumar, escribir), y si dos hilos leen el mismo valor antes de que el otro escriba, una suma se pierde. En muchos lenguajes esto compila igual y falla (o no) al ejecutar, de forma difícil de reproducir. **En Rust no compila**: el verificador de préstamos (*borrow checker*) no deja que una closure capture `&mut` a algo que otro hilo también podría tocar. Hay que envolver el dato en algo que **demuestre** que el acceso es seguro:

| Herramienta | Qué da |
|---|---|
| **`Mutex<T>`** | Acceso **exclusivo**: `.lock()` da una referencia mutable, y solo un hilo a la vez puede tenerla |
| **`Arc<T>`** (*Atomic Reference Counted*) | Propiedad **compartida** entre hilos: cada `Arc::clone` es barato (solo suma un contador) y el valor se libera cuando el último `Arc` desaparece |
| **`Arc<Mutex<T>>`** | La combinación habitual: varios hilos son dueños a la vez de un dato que solo uno puede modificar en cada instante |

```rust
use std::sync::{Arc, Mutex};
use std::thread;

const TAREAS: usize = 4;
const VUELTAS: usize = 100_000;

fn main() {
    // El compilador NO DEJA compartir un contador mutable entre hilos sin protegerlo:
    // Arc (propiedad compartida, con recuento de referencias) + Mutex (acceso exclusivo)
    let contador = Arc::new(Mutex::new(0));
    let mut manejadores = Vec::new();
    for _ in 0..TAREAS {
        let contador = Arc::clone(&contador);
        manejadores.push(thread::spawn(move || {
            for _ in 0..VUELTAS {
                let mut valor = contador.lock().unwrap();
                *valor += 1;
            }
        }));
    }
    for h in manejadores {
        h.join().unwrap();
    }
    println!("con Mutex: {}", *contador.lock().unwrap());
}
```

Salida:

```text
con Mutex: 400000
```

`contador.lock()` devuelve un `Result` que se resuelve con `.unwrap()`; el `MutexGuard` que da **libera el candado solo**, al salir de su ámbito (`{ }` o el final del bucle), así que no hay que acordarse de soltarlo a mano.

## Mejor que compartir: pasarse mensajes

La mejor manera de no tener condiciones de carrera es **no compartir** datos modificables. En su lugar, un hilo **envía** valores a otro por un **canal** (`std::sync::mpsc`): un **productor** escribe con `.send()`, un **consumidor** lee con un `for` sobre el receptor, y nadie toca el dato del otro.

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    let productor = thread::spawn(move || {
        for n in 1..=5 {
            tx.send(n).unwrap();
            println!("envía {}", n);
        }
        // al terminar el hilo, "tx" se suelta y el canal se cierra solo
    });

    let mut suma = 0;
    for n in rx {
        suma += n;
    }
    productor.join().unwrap();
    println!("suma: {}", suma);
}
```

Salida:

```text
envía 1
envía 2
envía 3
envía 4
envía 5
suma: 15
```

El `for n in rx` recibe valores hasta que el canal se cierra, y se cierra **solo** cuando se suelta el último `Sender` (aquí, cuando el hilo productor termina y `tx` sale de ámbito). No hace falta una señal de «fin» explícita, a diferencia de otros lenguajes.

## Reglas para dormir tranquilo

* **Prefiere no compartir.** Datos inmutables ([A1.C](../a1/t-composicion.md)) y canales eliminan casi todos los problemas.
* El compilador **obliga** a proteger lo que comparten varios hilos; si tu código compila, ya tienes una garantía fuerte de que no hay una condición de carrera clásica sobre esos datos.
* Para un solo número que cambia mucho, también existen los tipos **atómicos** (`std::sync::atomic::AtomicI32`, `.fetch_add(...)`), más baratos que un `Mutex`.
* Si necesitas varios `Mutex` a la vez, tómalos **siempre en el mismo orden** para evitar un *interbloqueo* (cada hilo espera el candado que tiene el otro).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Intentar compartir `&mut` entre hilos sin `Mutex` | El compilador lo rechaza: envuelve el dato en `Arc<Mutex<T>>` |
| Clonar el dato en vez de `Arc` (cada hilo tendría su copia) | `Arc::clone` comparte el **mismo** dato; un `.clone()` normal copiaría el valor |
| Mantener el candado (`MutexGuard`) más tiempo del necesario | Suéltalo pronto: sácalo de un bloque `{ }` corto si el resto del código no lo necesita |
| Usar `Mutex` para un solo contador cuando bastaría un atómico | `AtomicI32`/`AtomicUsize` son más baratos para un solo número |

## Para practicar

Los ejercicios [A3.5 y A3.6](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
