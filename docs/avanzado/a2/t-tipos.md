# A2.A Tipos parametrizados

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5, en especial structs, traits y colecciones.

## Por qué existen

Imagina una caja que guarda **un valor de cualquier tipo**. Sin genéricos hay dos malas opciones: escribir un struct por cada tipo (`CajaI32`, `CajaTexto`...) o guardar algo tipo `Box<dyn Any>` y **convertir** al sacar el valor, perdiendo la comprobación de tipos. Los **tipos parametrizados** (genéricos) resuelven esto: escribes el struct o la función **una vez**, con un **parámetro de tipo** (`T`) que se concreta al usarlo, y el compilador sigue comprobando todo.

Mira lo que pasa guardando "cualquier cosa" frente a una colección genérica:

```rust
fn main() {
    // Vec<Box<dyn Any>> guarda "cualquier cosa": hay que preguntar el tipo y convertir
    use std::any::Any;
    let variada: Vec<Box<dyn Any>> = vec![Box::new(1i32), Box::new(2i32), Box::new("tres")];
    let mut suma = 0;
    for valor in &variada {
        match valor.downcast_ref::<i32>() {
            Some(n) => suma += n,
            None => println!("no es un i32: se ignora"),
        }
    }
    println!("suma de los i32: {}", suma);

    // Vec<i32> solo admite i32: el error se detecta al COMPILAR
    let normal: Vec<i32> = vec![1, 2];
    // normal.push("tres");      // error[E0308]: mismatched types
    println!("Vec<i32>: suma {}", normal.iter().sum::<i32>());
}
```

Salida:

```text
no es un i32: se ignora
suma de los i32: 3
Vec<i32>: suma 3
```

## La sintaxis en Rust

* Un struct genérico declara su parámetro entre ángulos: `struct Caja<T> { valor: T }`. Un `impl<T> Caja<T>` añade métodos; si necesitas que `T` sepa hacer algo (como mostrarse con `{}`), lo restringes ahí: `impl<T: Display> Caja<T>`.
* Una función genérica lo declara tras su nombre: `fn primero<T>(lista: &[T]) -> T`. Casi siempre **no hace falta indicar el tipo al llamarla**: el compilador lo **deduce** de los argumentos.
* Puede haber varios parámetros de tipo: `fn intercambiar<A, B>(a: A, b: B) -> (B, A)`. Por convención se usan letras mayúsculas sueltas, salvo que aclarar con un nombre (`TClave`) ayude a leer.
* Los genéricos funcionan también en `enum`, `trait` y las funciones asociadas de un `impl`.

## Un ejemplo

```rust
struct Caja<T> {
    valor: T,
}

impl<T: std::fmt::Display> Caja<T> {
    fn nuevo(valor: T) -> Self {
        Caja { valor }
    }
}

impl<T: std::fmt::Display> std::fmt::Display for Caja<T> {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "Caja({})", self.valor)
    }
}

// el compilador deduce T a partir del argumento
fn primero<T: Clone>(lista: &[T]) -> T {
    lista[0].clone()
}

fn intercambiar<A, B>(a: A, b: B) -> (B, A) {
    (b, a)
}

fn main() {
    let caja_numero = Caja::nuevo(42);
    let caja_texto = Caja::nuevo(String::from("hola"));
    println!("{} y {}", caja_numero.valor + 1, caja_texto.valor.to_uppercase());
    println!("{}", caja_numero);
    println!("{}", caja_texto);

    println!("{}", primero(&["a", "b", "c"]));
    println!("{}", primero(&[10, 20]));

    let (x, y) = intercambiar(1, "uno");
    println!("{} / {}", x, y);
}
```

Salida:

```text
43 y HOLA
Caja(42)
Caja(hola)
a
10
uno / 1
```

`Caja<i32>` y `Caja<String>` son tipos **distintos para el compilador**, aunque vengan del mismo struct genérico (se explica más en [A2.C](t-coleccion.md)). E `intercambiar` devuelve una tupla de tipos `(B, A)`: el compilador sabe que `x` es un `i32` y `y` un `&str`, sin que lo escribas.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Guardar "cualquier cosa" con `Box<dyn Any>` por comodidad | Usa un tipo genérico; `Any` es para los casos raros donde de verdad no se puede saber el tipo al compilar |
| Escribir `T::default()` o comparar con `<` dentro de una función genérica sin más | `T` no se sabe hacer eso: hay que **restringirlo** ([A2.B](t-restricciones.md)) |
| Indicar el tipo cuando se puede deducir | Deja que el compilador lo deduzca, salvo que haya ambigüedad (`::<Vec<i32>>()`) |
| Ponerle nombres largos al parámetro de tipo sin necesidad | `T` basta; usa `K`/`V` o `TClave`/`TValor` cuando haya varios con sentido distinto |

## Para practicar

Los ejercicios [A2.1 y A2.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
