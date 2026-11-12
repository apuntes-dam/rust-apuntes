# 2.1 Condicionales: decidir qué hacer

Un programa no ejecuta siempre las mismas líneas: **decide** según los datos. Para decidir usa una **condición**, una expresión cuyo resultado es verdadero o falso (un valor **booleano**, `bool`), y las estructuras `if` y `match`.

## `if`, `else if` y `else`

La estructura completa tiene tres partes: lo que se hace **si** se cumple la condición, lo que se hace si **no** pero se cumple otra, y lo que se hace en **cualquier otro caso**.

```rust
fn calificacion(nota: i32) -> &'static str {
    if nota < 0 || nota > 10 {
        "Nota no válida"
    } else if nota < 5 {
        "Insuficiente"
    } else if nota < 6 {
        "Suficiente"
    } else if nota < 7 {
        "Bien"
    } else if nota < 9 {
        "Notable"
    } else {
        "Sobresaliente"
    }
}

fn main() {
    for nota in [3, 5, 6, 8, 10, 11] {
        println!("{nota}: {}", calificacion(nota));
    }
}
```

Salida:

```text
3: Insuficiente
5: Suficiente
6: Bien
8: Notable
10: Sobresaliente
11: Nota no válida
```

Fíjate en dos cosas propias de Rust:

* La condición **no lleva paréntesis** (`if nota < 5 {`), pero las **llaves son obligatorias**.
* La condición tiene que ser un **`bool`** de verdad. No existe «lo que vale 0 es falso»: `if nota { ... }` con un número **no compila**.

Se evalúan **en orden** y solo se ejecuta la **primera** rama que se cumple, por eso `nota < 6` no necesita comprobar también `nota >= 5`.

## Operadores de comparación y lógicos

| Operador | Significa | Ejemplo |
|---|---|---|
| `==` `!=` | igual, distinto | `edad == 18` |
| `<` `>` `<=` `>=` | menor, mayor, menor o igual, mayor o igual | `nota >= 5` |
| `&&` | **y** (las dos condiciones) | `edad >= 18 && edad < 65` |
| `\|\|` | **o** (al menos una) | `dia == "sábado" \|\| dia == "domingo"` |
| `!` | **no** (invierte) | `!terminado` |

`&&` y `||` **cortocircuitan**: si con el primer lado ya se sabe el resultado, el segundo ni se evalúa (`n != 0 && 10 / n > 1` no divide nunca entre cero).

## `if` es una expresión

En Rust `if` **devuelve un valor**, así que no hace falta un operador `? :` como en otros lenguajes:

```rust
fn main() {
    let edad = 20;
    let tipo = if edad >= 18 { "adulto" } else { "menor" };
    println!("Eres {tipo}");

    let n = -7;
    let valor_absoluto = if n < 0 { -n } else { n };
    println!("{valor_absoluto}");
}
```

Salida:

```text
Eres adulto
7
```

Las dos ramas tienen que dar el **mismo tipo**: `if x { 1 } else { "uno" }` no compila.

## Selección múltiple con `match`

Cuando comparas **una misma variable con muchos valores**, `match` es más claro que una cadena de `else if`. Admite valores sueltos, varios a la vez con `|`, **rangos** con `..=` y una condición extra con `if` (*guarda*):

```rust
fn tipo_impositivo(renta: f64) -> u32 {
    match renta {
        r if r < 10_000.0 => 5,
        r if r < 20_000.0 => 15,
        r if r < 35_000.0 => 20,
        r if r < 60_000.0 => 30,
        _ => 45,
    }
}

fn dias_del_mes(mes: u32) -> u32 {
    match mes {
        2 => 28,
        4 | 6 | 9 | 11 => 30,
        1..=12 => 31,
        _ => 0,
    }
}

fn main() {
    println!("{} %", tipo_impositivo(25_000.0));
    println!("{} {} {} {}", dias_del_mes(2), dias_del_mes(4), dias_del_mes(8), dias_del_mes(13));
}
```

Salida:

```text
20 %
28 30 31 0
```

!!! tip "`match` comprueba que no te dejas ningún caso"
    El compilador **exige que `match` cubra todos los valores posibles**. Para los números, eso obliga a cerrar con el brazo comodín `_`; para un `enum`, a nombrar todas las variantes. Si añades una variante nueva más tarde, el compilador te marca cada `match` que olvidaste actualizar. En C# o Java esto es solo un aviso, o ni eso.

`match` también es una expresión (da un valor, como `if`) y se puede **desestructurar** lo que se compara:

```rust
fn main() {
    let punto = (3, 0);
    match punto {
        (0, 0) => println!("el origen"),
        (x, 0) => println!("en el eje X, a {x}"),
        (0, y) => println!("en el eje Y, a {y}"),
        (x, y) => println!("en ({x}, {y})"),
    }

    let letra = 'k';
    let tipo = match letra {
        'a' | 'e' | 'i' | 'o' | 'u' => "vocal",
        'a'..='z' => "consonante",
        _ => "otro carácter",
    };
    println!("{letra} es {tipo}");
}
```

Salida:

```text
en el eje X, a 3
k es consonante
```

## `if let`: cuando solo importa un caso

Con un `Option` (un valor que puede faltar) o un `Result`, `if let` evita escribir un `match` entero cuando solo te interesa un caso:

```rust
fn main() {
    let nombres = vec!["Ana", "Luis", "Eva"];

    if let Some(primero) = nombres.first() {
        println!("El primero es {primero}");
    }

    let vacia: Vec<&str> = Vec::new();
    if let Some(primero) = vacia.first() {
        println!("El primero es {primero}");
    } else {
        println!("La lista está vacía");
    }
}
```

Salida:

```text
El primero es Ana
La lista está vacía
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Condiciones en mal orden, y una rama nunca se ejecuta | Prueba con un valor de cada rama y con los valores **límite** (aquí 4, 5, 9 y 10) |
| Confundir `=` con `==` | Léelo en voz alta: «es igual a». Con `=` Rust no compila una condición |
| Una condición que no es `bool` (`if n {`) | Escribe la comparación: `if n != 0 {` |
| `if` y `else` que devuelven tipos distintos | Las dos ramas deben dar el mismo tipo |
| Olvidar el brazo `_` en un `match` | El compilador lo avisa: léelo, dice qué valores faltan |
| Comparar un `String` con un `&str` con `==` en sentido contrario | Funciona en los dos sentidos (`texto == "hola"`), pero no mezcles `String` con `char` |
| Comparar decimales con `==` | `0.1 + 0.2 == 0.3` es `false`: compara con una tolerancia (`(a - b).abs() < 1e-9`) |

## Para practicar

Haz los [ejercicios 2.1 de condicionales](condicionales.md). Empieza por los primeros y, cuando funcionen, prueba cada uno con un valor de cada rama y con los límites.
