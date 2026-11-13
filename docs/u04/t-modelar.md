# 4.E Del problema al programa

## Cómo diseñar con `struct` y `trait`

Al pasar de un enunciado a código en Rust, conviene preguntarse:

1. **¿Qué datos hay que guardar?** Eso es un `struct` (o varios).
2. **¿Qué puede hacerse con esos datos?** Eso es el `impl`.
3. **¿Hay varias formas de la misma cosa?** Si son un conjunto cerrado y conocido, un `enum`; si son abiertas y pueden crecer, un `trait`.
4. **¿Qué puede salir mal?** Decide qué devuelve `Result` y qué, si de verdad es imposible, puede hacer `panic!`.

## Del programa con bucles al programa con structs

```rust
// Antes: variables sueltas
fn main() {
    let nombre1 = "Ana";
    let nota1 = 8.5;
    let nombre2 = "Luis";
    let nota2 = 6.0;
    println!("{nombre1}: {nota1}, {nombre2}: {nota2}");
}
```

Salida:

```text
Ana: 8.5, Luis: 6
```

se convierte en:

```rust
struct Alumno {
    nombre: String,
    nota: f64,
}

impl Alumno {
    fn aprobado(&self) -> bool {
        self.nota >= 5.0
    }
}

fn main() {
    let alumnos = vec![
        Alumno { nombre: String::from("Ana"), nota: 8.5 },
        Alumno { nombre: String::from("Luis"), nota: 4.0 },
    ];
    for a in &alumnos {
        println!("{}: {} ({})", a.nombre, a.nota, if a.aprobado() { "aprobado" } else { "suspenso" });
    }
}
```

Salida:

```text
Ana: 8.5 (aprobado)
Luis: 4 (suspenso)
```

Ahora **añadir un alumno** es añadir un elemento a la lista, no inventar `nombre3` y `nota3`; y una regla como «aprobado» vive **junto al dato** al que pertenece, no suelta en `main`.

## Comportamiento que cambia: pasar una función como parámetro

Rust no tiene «pasar un método» como en C# o Java, pero sus **closures** (funciones anónimas) hacen lo mismo y son muy usadas:

```rust
fn filtrar<T, F: Fn(&T) -> bool>(valores: &[T], cond: F) -> Vec<&T> {
    valores.iter().filter(|v| cond(v)).collect()
}

fn main() {
    let numeros = vec![1, 2, 3, 4, 5, 6, 7, 8];

    let pares = filtrar(&numeros, |n| n % 2 == 0);
    let mayores_de_4 = filtrar(&numeros, |n| *n > 4);

    println!("{:?}", pares);
    println!("{:?}", mayores_de_4);
}
```

Salida:

```text
[2, 4, 6, 8]
[5, 6, 7, 8]
```

`|n| n % 2 == 0` es una closure: una función sin nombre que se pasa como dato. Es el mismo mecanismo que ya usaste con `filter` y `map` en [3.1](../u03/01-vectores.md).

## Señales de un mal diseño

| Señal | Qué suele significar |
|---|---|
| Un `struct` con 15 campos y un `impl` con un solo método gigante | Probablemente son **varios conceptos** mezclados: separa en varios tipos |
| Muchos `if`/`match` repetidos sobre el mismo `enum`, esparcidos por el código | Mete esa lógica en un método del `impl` de ese `enum` |
| Pasar 6 parámetros sueltos a una función, varios de ellos relacionados | Agrúpalos en un `struct` |
| Un campo `pub` que cualquiera puede dejar en un estado inválido | Hazlo privado y da un método que mantenga la regla |

## Para practicar

El proyecto de la unidad: [U4.7](proyecto.md) o [U4.8 · Cambio de rol](cambio-de-rol.md).
