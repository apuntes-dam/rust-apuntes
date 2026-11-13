# U5 · Herencia, clases abstractas e interfaces

Reutilizar y organizar tipos sin herencia: **traits** (contratos compartidos), **polimorfismo** con trait objects y **jerarquías cerradas** con `enum`.

## Teoría

* [5.A Traits](t-traits.md): contratos compartidos, en vez de herencia.
* [5.B Polimorfismo](t-polimorfismo.md): trait objects (`Box<dyn Trait>`) frente a genéricos.
* [5.C Comportamiento por defecto](t-default.md): el sustituto de las clases abstractas.
* [5.D Jerarquías cerradas](t-cerradas.md): `enum` con datos y `match` exhaustivo.

## Antes de empezar: qué debes dominar

* Composición (un struct que contiene a otro), en vez de herencia.
* Trait objects (`Box<dyn Trait>`) y comportamiento por defecto sobrescribible.
* Traits con y sin implementación por defecto.
* Jerarquías cerradas con `enum` y datos con `#[derive(...)]`.

## Ejercicios de la unidad

| Bloque | Ejercicios |
|---|---|
| [U5.1 · Clases abstractas, interfaces y herencia](herencia.md) | 10 |

## Antes de pasar a los ejercicios

Cuando hayas leído y practicado la teoría, marca la casilla para **desbloquear** los ejercicios de esta unidad:

<div class="ej-check" data-unit="u05"></div>
