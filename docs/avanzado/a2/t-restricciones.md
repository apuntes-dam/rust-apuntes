# A2.B Restricciones y variación

## Restringir el tipo

Dentro de una función genérica, `T` es «cualquier tipo», y de «cualquier tipo» no se sabe nada: no se puede compararlo con `<`, ni sumarlo, ni crear un valor por defecto. Una **restricción** (*trait bound*) dice qué traits tiene que implementar `T`, y a cambio permite usar esas capacidades:

| Restricción | Significa | Permite |
|---|---|---|
| `T: PartialOrd` | `T` se puede comparar | `a >= b`, `a < b` |
| `T: Default` | `T` tiene un valor por defecto | `T::default()` |
| `T: Copy` | `T` se copia en vez de moverse | usar `x` después de pasarlo por valor |
| `T: Add<Output = T>` | `T` admite el operador `+` | `a + b` |
| `T: MiTrait` | `T` cumple un trait propio | llamar a sus métodos |

```rust
use std::ops::Add;

fn mayor<T: PartialOrd>(a: T, b: T) -> T {
    if a >= b { a } else { b }
}

// T: Add<Output = T> permite usar + entre dos T
fn suma_todos<T: Add<Output = T> + Default + Copy>(elementos: &[T]) -> T {
    let mut total = T::default();
    for &x in elementos {
        total = total + x;
    }
    total
}

fn main() {
    println!("{}", mayor(3, 9));
    println!("{}", mayor("pera", "casa"));
    println!("{}", mayor(2.5, 1.5));

    println!("{}", suma_todos(&[1, 2, 3]));
    println!("{}", suma_todos(&[1.5, 2.5]));
}
```

Salida:

```text
9
pera
2.5
6
4
```

`mayor` solo compila con tipos que sepan compararse (`i32`, `&str`, `f64`...). Si lo llamas con un tipo propio que no implementa `PartialOrd`, el compilador lo rechaza **antes de ejecutar**. Combinar varias restricciones con `+` (`Add<Output = T> + Default + Copy`) es la forma de Rust de pedir "varias capacidades a la vez": no hay un único trait numérico como el `INumber<T>` de otros lenguajes, pero el resultado es parecido.

## `impl Trait` frente a un genérico explícito

Pedir un parámetro que cumpla un trait se puede escribir de dos formas equivalentes:

```rust
trait Saluda {
    fn saludar(&self) -> String;
}

struct Persona {
    nombre: String,
}

impl Saluda for Persona {
    fn saludar(&self) -> String {
        format!("hola, {}", self.nombre)
    }
}

// "impl Trait" en el parámetro es azúcar sintáctico de un genérico con "where"
fn saludar_con_impl(x: impl Saluda) -> String {
    x.saludar()
}

fn saludar_generico<T>(x: T) -> String
where
    T: Saluda,
{
    x.saludar()
}

fn main() {
    let ana = Persona { nombre: String::from("Ana") };
    println!("{}", saludar_con_impl(ana));
    let eva = Persona { nombre: String::from("Eva") };
    println!("{}", saludar_generico(eva));
}
```

Salida:

```text
hola, Ana
hola, Eva
```

`impl Saluda` en la firma es azúcar sintáctico: el compilador lo trata exactamente como el `T` genérico con `where T: Saluda`. La diferencia aparece cuando necesitas **nombrar** el tipo (por ejemplo, para que dos parámetros sean del mismo tipo concreto): ahí sí hace falta el genérico explícito, `fn f<T: Saluda>(a: T, b: T)`.

## ¿Un `Vec<i32>` es un `Vec<Box<dyn Any>>`?

No, y es una protección: los genéricos de Rust son **invariantes** y se **monomorfizan** (cada `T` concreto genera su propio código al compilar, no hay un tipo común en tiempo de ejecución entre `Vec<i32>` y `Vec<String>`). Si necesitas guardar valores de tipos distintos que comparten comportamiento, la herramienta es un **trait object**, `Vec<Box<dyn MiTrait>>`: ahí sí todos los elementos cumplen el mismo trait, aunque sean structs distintos por detrás (se explica más en la [unidad 5](../../u05/t-polimorfismo.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Comparar con `<` valores de tipo `T` sin restricción | Añade `T: PartialOrd` (o el trait que corresponda) |
| Usar `+`/`*` sobre `T` sin restricción | Añade `T: Add<Output = T>` (o el trait de ese operador) |
| Pasar un valor por valor y querer usarlo después | `T: Copy` o `T: Clone`, o pasar una referencia |
| Guardar tipos distintos esperando que `Vec<T>` lo permita | Usa un trait object, `Vec<Box<dyn Trait>>` |

## Para practicar

Los ejercicios [A2.3 y A2.5](ejercicios.md) usan restricciones y funciones como parámetro. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
