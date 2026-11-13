# 7.D Interfaces gráficas

Hasta ahora los programas **leían** de la consola y **escribían** en ella, en un orden fijo. Una **interfaz gráfica** (GUI) cambia la idea: el programa muestra una **ventana** y **espera** a que el usuario haga algo (escribir, pulsar, elegir). Se dice que está **dirigido por eventos**.

## Ideas comunes a todas las librerías

| Idea | Qué es |
|---|---|
| **Componente** (*widget*) | Cada elemento de la ventana: texto, campo de entrada, botón |
| **Contenedor y diseño** (*layout*) | Cómo se colocan los componentes |
| **Evento** | Algo que ocurre: una pulsación, una tecla, un clic |
| **Bucle de eventos** | El ciclo que espera eventos y los reparte; se pone en marcha al abrir la ventana |
| **Estado** | Los datos que determinan lo que se ve, y que el programa va actualizando |

El programa no controla el orden: **reacciona**.

## Rust también tiene varias librerías

| Librería | Qué es | Notas |
|---|---|---|
| **egui** (con `eframe`) | Inmediata (se redibuja todo cada fotograma), multiplataforma | La más sencilla para empezar; la usada aquí |
| **iced** | Declarativa, inspirada en Elm | Más estructura, más código |
| **gtk-rs** | Enlaza con GTK | Se integra bien en Linux |
| **Tauri** | Interfaz web (HTML/CSS/JS) con lógica en Rust | Para apps de escritorio con pinta de web |

`egui` es **inmediata**: en vez de crear los componentes una vez y guardarlos, **describes la pantalla entera en cada fotograma**, como en un videojuego. No hay controladores de eventos a los que apuntarte: un botón `clicked()` ya te dice en esa misma línea si lo acabas de pulsar.

## Separar la lógica de la ventana

Como en los demás lenguajes del módulo, conviene que la función que **calcula** no sepa nada de ventanas, para poder probarla sin abrir ninguna:

```rust
fn calcular_propina(texto: &str) -> String {
    match texto.trim().parse::<f64>() {
        Ok(importe) if importe >= 0.0 => format!("Propina (10%): {:.2}", importe * 0.10),
        Ok(_) => String::from("El importe no puede ser negativo"),
        Err(_) => String::from("Eso no es un número"),
    }
}

fn main() {
    println!("{}", calcular_propina("100"));
    println!("{}", calcular_propina("-5"));
    println!("{}", calcular_propina("abc"));
}
```

Salida:

```text
Propina (10%): 10.00
El importe no puede ser negativo
Eso no es un número
```

## La ventana

Con `calcular_propina` ya probada, la ventana solo **recoge** el texto, **llama** a la función y **muestra** el resultado:

```toml
[dependencies]
eframe = "0.32"
```

```rust
use eframe::egui;

#[derive(Default)]
struct Propina {
    importe: String,
    resultado: String,
}

impl eframe::App for Propina {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.label("Importe:");
            ui.text_edit_singleline(&mut self.importe);
            if ui.button("Calcular").clicked() {
                self.resultado = calcular_propina(&self.importe);
            }
            ui.label(&self.resultado);
        });
    }
}

fn main() -> eframe::Result {
    eframe::run_native("Propina", eframe::NativeOptions::default(),
        Box::new(|_cc| Ok(Box::new(Propina::default()))))
}
```

Se lanza con `cargo run`. Este ejemplo se ha **comprobado con `cargo check`** (compila de verdad), pero no se ejecuta aquí: abrir una ventana no tiene sentido al generar una página web, y es justo lo que harías tú al probarlo en tu ordenador.

## Lo que cambia frente a la consola

| Consola | GUI (egui) |
|---|---|
| El programa pide datos **en orden** | El usuario decide **cuándo** y **qué** rellenar |
| `Console.ReadLine` / `io::stdin()` bloquea hasta que hay respuesta | `update` se llama **muchas veces por segundo**; lee el estado actual, no espera |
| El estado son variables locales de `main` | El estado vive en el `struct` de la app (aquí, `Propina`), porque `update` se vuelve a llamar desde cero |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Meter la lógica de cálculo dentro de `update` | Sepárala en una función aparte: se puede probar sin abrir una ventana |
| Guardar el resultado en una variable local de `update` | Se perdería en el siguiente fotograma: tiene que ir en el `struct` de la app |
| Esperar que el programa se bloquee esperando una pulsación | En el modo inmediato, el bucle **siempre sigue corriendo**; tú decides qué dibujar según el estado |

## Para practicar

Haz el [ejercicio 7.4](gui.md) (en tu ordenador, con ventana de verdad).
