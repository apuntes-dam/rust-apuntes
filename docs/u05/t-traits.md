# 5.A Traits: contratos compartidos

Rust **no tiene herencia de clases**: no se puede crear un `struct` «a partir de» otro. En su lugar usa **composición** (un struct contiene a otro) y **`trait`**, que es su equivalente a una interfaz: una lista de métodos que **cualquier tipo** puede prometer cumplir, sin importar su forma por dentro.

| En otros lenguajes | En Rust |
|---|---|
| Interfaz | `trait` |
| «La clase `Perro` implementa `Hablador`» | «El tipo `Perro` **implementa** el trait `Hablador`» (`impl Hablador for Perro`) |
| Relación «es un» por herencia | Relación «**se comporta como**» por trait |

## Definir y aplicar un trait

```rust
trait Animal {
    fn nombre(&self) -> &str;
    fn sonido(&self) -> String;

    fn presentarse(&self) -> String {               // con cuerpo: ver 5.C
        format!("{} dice {}", self.nombre(), self.sonido())
    }
}

struct Perro { nombre: String }
struct Gato { nombre: String }

impl Animal for Perro {
    fn nombre(&self) -> &str { &self.nombre }
    fn sonido(&self) -> String { String::from("guau") }
}

impl Animal for Gato {
    fn nombre(&self) -> &str { &self.nombre }
    fn sonido(&self) -> String { String::from("miau") }
}

fn main() {
    let rex = Perro { nombre: String::from("Rex") };
    let misi = Gato { nombre: String::from("Misi") };
    println!("{}", rex.presentarse());
    println!("{}", misi.presentarse());
}
```

Salida:

```text
Rex dice guau
Misi dice miau
```

`Perro` y `Gato` **no tienen ninguna relación entre sí**: no heredan de nada. Solo comparten que los dos implementan `Animal`. Eso basta para tratarlos de forma parecida.

## Composición en vez de herencia

Cuando en otro lenguaje dirías «`PerroGuardian` **es un** `Perro`, pero más», en Rust se suele decir «`PerroGuardian` **tiene un** `Perro`»:

```rust
struct Perro { nombre: String }

impl Perro {
    fn ladrar(&self) -> String {
        format!("{}: ¡guau!", self.nombre)
    }
}

struct PerroGuardian {
    base: Perro,           // composición: «tiene un» Perro, en vez de heredar de él
    zona: String,
}

impl PerroGuardian {
    fn vigilar(&self) -> String {
        format!("{} vigila {}", self.base.nombre, self.zona)
    }
}

fn main() {
    let rex = PerroGuardian { base: Perro { nombre: String::from("Rex") }, zona: String::from("el jardín") };
    println!("{}", rex.base.ladrar());   // se reutiliza el método del Perro de dentro
    println!("{}", rex.vigilar());
}
```

Salida:

```text
Rex: ¡guau!
Rex vigila el jardín
```

Es un poco más explícito que la herencia (`rex.base.ladrar()` en vez de `rex.ladrar()`), pero evita un problema clásico de la herencia: cambiar `Perro` nunca puede romper `PerroGuardian` de formas inesperadas, porque la relación es clara y a la vista.

## Traits de la biblioteca estándar

Muchos de los traits que más se usan ya vienen hechos; los implementas sobre tu tipo con `impl Trait for TuTipo` o, si siguen un patrón sencillo, con `#[derive(...)]`:

| Trait | Para qué | Se usa con |
|---|---|---|
| `Display` | Texto para el usuario | `{}` en `println!` |
| `Debug` | Texto para depurar | `{:?}` |
| `PartialEq` | Comparar con `==` | `#[derive(PartialEq)]` |
| `Clone` | Copiar un valor a propósito | `.clone()`, `#[derive(Clone)]` |
| `Default` | Un valor «vacío» razonable | `Tipo::default()`, `#[derive(Default)]` |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Buscar `class B : A` para heredar | No existe: usa un `trait` compartido o composición |
| Olvidar `impl Trait for Tipo` y solo escribir los métodos sueltos | Un método de un trait **tiene** que ir dentro de ese bloque `impl` |
| Pedir a un trait un método que no está en su definición | Añádelo al `trait`; todos los que lo implementen tendrán que escribirlo (o usar el valor por defecto, ver [5.C](t-default.md)) |
| Pensar que `impl Animal for Perro` «convierte» a `Perro` en otra cosa | `Perro` sigue siendo `Perro`; solo promete los métodos de `Animal` |

## Para practicar

Primeros ejercicios de traits y jerarquías en [U5.1](herencia.md), ejercicios 5.1 a 5.3.
