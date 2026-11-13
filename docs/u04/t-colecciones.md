# 4.D Colecciones y asociados

## Valores que contienen otros valores

Un `struct` puede tener como campo un `Vec` de **otro** struct: es la forma natural de modelar «una cosa que tiene varias cosas» (una biblioteca con libros, un equipo con jugadores):

```rust
struct Jugador {
    nombre: String,
    puntos: u32,
}

struct Equipo {
    nombre: String,
    jugadores: Vec<Jugador>,
}

impl Equipo {
    fn new(nombre: &str) -> Equipo {
        Equipo { nombre: nombre.to_string(), jugadores: Vec::new() }
    }

    fn fichar(&mut self, nombre: &str) {
        self.jugadores.push(Jugador { nombre: nombre.to_string(), puntos: 0 });
    }

    fn anotar(&mut self, nombre: &str, puntos: u32) {
        if let Some(j) = self.jugadores.iter_mut().find(|j| j.nombre == nombre) {
            j.puntos += puntos;
        }
    }

    fn total_puntos(&self) -> u32 {
        self.jugadores.iter().map(|j| j.puntos).sum()
    }
}

fn main() {
    let mut equipo = Equipo::new("Halcones");
    equipo.fichar("Ana");
    equipo.fichar("Luis");
    equipo.anotar("Ana", 10);
    equipo.anotar("Luis", 7);

    for j in &equipo.jugadores {
        println!("{}: {} puntos", j.nombre, j.puntos);
    }
    println!("{}: {} puntos en total", equipo.nombre, equipo.total_puntos());
}
```

Salida:

```text
Ana: 10 puntos
Luis: 7 puntos
Halcones: 17 puntos en total
```

`iter_mut().find(...)` busca el jugador y permite **modificarlo** dentro del propio `Vec`, sin sacarlo ni volver a meterlo.

## Funciones asociadas y constantes: lo que pertenece al *tipo*, no a un valor

Rust no tiene `static` sobre campos sueltos, pero sí **funciones asociadas** (sin `self`, como `new`) y **constantes asociadas**, que pertenecen al tipo entero y no a un valor concreto:

```rust
struct Circulo {
    radio: f64,
}

impl Circulo {
    const PI: f64 = 3.14159;          // constante del tipo, no de un círculo concreto
    const MAXIMO_RADIO: f64 = 1000.0;

    fn new(radio: f64) -> Circulo {
        Circulo { radio: radio.min(Self::MAXIMO_RADIO) }
    }

    fn area(&self) -> f64 {
        Self::PI * self.radio * self.radio
    }

    fn unidad() -> Circulo {          // función asociada: fabrica un círculo concreto, el de radio 1
        Circulo::new(1.0)
    }
}

fn main() {
    let c = Circulo::new(5.0);
    println!("{:.2}", c.area());
    println!("{}", Circulo::MAXIMO_RADIO);
    println!("{:.2}", Circulo::unidad().area());
}
```

Salida:

```text
78.54
1000
3.14
```

`Self` (con mayúscula) dentro de un `impl` significa «el tipo sobre el que estoy escribiendo este bloque»: evita repetir `Circulo` por todas partes.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Sacar un valor de un `Vec` para modificarlo y no volver a meterlo | Usa `iter_mut()` para modificar **en el sitio** |
| Esperar `static` como en Java/C# | Usa una **función o constante asociada** dentro de `impl`, sin `self` |
| Repetir el nombre del tipo dentro de su propio `impl` | Usa `Self` |
| Sumar con un bucle manual en vez de un iterador | `.iter().map(...).sum()` es más corto y menos propenso a errores de índice |

## Para practicar

[U4.6 · Prueba: Cafetera y Taza](prueba.md), y luego el [proyecto de la unidad](proyecto.md) o el [cambio de rol](cambio-de-rol.md).
