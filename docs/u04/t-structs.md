# 4.A Structs y métodos

Hasta ahora los datos (variables) y las operaciones (funciones) iban por separado. Rust los **agrupa**, pero en **dos piezas distintas**: un **`struct`** con los datos y un bloque **`impl`** con las operaciones que trabajan sobre ellos. Es la misma idea que una clase, pero sin mezclar las dos cosas en un solo sitio.

| Concepto | Qué es | Ejemplo |
|---|---|---|
| **`struct`** | La **plantilla** de datos (como una clase, pero sin métodos dentro) | `struct Libro { ... }` |
| **Valor** (no «objeto»: en Rust se dice *instancia* o, más a menudo, *valor*) | Un **dato concreto** creado a partir del struct | «El Quijote», «Dune» |
| **Campo** | Un dato que guarda cada valor (su **estado**) | `titulo`, `paginas`, `leidas` |
| **Método** | Una operación en el bloque `impl`, que recibe `self` | `leer(...)`, `progreso()` |
| **Función asociada** | Una operación del tipo que **no** recibe `self` (el constructor, normalmente) | `Libro::new(...)` |

## Tu primer struct

```rust
struct Libro {
    titulo: String,
    paginas: u32,
    leidas: u32,
}

impl Libro {
    fn new(titulo: &str, paginas: u32) -> Libro {
        Libro { titulo: titulo.to_string(), paginas, leidas: 0 }
    }

    fn leer(&mut self, cantidad: u32) {
        self.leidas += cantidad;
    }

    fn progreso(&self) -> u32 {
        self.leidas * 100 / self.paginas
    }
}

impl std::fmt::Display for Libro {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "Libro({}, {} págs, leídas {})", self.titulo, self.paginas, self.leidas)
    }
}

fn main() {
    let mut dune = Libro::new("Dune", 400);
    dune.leer(100);
    println!("{dune}");
    println!("progreso de Dune: {}%", dune.progreso());

    let quijote = Libro::new("El Quijote", 1000);
    println!("{quijote}");
}
```

Salida:

```text
Libro(Dune, 400 págs, leídas 100)
progreso de Dune: 25%
Libro(El Quijote, 1000 págs, leídas 0)
```

Observa:

1. El `struct` solo declara los **campos**; los métodos van en un bloque `impl` aparte (se pueden tener varios bloques `impl` para el mismo tipo).
2. `&self` es «este método **mira** el valor»; `&mut self` es «este método **puede cambiarlo**» (por eso `leer` necesita `&mut self` y `dune` tiene que ser `let mut`).
3. `Libro::new(...)` es una **función asociada**, no un método: se llama con `::`, no con un valor delante. Hace el papel de constructor.
4. `impl std::fmt::Display` define **cómo se convierte el valor en texto**: con eso, `{dune}` dentro de `println!` ya funciona.

!!! note "¿Objeto o valor?"
    En Rust no se suele decir «objeto»: se dice **valor** (o *instancia*, en documentación más formal). La diferencia no es solo de palabras: un valor de un `struct` se comporta como cualquier otro valor de Rust (un entero, un texto), con las mismas reglas de propiedad, préstamo y movimiento que ya conoces. No hay una «identidad» de objeto aparte del dato.

## Propiedad: ¿quién tiene el libro?

A diferencia de C# o Java, asignar un valor **no lo comparte**: lo **mueve**. Si quieres que dos variables puedan usar el mismo dato, tienes que **prestarlo** (una referencia) o **clonarlo**:

```rust
#[derive(Clone)]
struct Libro { titulo: String, leidas: u32 }

fn main() {
    let dune = Libro { titulo: String::from("Dune"), leidas: 100 };
    let copia = dune.clone();    // una copia independiente

    // let otro = dune; println!("{}", dune.titulo); // no compilaría: dune ya se movió a otro

    println!("{} {}", copia.titulo, copia.leidas);
    println!("{} {}", dune.titulo, dune.leidas);     // dune sigue siendo válido: solo clonamos, no movimos
}
```

Salida:

```text
Dune 100
Dune 100
```

## Cómo se escribe en Rust

| Necesito... | En Rust |
|---|---|
| Declarar los datos | `struct Libro { titulo: String, paginas: u32 }` |
| Declarar las operaciones | `impl Libro { fn progreso(&self) -> u32 { ... } }` |
| Constructor | Una función asociada: `fn new(...) -> Libro { ... }` |
| Crear un valor | `Libro::new("Dune", 400)` o la forma literal `Libro { titulo: ..., paginas: ... }` |
| Llamar a un método | `dune.leer(100)` (Rust añade el `&` o `&mut` automáticamente) |
| El propio valor, dentro de un método | `self`, `&self` o `&mut self` |
| Texto del valor | `impl std::fmt::Display for Libro` |
| Inspeccionar el valor (depurar) | `#[derive(Debug)]` sobre el struct, y `{:?}` o `{:#?}` al imprimir |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Mezclar datos y métodos en el mismo bloque | Van separados: `struct` para los campos, `impl` para los métodos |
| Olvidar `&mut self` en un método que cambia un campo | Sin él, el compilador avisa: «cannot borrow as mutable» |
| Usar una variable después de moverla (`let otro = dune;`) | Pide una referencia (`&dune`) o clónala (`dune.clone()`) si de verdad necesitas dos |
| Llamar al constructor como si fuera un método (`libro.new(...)`) | Es una función **asociada**: `Libro::new(...)` |
| Olvidar `#[derive(Debug)]` para poder usar `{:?}` | Añádelo sobre el `struct` cuando solo necesites inspeccionar, no un texto bonito |

## Para practicar

Las primeras prácticas están en [U4.2](poo-1.md). La [tabla de equivalencias](index.md#equivalencias-con-la-poo-clasica) del índice resume la sintaxis de Rust.
