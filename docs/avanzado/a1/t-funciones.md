# A1.A Funciones como valores

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5 (variables, estructuras de control, colecciones y structs). Si alguna idea se te resiste, vuelve a la unidad correspondiente.

Hasta ahora has **llamado** funciones. Aquí das un paso más: tratar una función (o una *closure*, una función anónima que puede capturar variables de alrededor) **como un valor**. Se puede guardar en una variable, pasar a otra función y devolver desde una función.

## Tres formas de capturar: `Fn`, `FnMut` y `FnOnce`

Una closure que no captura nada es como una función normal. Si captura variables, el compilador decide **cómo** las usa y le asigna uno de estos tres traits:

| Trait | La closure... | Ejemplo |
|---|---|---|
| `Fn` | Solo **lee** lo que captura; se puede llamar varias veces | multiplicar por un `factor` guardado |
| `FnMut` | **Modifica** algo que captura; se puede llamar varias veces | un contador que avanza cada llamada |
| `FnOnce` | **Consume** (se queda con la propiedad de) algo que captura; solo se puede llamar una vez | mover un `String` fuera de la closure |

Un parámetro de función que recibe una closure se escribe `f: impl Fn(i32) -> i32` (o genérico, `f: F` con `where F: Fn(i32) -> i32`). Para **devolver** una closure hace falta `impl Fn(...)` (o `Box<dyn Fn(...)>` si el tipo concreto puede variar), y casi siempre con `move`, para que la closure sea dueña de lo que captura y pueda sobrevivir a la función que la crea.

```rust
fn aplicar_dos_veces(f: impl Fn(i32) -> i32, x: i32) -> i32 {
    f(f(x))
}

// Devuelve una función que "recuerda" factor
fn multiplicador(factor: i32) -> impl Fn(i32) -> i32 {
    move |n| n * factor
}

// La closure captura cuenta y la modifica: necesita FnMut
fn crear_contador() -> impl FnMut() -> i32 {
    let mut cuenta = 0;
    move || {
        cuenta += 1;
        cuenta
    }
}

fn doble(n: i32) -> i32 {
    n * 2
}

fn main() {
    let cuadrado = |n: i32| n * n; // closure anónima guardada en una variable
    println!("doble(4) = {}", doble(4));
    println!("cuadrado(4) = {}", cuadrado(4));
    println!("aplicar dos veces doble a 3 = {}", aplicar_dos_veces(doble, 3));
    println!("multiplicador(5)(7) = {}", multiplicador(5)(7));

    let mut a = crear_contador();
    let mut b = crear_contador();
    println!("contador A: {} {} {}", a(), a(), a());
    println!("contador B: {}", b());
}
```

Salida:

```text
doble(4) = 8
cuadrado(4) = 16
aplicar dos veces doble a 3 = 12
multiplicador(5)(7) = 35
contador A: 1 2 3
contador B: 1
```

Qué hace cada línea:

| Línea de salida | Qué demuestra |
|---|---|
| `doble(4)` y `cuadrado(4)` | Una función con nombre y una closure guardada en una variable se llaman igual |
| `aplicar dos veces doble a 3` | Una función recibe **otra función** como parámetro (aquí, `doble`, una función normal, sirve porque también implementa `Fn`) |
| `multiplicador(5)(7)` | Una función **devuelve una closure** que recuerda `factor` |
| `contador A` y `contador B` | Cada contador captura **su propia** `cuenta`: no se pisan |

## La trampa de capturar por referencia

Si la closure no usa `move`, captura **referencias** a las variables de alrededor, y todas las closures de un bucle `for` que compartiesen la misma variable se pisarían. Con `move`, cada vuelta copia su propio valor:

```rust
fn main() {
    let mut acciones: Vec<Box<dyn Fn() -> i32>> = Vec::new();
    for i in 0..3 {
        acciones.push(Box::new(move || i)); // "move" copia i a cada closure
    }
    let valores: Vec<String> = acciones.iter().map(|a| a().to_string()).collect();
    println!("{}  <- cada closure guarda su propio i", valores.join(" "));
}
```

Salida:

```text
0 1 2  <- cada closure guarda su propio i
```

## `FnOnce`: cuando la closure se queda con el valor

```rust
fn llama_una_vez(f: impl FnOnce() -> String) -> String {
    f()
}

fn main() {
    let texto = String::from("hola");
    // move consume "texto": la closure solo puede llamarse una vez (FnOnce)
    let consumidora = move || texto;
    println!("{}", llama_una_vez(consumidora));
}
```

Salida:

```text
hola
```

`texto` se mueve dentro de la closure con `move`; como la closure **se queda con la propiedad** de `texto` para devolverlo, solo puede ejecutarse una vez (el compilador no deja llamarla dos veces). Es el trait más general: toda `Fn` es también `FnMut`, y toda `FnMut` es también `FnOnce`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar `move` al devolver una closure que usa variables locales | El compilador avisa (`closure may outlive the current function`); añade `move` |
| Esperar que cada closure de un bucle `for` sin `move` tenga su propia copia | Sin `move` capturan por referencia y comparten la variable; usa `move` |
| Pedir `Fn` cuando la closure modifica algo capturado | Pide `FnMut` |
| Llamar dos veces a una closure `FnOnce` | Solo se puede llamar una vez; si necesitas llamarla varias, no debe consumir nada |

## Para practicar

Los ejercicios [A1.1 y A1.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
