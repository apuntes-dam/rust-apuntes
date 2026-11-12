# P1.2 · Primeros programas

<div class="ej-gate" data-unit="u01" data-nombre="U1 · Primer programa"></div>

Traduce algoritmos sencillos a código. Para cada ejercicio anota **entradas, proceso y salidas** antes de programar. Prueba con valores normales, cero y negativos cuando tenga sentido.

## Ejercicio 1.2.1

Pide el nombre del usuario y salúdalo.

```text
Escribe tu nombre: Juan
Hola, Juan.
```

<details class="sol" data-key="p1-2/1.2.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::io::{self, Write};
/// Muestra un mensaje y devuelve la línea escrita por el usuario, sin el salto final.
fn leer(mensaje: &amp;str) -&gt; String {
    print!("{mensaje}");
    io::stdout().flush().unwrap();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 1.2.2

Pide las horas trabajadas y el precio por hora y muestra el importe total del servicio.

```text
Horas de trabajo: 6
Coste por hora: 10
Importe total: 60
```

## Ejercicio 1.2.3

Suponiendo estas declaraciones:

```rust
let ancho = 17;      // i32
let alto = 12.0;     // f64
```

Antes de ejecutarlas, adivina el **valor** y el **tipo** de cada expresión y luego compruébalo con esta función, que devuelve el nombre del tipo de lo que le pases: `fn tipo<T>(_: &T) -> &'static str { std::any::type_name::<T>() }`.

1. `ancho / 2`
2. `ancho as f64 / 2.0`
3. `alto / 3.0`
4. `1 + 2 * 5`

??? success "Respuestas"
    1. `8` → `i32` (entero entre entero es división entera)
    2. `8.5` → `f64` (hay que convertir `ancho` con `as f64`: `ancho / 2.0` no compila, porque Rust no mezcla tipos)
    3. `4` → `f64` (se muestra `4`, pero es un decimal)
    4. `11` → `i32` (primero la multiplicación)

## Ejercicio 1.2.4

Pide una temperatura en grados Celsius, conviértela a Fahrenheit y muéstrala.

<details class="sol" data-key="p1-2/1.2.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::io::{self, Write};
/// Muestra un mensaje y devuelve la línea escrita por el usuario, sin el salto final.
fn leer(mensaje: &amp;str) -&gt; String {
    print!("{mensaje}");
    io::stdout().flush().unwrap();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 1.2.5

Pide el importe de un artículo sin IVA y el tipo de IVA (por ejemplo, 21) y muestra el precio final.

## Ejercicio 1.2.6

Pide el precio final de un artículo y, suponiendo un IVA del 10 %, muestra el IVA pagado y el importe sin IVA.

## Ejercicio 1.2.7

Pide tres números y muestra su suma.

!!! note "En Rust"
    Si el usuario escribe los tres números en una sola línea (`4 5 6`), puedes leerla entera y separarla con `split_whitespace()`, convirtiendo cada trozo con `parse()`.

## Ejercicio 1.2.8

Pide una cantidad de segundos y muéstrala en horas, minutos y segundos. Por ejemplo, `3725` segundos son `1 h 2 min 5 s`.

## Ejercicio 1.2.9

Guarda dos valores en las variables `a` y `b`, pedidos por teclado. Intercambia sus valores (que `a` acabe con lo que tenía `b` y al revés) y muestra ambas variables antes y después del intercambio.

## Ejercicio 1.2.10

Muestra por pantalla el resultado de esta operación: `3 + 5 · (2 + 4)² ÷ 6`. Respeta la precedencia de los operadores y usa la potencia de tu lenguaje.

*(El enunciado original es una imagen; aquí se sustituye por una operación equivalente.)*

!!! note "En Rust"
    La potencia de un `f64` es `x.powi(2)` (exponente entero) o `x.powf(2.5)`; no existe el operador `**`. Escribe los números como decimales (`6.0`) para que la división no sea entera.

## Ejercicio 1.2.11

Lee un entero positivo `n` y muestra la suma de los enteros de 1 a `n`. Hazlo de dos formas: con un bucle y con la fórmula `n · (n + 1) / 2`.

## Ejercicio 1.2.12

Pide peso (kg) y estatura (m), calcula el índice de masa corporal (peso / estatura²) y muestra `Tu índice de masa corporal es X`, con X redondeado a 2 decimales.

!!! note "En Rust"
    `parse::<f64>()` solo acepta el punto decimal: el usuario debe escribir `1.75`. Con `1,75` el `unwrap()` para el programa. Si quieres admitir la coma, sustitúyela antes: `texto.replace(',', ".").parse::<f64>()`.

<details class="sol" data-key="p1-2/1.2.12">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::io::{self, Write};
/// Muestra un mensaje y devuelve la línea escrita por el usuario, sin el salto final.
fn leer(mensaje: &amp;str) -&gt; String {
    print!("{mensaje}");
    io::stdout().flush().unwrap();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 1.2.13

Pide dos enteros `n` y `m` y muestra `la división de n entre m da un cociente c y un resto r`. Controla también la división entre cero.

<details class="sol" data-key="p1-2/1.2.13">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::io::{self, Write};
/// Muestra un mensaje y devuelve la línea escrita por el usuario, sin el salto final.
fn leer(mensaje: &amp;str) -&gt; String {
    print!("{mensaje}");
    io::stdout().flush().unwrap();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 1.2.14

Una juguetería envía payasos (112 g cada uno) y muñecas (75 g cada una). Lee cuántos de cada se han vendido y calcula el peso total del paquete.

## Ejercicio 1.2.15

Una cuenta de ahorros da un 4 % de interés anual que se suma al saldo a final de año. Lee el dinero depositado y muestra el saldo tras el primer, segundo y tercer año, redondeado a 2 decimales. Fórmula de cada año: `capital · (1 + interés)`.

!!! note "En Rust"
    `parse::<f64>()` solo acepta el punto decimal: el usuario debe escribir `1.75`. Con `1,75` el `unwrap()` para el programa. Si quieres admitir la coma, sustitúyela antes: `texto.replace(',', ".").parse::<f64>()`.

## Ejercicio 1.2.16

Una panadería vende barras a 3,49 € (defínelo como constante). Las que no son del día tienen un 60 % de descuento. Lee cuántas barras no frescas se venden y muestra el precio habitual, el descuento aplicado y el coste total.

## Ejercicio 1.2.17

Pide un nombre y un entero `n` y muestra el nombre `n` veces, cada una en una línea.

## Ejercicio 1.2.18

Pide el nombre completo y muéstralo tres veces: todo en minúsculas, todo en mayúsculas y con la inicial de cada palabra en mayúscula. El usuario puede escribirlo con cualquier combinación de mayúsculas y minúsculas.

!!! note "En Rust"
    Separa las palabras con `split_whitespace()` y, en cada una, pon en mayúscula el primer carácter con `chars()`: `let mut c = p.chars(); c.next().unwrap().to_uppercase().collect::<String>() + &c.as_str().to_lowercase()`.

## Ejercicio 1.2.19

Pide un nombre y muestra `NOMBRE tiene n letras.`, con el nombre en mayúsculas.

## Ejercicio 1.2.20

Los teléfonos de una empresa tienen el formato `prefijo-número-extensión`, por ejemplo `+34-913724710-56`. Pide uno y muestra solo el número, sin prefijo ni extensión.

## Ejercicio 1.2.21

Pide una frase y muéstrala invertida.

## Ejercicio 1.2.22

Pide una frase y una vocal, y muestra la frase con esa vocal en mayúscula.

## Ejercicio 1.2.23

Pide un correo electrónico y muestra otro con el mismo nombre de usuario (lo que va antes de la `@`) pero con el dominio `ceu.es`.

## Ejercicio 1.2.24

Pide el precio de un producto en euros con dos decimales y muestra cuántos euros y cuántos céntimos son.

!!! note "En Rust"
    No obtengas los céntimos multiplicando un `f64` por 100: `19.99 * 100.0` no da exactamente `1999`, y al truncar con `as i32` salen 1998. Separa el texto por el punto (`split('.')`) o redondea con `.round()` antes de convertir.

## Ejercicio 1.2.25

Pide una fecha de nacimiento con formato `dd/mm/aaaa` y muestra día, mes y año. Después adáptalo para que funcione si el día o el mes se escriben con un solo dígito.

## Ejercicio 1.2.26

Pide los productos de una cesta de la compra separados por comas y muestra cada producto en una línea.

## Ejercicio 1.2.27

Pide el nombre de un producto, su precio y las unidades, y muestra una línea con el nombre, el precio unitario (6 dígitos enteros y 2 decimales), las unidades (3 dígitos) y el coste total (8 dígitos enteros y 2 decimales).

!!! note "En Rust"
    En `{precio:>9.2}`, `>9` reserva 9 posiciones alineadas a la derecha y `.2` fija dos decimales; para los enteros, `{unidades:>3}`. El ancho y la precisión se escriben dentro de las llaves.

## Ejercicio 1.2.28

Calcula el área de un triángulo a partir de sus tres lados (fórmula de Herón). Indica qué ocurre si las longitudes no pueden formar un triángulo.

!!! note "En Rust"
    `(-4.0f64).sqrt()` no para el programa: devuelve `NaN` (se comprueba con `.is_nan()`). Comprueba antes que cada lado es menor que la suma de los otros dos.

## Ejercicio 1.2.29

Genera un número aleatorio entre dos valores que introduce el usuario (ambos incluidos).

!!! note "En Rust"
    La librería estándar no trae aleatorios: añade el crate `rand` (`cargo add rand`) y usa `rand::random_range(a..=b)`, que incluye los dos extremos.

## Ejercicio 1.2.30

Determina si un número es primo. Ten en cuenta el 0, el 1 y los negativos.

## Ejercicio 1.2.31

Muestra todos los divisores de un número.

## Ejercicio 1.2.32

Muestra la serie de Fibonacci hasta un número dado.

## Herramientas útiles en Rust

| Necesitas... | Usa... |
|---|---|
| Leer una línea | `io::stdin().read_line(&mut texto)` y luego `texto.trim()` |
| Texto a número | `texto.trim().parse::<i32>()`, `parse::<f64>()` (devuelven `Result`: `unwrap()` o `match`) |
| Redondear al mostrar | `format!("{x:.2}")` o `println!("{x:.2}")` |
| Potencia y raíz | `x.powi(2)`, `x.powf(0.5)`, `x.sqrt()` (sobre `f64`) |
| Aleatorios | el crate `rand` (`cargo add rand`): `rand::random_range(a..=b)` |
| Mayúsculas / minúsculas | `to_uppercase()`, `to_lowercase()` |
| Cortar y unir texto | `split(',')`, `&texto[a..b]`, `lista.join(", ")` |
| Rellenar con ceros | `format!("{:03}", 7)` → `007` |
| Parar con un error | `panic!("mensaje")` |

!!! note "Decimales siempre con punto"
    `parse::<f64>()` solo entiende `1.75`; con `1,75` falla. Al mostrar, Rust también usa siempre el punto, sea cual sea el idioma del equipo. Si el usuario escribe con coma, cámbiala antes: `texto.replace(',', ".")`.

Para ejecutar cada ejercicio, guárdalo en un archivo (por ejemplo `ejercicio.rs`), compílalo con `rustc ejercicio.rs` y lanza el ejecutable, o crea un proyecto con `cargo new` y usa `cargo run`.
