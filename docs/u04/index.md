# U4 · Programación orientada a objetos

Pasar del código suelto a **structs y métodos**: campos, funciones asociadas, validación, encapsulamiento y colecciones de valores. Termina con un proyecto personal o en grupo.

## Teoría

* [4.A Structs y métodos](t-structs.md): datos y comportamiento, en vez de clases.
* [4.B Encapsulamiento](t-encapsulamiento.md): privado por defecto, validar al crear.
* [4.C Constructores y enumerados](t-constructores.md): funciones asociadas, `Default` y `enum` con datos.
* [4.D Colecciones y asociados](t-colecciones.md): structs anidados, funciones y constantes asociadas.
* [4.E Del problema al programa](t-modelar.md): cómo decidir structs y traits, y closures como parámetro.

## Antes de empezar: qué debes dominar

* Struct, valor, campo y método.
* Funciones asociadas como constructores (y varias con nombre distinto) y el trait `Default`.
* Encapsulamiento: todo privado por defecto, y métodos de solo lectura.
* Enumerados (`enum`) con datos propios, y los traits `Display`/`PartialEq`.
* Colecciones de structs y validación con `Result`.

## Ejercicios de la unidad

| Bloque | Ejercicios |
|---|---|
| [U4.1 · Repaso de las unidades 1 a 3](repaso.md) | 1 |
| [U4.2 · POO I (ejercicios 1 al 5)](poo-1.md) | 5 |
| [U4.3 · POO II (ejercicios 6 al 10)](poo-2.md) | 5 |
| [U4.4 · Robots (parte 1)](robots-1.md) | 2 |
| [U4.5 · Robots (parte 2 y reto)](robots-2.md) | 1 |
| [U4.6 · Prueba: Cafetera y Taza](prueba.md) | 2 |
| [U4.7 · Juego del ahorcado (grupos)](proyecto.md) | 1 |
| [U4.8 · Cambio de rol: explícamelo tú (grupos)](cambio-de-rol.md) | 2 |

## Equivalencias con la POO clásica

Rust no tiene clases ni herencia: separa los **datos** (`struct`) del **comportamiento** (`impl`), y usa **traits** en vez de interfaces. La tabla compara con lo que se ve en otros lenguajes del módulo:

| Idea | En Rust |
|---|---|
| Clase | `struct Persona { nombre: String, edad: u32 }` (los datos) + `impl Persona { ... }` (los métodos), por separado |
| Constructor | Una **función asociada**: `impl Persona { fn new(nombre: &str, edad: u32) -> Persona { ... } }`; se llama `Persona::new(...)` |
| Valores por defecto | El trait `Default`: `#[derive(Default)]` o un `impl Default for ...` propio |
| Propiedad calculada de solo lectura | Un **método** normal: `fn imc(&self) -> f64 { self.peso / (self.altura * self.altura) }` |
| Atributo que solo cambia desde dentro | Un campo **privado** (lo normal) con un método público para leerlo y otro, interno, para cambiarlo |
| Privado / público | **Privado por defecto**; `pub` lo hace visible fuera del módulo |
| Validar al crear | Un `new` que devuelve `Result<T, E>`, o que hace `panic!` si el dato es imposible |
| Herencia | No existe. Se usa **composición** (un `struct` que contiene a otro) o **traits** |
| Interfaz / clase abstracta | **`trait`**: define métodos que varios tipos implementan; puede traer una implementación **por defecto** |
| Enumerado | **`enum`**: además de nombres, cada variante puede **llevar datos propios** |
| Datos con igualdad por valor | `#[derive(PartialEq, Eq, Debug, Clone)]` sobre el `struct` |
| Jerarquía cerrada | Un **`enum`**: el compilador obliga a tratar todas las variantes en cada `match` |
| Representación en texto | El trait `std::fmt::Display` (o `#[derive(Debug)]` para depurar con `{:?}`) |
| Igualdad | El trait `PartialEq` (normalmente, `derive`) |

## Antes de pasar a los ejercicios

Cuando hayas leído y practicado la teoría, marca la casilla para **desbloquear** los ejercicios de esta unidad:

<div class="ej-check" data-unit="u04"></div>
