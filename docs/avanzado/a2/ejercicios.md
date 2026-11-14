# A2 · Ejercicios de genéricos

<div class="ej-gate" data-unit="a2" data-nombre="A2 · Genéricos"></div>

Practica funciones y clases genéricas, restricciones y una colección propia. Cada ejercicio indica la **salida esperada**.

## Ejercicio A2.1

**El último, si lo hay.** Escribe una función genérica `ultimo` que reciba una lista de cualquier tipo y devuelva su **último elemento**, o **«nada»** si la lista está vacía (usa el mecanismo del lenguaje para un valor que puede faltar).

Pruébala con `[1, 2, 3]`, con `[a, b]` y con una lista de enteros **vacía**. Cuando falte el valor, imprime la palabra `nada`.

**Salida esperada:**

```text
ultimo de [1, 2, 3]: 3
ultimo de [a, b]: b
ultimo de []: nada
```

!!! note "En Rust"
    `Option<T>` es el «valor que puede faltar» de Rust: `Some(x)` o `None`. `.unwrap_or_else(...)` da un valor alternativo cuando es `None`.

<details class="sol" data-key="av/a2/A2.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>fn ultimo&lt;T: Clone&gt;(lista: &amp;[T]) -&gt; Option&lt;T&gt; {
    lista.last().cloned()
}
fn mostrar&lt;T: std::fmt::Display&gt;(lista: &amp;[T]) -&gt; String {
    let partes: Vec&lt;String&gt; = lista.iter().map(|x| x.to_string()).collect();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A2.2

**Un par que se da la vuelta.** Crea una clase genérica `Par<A, B>` con dos valores, `primero` y `segundo`, y un método `intercambiar()` que devuelva **otro par** con los valores cambiados de sitio (un `Par<B, A>`). Haz que se escriba como `(primero, segundo)`.

Crea el par `(1, uno)` y muestra el original y el intercambiado.

**Salida esperada:**

```text
(1, uno)
(uno, 1)
```

<details class="sol" data-key="av/a2/A2.2">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>struct Par&lt;A, B&gt; {
    primero: A,
    segundo: B,
}
impl&lt;A, B&gt; Par&lt;A, B&gt; {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A2.3

**El mínimo de cualquier cosa comparable.** Escribe una función genérica `minimo` que devuelva el **menor** elemento de una lista, aceptando solo tipos que se puedan comparar entre sí.

Pruébala con `[4, 1, 7]` y con `[pera, casa, uva]`.

**Salida esperada:**

```text
1
casa
```

!!! note "En Rust"
    Restringe con `T: PartialOrd` para poder comparar con `<`, y con `T: Copy` para poder leer `lista[0]` sin moverlo fuera del slice.

<details class="sol" data-key="av/a2/A2.3">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>fn minimo&lt;T: PartialOrd + Copy&gt;(lista: &amp;[T]) -&gt; T {
    let mut m = lista[0];
    for &amp;x in &amp;lista[1..] {
        if x &lt; m {
            m = x;
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A2.4

**Invertir con una pila.** Escribe tu propia clase genérica `Pila<T>` (apilar, desapilar y saber si está vacía) y úsala para escribir `invertir`, una función genérica que devuelva la lista **al revés**: apila todos los elementos y desapílalos uno a uno.

Muestra `[1, 2, 3]` y `[a, b, c]` invertidas, cada una en su línea, con los elementos separados por coma y espacio.

**Salida esperada:**

```text
3, 2, 1
c, b, a
```

<details class="sol" data-key="av/a2/A2.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>struct Pila&lt;T&gt; {
    elementos: Vec&lt;T&gt;,
}
impl&lt;T&gt; Pila&lt;T&gt; {
    fn nueva() -&gt; Self {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A2.5

**Contar con un criterio.** Escribe una función genérica `contarSi` que reciba una lista y una función que decide si un elemento cumple una condición, y devuelva **cuántos** la cumplen.

Úsala para contar los números **pares** de `1` a `10`, y las palabras de **más de 3 letras** en `[sol, luna, mar, estrella]`.

**Salida esperada:**

```text
5
2
```

<details class="sol" data-key="av/a2/A2.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>fn contar_si&lt;T&gt;(lista: &amp;[T], cumple: impl Fn(&amp;T) -&gt; bool) -&gt; usize {
    lista.iter().filter(|x| cumple(x)).count()
}
fn main() {
    let numeros: Vec&lt;i32&gt; = (1..=10).collect();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A2.6

**Una caché genérica.** Crea una clase genérica `Cache<K, V>` con un método `obtener(clave, calcular)`: si ya conoce el resultado de esa clave, lo devuelve **sin calcularlo otra vez**; si no, ejecuta la función `calcular`, lo guarda y lo devuelve.

Pruébala con una función que imprime `calculando N` y devuelve `N` al cuadrado: pide el `4`, otra vez el `4`, y luego el `5`. Comprueba que `calculando 4` aparece **una sola vez**.

**Salida esperada:**

```text
calculando 4
16
16
calculando 5
25
```

!!! note "En Rust"
    `HashMap::get` seguido de `insert` evita calcular dos veces; con una sola llamada a `.entry(clave).or_insert_with(...)` también se puede, pero complica pasar la clave a `calcular`.

<details class="sol" data-key="av/a2/A2.6">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::collections::HashMap;
struct Cache&lt;K, V&gt; {
    guardado: HashMap&lt;K, V&gt;,
}
impl&lt;K: std::hash::Hash + Eq + Clone, V: Clone&gt; Cache&lt;K, V&gt; {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
