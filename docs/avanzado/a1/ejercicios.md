# A1 · Ejercicios de programación funcional

<div class="ej-gate" data-unit="a1" data-nombre="A1 · Programación funcional"></div>

Practica funciones como valores, colecciones sin bucles y secuencias perezosas. Cada ejercicio indica la **salida esperada** para que compruebes tu programa.

## Ejercicio A1.1

**Aplicar una función varias veces.** Escribe `aplicarNVeces(f, n, x)`, que aplica la función `f` al valor `x`, `n` veces seguidas (con `n = 0` devuelve `x` sin cambios).

En `main` pruébala con estas llamadas, cada resultado en su línea:

* «doble» (multiplicar por 2) aplicada 3 veces a `1`.
* «sumar 3» aplicada 4 veces a `0`.
* «doble» aplicada 0 veces a `1`.

**Salida esperada:**

```text
8
12
1
```

<details class="sol" data-key="av/a1/A1.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>fn aplicar_n_veces(f: impl Fn(i32) -&gt; i32, n: i32, x: i32) -&gt; i32 {
    let mut valor = x;
    for _ in 0..n {
        valor = f(valor);
    }
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.2

**Contador con paso.** Escribe `crearContador(inicio, paso)`, que devuelve una función. Cada vez que se llama a esa función devuelve el valor actual y después avanza `paso` unidades.

En `main` crea un contador que empieza en `10` y avanza de `5` en `5`, y otro que empieza en `0` y avanza de `1` en `1`. Llama tres veces al primero (en una sola línea, separando con espacios) y una vez al segundo. Comprueba que **cada contador lleva su propia cuenta**.

**Salida esperada:**

```text
10 15 20
0
```

!!! note "En Rust"
    Para que la closure «recuerde» y modifique `actual` entre llamadas necesita el trait `FnMut`, y `move` para que sea dueña de la variable (si no, no podría sobrevivir fuera de `crear_contador`).

<details class="sol" data-key="av/a1/A1.2">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>fn crear_contador(inicio: i32, paso: i32) -&gt; impl FnMut() -&gt; i32 {
    let mut actual = inicio;
    move || {
        let valor = actual;
        actual += paso;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.3

**Palabras seleccionadas.** Dada la lista `sol, luna, estrella, mar, cielo, nube`:

1. Quédate con las palabras de **4 letras o más**.
2. Pásalas a **mayúsculas**.
3. **Ordénalas** alfabéticamente.
4. Muestra el resultado unido con guiones y, en otra línea, cuántas palabras quedaron.

Hazlo **sin escribir un bucle**: encadena operaciones de colección.

**Salida esperada:**

```text
CIELO-ESTRELLA-LUNA-NUBE
4 palabras
```

<details class="sol" data-key="av/a1/A1.3">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>fn main() {
    let palabras = ["sol", "luna", "estrella", "mar", "cielo", "nube"];
    let mut resultado: Vec&lt;String&gt; = palabras.iter().filter(|p| p.len() &gt;= 4).map(|p| p.to_uppercase()).collect();
    resultado.sort();
    println!("{}", resultado.join("-"));
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.4

**Contar por longitud.** Con la misma lista del ejercicio anterior (`sol, luna, estrella, mar, cielo, nube`), cuenta **cuántas palabras hay de cada longitud** y muestra el resultado ordenado por longitud, con el formato `longitud:cantidad` separado por comas y espacios.

**Salida esperada:**

```text
3:2, 4:2, 5:1, 8:1
```

<details class="sol" data-key="av/a1/A1.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::collections::BTreeMap;
fn main() {
    let palabras = ["sol", "luna", "estrella", "mar", "cielo", "nube"];
    let mut por_longitud: BTreeMap&lt;usize, i32&gt; = BTreeMap::new();
    for p in palabras {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.5

**Una tubería de funciones.** Escribe `pipeline(funciones)`, que recibe una lista de funciones de entero a entero y devuelve **una sola función** que las aplica una tras otra, en orden.

En `main`: construye la tubería «multiplicar por 2», «sumar 1», «elevar al cuadrado» y aplícala a `5`. Después comprueba qué ocurre con una tubería **vacía** aplicada a `7`.

**Salida esperada:**

```text
121
7
```

<details class="sol" data-key="av/a1/A1.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>fn pipeline(funciones: Vec&lt;Box&lt;dyn Fn(i32) -&gt; i32&gt;&gt;) -&gt; impl Fn(i32) -&gt; i32 {
    move |x| funciones.iter().fold(x, |acum, f| f(acum))
}
fn main() {
    let tuberia: Vec&lt;Box&lt;dyn Fn(i32) -&gt; i32&gt;&gt; =
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A1.6

**Secuencia perezosa.** Obtén los **cuatro primeros** números naturales (empezando en 1) que sean múltiplos de 7 y mayores que 20, y muéstralos como una lista.

La secuencia de números naturales es **infinita**: usa una secuencia perezosa y pide solo los cuatro que necesitas.

**Salida esperada:**

```text
[21, 28, 35, 42]
```

!!! note "En Rust"
    `(1..)` es un rango sin límite superior: es la secuencia infinita de Rust. `.take(4)` corta justo lo que necesitas.

<details class="sol" data-key="av/a1/A1.6">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>fn main() {
    let resultado: Vec&lt;i32&gt; = (1..).filter(|n| n % 7 == 0 &amp;&amp; *n &gt; 20).take(4).collect();
    println!("{:?}", resultado);
}
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
