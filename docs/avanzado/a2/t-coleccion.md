# A2.C Tu colección y qué pasa al compilar

## Una colección genérica propia

Con lo que ya sabes se puede escribir una colección propia. Una **pila** (*stack*) es una colección donde el último en entrar es el primero en salir: `apilar` añade por arriba y `desapilar` quita el de arriba:

```rust
struct Pila<T> {
    elementos: Vec<T>,
}

impl<T> Pila<T> {
    fn nueva() -> Self {
        Pila { elementos: Vec::new() }
    }

    fn apilar(&mut self, x: T) {
        self.elementos.push(x);
    }

    fn desapilar(&mut self) -> T {
        self.elementos.pop().expect("la pila está vacía")
    }

    fn cantidad(&self) -> usize {
        self.elementos.len()
    }
}

fn main() {
    let mut pila: Pila<String> = Pila::nueva();
    pila.apilar(String::from("uno"));
    pila.apilar(String::from("dos"));
    pila.apilar(String::from("tres"));

    println!("elementos: {}", pila.cantidad());
    println!("desapila: {}", pila.desapilar());
    println!("desapila: {}", pila.desapilar());
    println!("desapila: {}", pila.desapilar());

    let resultado = std::panic::catch_unwind(std::panic::AssertUnwindSafe(|| pila.desapilar()));
    match resultado {
        Ok(_) => println!("no debería llegar aquí"),
        Err(_) => println!("error: la pila está vacía"),
    }
}
```

Salida:

```text
elementos: 3
desapila: tres
desapila: dos
desapila: uno
error: la pila está vacía
```

Qué hay que fijarse:

* `Pila<T>` sirve para cualquier tipo, y el compilador sabe que `Pila<String>::desapilar()` devuelve un `String`.
* `.pop()` sobre un `Vec` ya devuelve `Option<T>`; `.expect(mensaje)` lo convierte en un *panic* con ese mensaje si está vacío, que es lo que pide el ejercicio. En Rust ya existe `Vec` como pila (`push`/`pop`); escribir `Pila<T>` sirve para aprender.
* El ejemplo atrapa el *panic* con `catch_unwind` solo para poder mostrar el mensaje de error en la salida; en un programa normal, dejarías que el *panic* terminase el programa (o usarías un `Result` en vez de `.expect()`, si el caso "vacío" es esperable).

## Qué pasa con los tipos al compilar

Los genéricos de Rust se **monomorfizan**: por cada tipo concreto con el que se usa una función o struct genérico, el compilador genera una **copia especializada** del código, como si la hubieras escrito tú a mano para ese tipo. No hay un tipo "genérico" compartido en tiempo de ejecución, ni conversión de tipos (*boxing*):

```rust
fn describir<T>(_valor: &T) -> String {
    format!("T ocupa {} bytes", std::mem::size_of::<T>())
}

fn main() {
    let a: Vec<i32> = Vec::new();
    let b: Vec<String> = Vec::new();

    // cada T concreto genera SU PROPIO código al compilar (monomorfización):
    // no hay un único "Vec<T>" compartido en tiempo de ejecución
    println!("{}", describir(&a));
    println!("{}", describir(&b));
    println!("{}", describir(&1i64));

    // sin boxing: un Vec<i32> guarda i32 de verdad, uno junto al otro en memoria
    let enteros = vec![1, 2, 3];
    println!("suma: {}", enteros.iter().sum::<i32>());
}
```

Salida:

```text
T ocupa 24 bytes
T ocupa 24 bytes
T ocupa 8 bytes
suma: 6
```

* `describir::<Vec<i32>>` y `describir::<Vec<String>>` son, tras compilar, dos funciones **distintas**: el compilador ya conoce `T` en cada una y puede optimizarla a fondo (es la base de por qué los genéricos de Rust no cuestan nada al ejecutar).
* La contrapartida es el **tamaño del binario**: usar una función genérica con muchos tipos distintos genera una copia por cada uno. Para la mayoría de programas no es un problema; en bibliotecas muy usadas, a veces se evita con trait objects (`dyn Trait`) para compartir una sola copia.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Escribir una colección propia cuando ya existe en la biblioteca | Usa `Vec`, `VecDeque`, `HashMap`... y escribe la tuya solo para aprender o si necesitas un comportamiento distinto |
| Usar `.unwrap()`/`.expect()` sin pensar en el caso vacío | Decide si de verdad es un error de programación (panic) o un caso esperable (`Option`/`Result`) |
| Preocuparte por el tamaño del binario antes de medirlo | La monomorfización rara vez es un problema real; mide antes de optimizar |
| Creer que los genéricos de Rust se "borran" como en otros lenguajes con tipos reificados a medias | Rust los **especializa** por completo: no hay coste ni en tiempo ni en tipo al ejecutar |

## Para practicar

Los ejercicios [A2.4 y A2.6](ejercicios.md) usan una pila y una caché genérica. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
