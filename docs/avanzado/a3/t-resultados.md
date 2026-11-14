# A3.B Resultados, errores y esperas

## Resultados y errores de un hilo

`JoinHandle::join()` devuelve un `std::thread::Result<T>`: `Ok(valor)` con lo que la closure devolvió, o `Err(_)` si el hilo **entró en pánico**. Un *panic* en un hilo secundario no tira abajo todo el programa: se puede capturar con un `match` normal, igual que cualquier otro `Result`.

```rust
use std::thread;

fn main() {
    // 1. Un resultado directo: lo que devuelve la closure del hilo
    let h = thread::spawn(|| 40 + 2);
    let valor = h.join().unwrap();
    println!("resultado: {}", valor);

    // 2. Si el hilo entra en panic, join() devuelve Err en vez de tirar abajo el programa
    let h2 = thread::spawn(|| {
        panic!("algo salió mal");
    });
    match h2.join() {
        Ok(_) => println!("no debería llegar aquí"),
        Err(_) => println!("el hilo falló, pero el programa sigue"),
    }
}
```

Salida:

```text
resultado: 42
el hilo falló, pero el programa sigue
```

## Límites de tiempo y reintentos

Esperar sin límite es peligroso: un hilo que no responde deja esperando para siempre. La biblioteca estándar no tiene un «esperar como máximo X», pero un **canal** (`mpsc`, *multiple producer, single consumer*) lo da gratis: `recv_timeout` deja de esperar pasado un tiempo, aunque el hilo siga trabajando por detrás.

| Quiero... | Herramienta |
|---|---|
| Dejar de **esperar** pasado un tiempo | `receptor.recv_timeout(Duration)` → `Err` si se agota |
| **Repetir** una operación que falla a veces | Un bucle con `match` (o la *crate* *backoff* en proyectos reales) |

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn consulta(ms: u64) -> i32 {
    thread::sleep(Duration::from_millis(ms));
    42
}

// Un canal como límite de tiempo: si no llega nada antes de limite_ms, recv_timeout falla
fn con_limite(ms: u64, limite_ms: u64) -> String {
    let (tx, rx) = mpsc::channel();
    thread::spawn(move || {
        let _ = tx.send(consulta(ms));
    });
    match rx.recv_timeout(Duration::from_millis(limite_ms)) {
        Ok(valor) => format!("resultado: {}", valor),
        Err(_) => "tiempo agotado".to_string(),
    }
}

fn main() {
    println!("{}", con_limite(300, 100));
    println!("{}", con_limite(50, 200));

    // Reintentos: repetir una operación que falla a veces
    let mut llamadas = 0;
    let mut inestable = || {
        llamadas += 1;
        if llamadas < 3 { Err("fallo temporal") } else { Ok("listo") }
    };

    for intento in 1..=3 {
        match inestable() {
            Ok(resultado) => {
                println!("intento {} correcto", intento);
                println!("resultado: {}", resultado);
                break;
            }
            Err(_) => println!("intento {} falló", intento),
        }
    }
}
```

Salida:

```text
tiempo agotado
resultado: 42
intento 1 falló
intento 2 falló
intento 3 correcto
resultado: listo
```

`recv_timeout` **solo deja de esperar**: el hilo que hace la consulta sigue ejecutándose por detrás, aunque ya nadie recoja su resultado. Para pararlo de verdad haría falta que el propio hilo comprobase periódicamente una señal (por ejemplo, un `Arc<AtomicBool>`) y decidiese terminar él mismo: Rust no tiene una forma de «matar» un hilo desde fuera, a propósito, porque podría dejar datos a medio modificar.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Ignorar el `Err` de `.join()` y usar `.unwrap()` sin pensar | Decide si un *panic* en ese hilo debe parar tu programa o no |
| Esperar sin límite una respuesta que puede no llegar | `recv_timeout` sobre un canal |
| Creer que `recv_timeout` detiene el trabajo del otro hilo | Solo deja de esperarlo; para pararlo, el hilo debe cooperar comprobando una señal |
| Reintentar sin un máximo de intentos | Fija siempre un límite |

## Para practicar

Los ejercicios [A3.3 y A3.4](ejercicios.md) usan límites de tiempo y reintentos. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
