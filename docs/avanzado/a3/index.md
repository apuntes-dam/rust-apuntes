# A3 · Concurrencia y asincronía

Hacer varias cosas **a la vez** sin que el programa se bloquee: **hilos, corrutinas e isolates** según el lenguaje, recoger **resultados y errores**, poner **límites de tiempo**, y evitar las **condiciones de carrera** con candados o con mensajes.

## Teoría

* [A3.A Tareas a la vez](t-tareas.md): `std::thread`, `spawn` y esperar a varios hilos.
* [A3.B Resultados, errores y esperas](t-resultados.md): resultados y errores de un hilo, límites de tiempo y reintentos.
* [A3.C Compartir datos con seguridad](t-compartir.md): condición de carrera, `Arc<Mutex<T>>` y canales.

## Antes de empezar: qué debes dominar

* Funciones como valores (unidad A1 de este apartado).
* Excepciones (unidad 2).
* Clases y objetos (unidad 4).

## Ejercicios de la unidad

| Bloque | Ejercicios |
|---|---|
| [A3 · Ejercicios de concurrencia y asincronía](ejercicios.md) | 6 |

## Antes de pasar a los ejercicios

Cuando hayas leído y practicado la teoría, marca la casilla para **desbloquear** los ejercicios de esta unidad:

<div class="ej-check" data-unit="a3"></div>
