# A3 · Ejercicios de concurrencia y asincronía

<div class="ej-gate" data-unit="a3" data-nombre="A3 · Concurrencia y asincronía"></div>

Practica tareas simultáneas, resultados y errores, límites de tiempo, reintentos y datos compartidos. Cada ejercicio indica la **salida esperada**. Las esperas son de decenas de milisegundos, así que los resultados son siempre los mismos.

## Ejercicio A3.1

**Tres descargas a la vez.** Simula tres descargas con una pausa cada una: `uno` (200 ms), `dos` (50 ms) y `tres` (100 ms). Cada descarga imprime `termina <nombre>` al acabar.

Lánzalas **todas a la vez**, espera a que terminen y comprueba que el conjunto tardó **menos de 350 ms** (si fueran una tras otra tardarían unos 350 ms o más). Imprime `todas a la vez: sí` o `todas a la vez: no`.

**Salida esperada:**

```text
termina dos
termina tres
termina uno
todas a la vez: sí
```

!!! note "En Rust"
    `thread::spawn` lanza el hilo de inmediato; `.join()` solo bloquea hasta que termine. Llamar a los tres `spawn` antes de los tres `join` es lo que los hace ir a la vez.

<details class="sol" data-key="av/a3/A3.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::thread;
use std::time::{Duration, Instant};
fn main() {
    let inicio = Instant::now();
    let h1 = thread::spawn(|| {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.2

**Sumar en paralelo.** Reparte la suma de los números del `1` al `100` entre **dos tareas** que trabajan a la vez: una suma del `1` al `50` y la otra del `51` al `100`. Espera las dos y muestra la suma total.

**Salida esperada:**

```text
5050
```

<details class="sol" data-key="av/a3/A3.2">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::thread;
fn main() {
    let h1 = thread::spawn(|| (1..=50).sum::&lt;i32&gt;());
    let h2 = thread::spawn(|| (51..=100).sum::&lt;i32&gt;());
    let total = h1.join().unwrap() + h2.join().unwrap();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.3

**Con límite de tiempo.** Escribe una función asíncrona `consulta(ms)` que espere `ms` milisegundos y devuelva `42`. Después escribe `intentar(ms, limiteMs)`, que espera el resultado de la consulta **como máximo** `limiteMs` milisegundos: si llega a tiempo imprime `resultado: 42`, y si no, imprime `tiempo agotado`.

Pruébala con `intentar(300, 100)` y con `intentar(50, 200)`.

**Salida esperada:**

```text
tiempo agotado
resultado: 42
```

!!! note "En Rust"
    `mpsc::Receiver::recv_timeout(Duration)` deja de esperar pasado ese tiempo, aunque el hilo que calcula siga trabajando por detrás.

<details class="sol" data-key="av/a3/A3.3">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::sync::mpsc;
use std::thread;
use std::time::Duration;
fn consulta(ms: u64) -&gt; i32 {
    thread::sleep(Duration::from_millis(ms));
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.4

**Reintentos.** Escribe una operación asíncrona que **falle las dos primeras veces** que se llama y funcione la tercera (devolviendo el texto `listo`). Escribe `conReintentos(maximo)`, que llama a la operación y, si falla, la repite hasta `maximo` veces en total.

En cada intento imprime `intento N falló` o `intento N correcto`; al final, `resultado: listo`. Prueba con un máximo de 3.

**Salida esperada:**

```text
intento 1 falló
intento 2 falló
intento 3 correcto
resultado: listo
```

<details class="sol" data-key="av/a3/A3.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>fn con_reintentos(maximo: i32) -&gt; String {
    let mut llamadas = 0;
    let mut inestable = || {
        llamadas += 1;
        if llamadas &lt; 3 {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.5

**Un contador compartido, bien protegido.** Lanza **5 tareas** que suman 200 veces cada una a un mismo contador, y muestra el valor final (siempre `1000`).

Hazlo de forma que **no se pierda ninguna suma** aunque las tareas se ejecuten a la vez.

**Salida esperada:**

```text
1000
```

!!! note "En Rust"
    Sin `Arc<Mutex<i32>>`, el compilador ni siquiera deja compilar un `contador` mutable compartido entre varios `thread::spawn`.

<details class="sol" data-key="av/a3/A3.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::sync::{Arc, Mutex};
use std::thread;
fn main() {
    let contador = Arc::new(Mutex::new(0));
    let mut manejadores = Vec::new();
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.6

**Productor y consumidor.** Un **productor** envía los números del `1` al `5` y un **consumidor** los va recibiendo y sumando. Se comunican por un **canal** (cola, *stream*...), no por una variable compartida. Cuando el productor termina, avisa de que no hay más.

Muestra solo la suma final.

**Salida esperada:**

```text
suma: 15
```

<details class="sol" data-key="av/a3/A3.6">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>use std::sync::mpsc;
use std::thread;
fn main() {
    let (tx, rx) = mpsc::channel();
    let productor = thread::spawn(move || {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
