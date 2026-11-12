# Rust

Apuntes personales para aprender **Rust**, organizados con la misma estructura que la unidad 1 de Programación: del concepto de programa hasta compilar, ejecutar y comprobar código.

!!! info "Cómo está organizado"
    Cada apartado responde a una pregunta: *qué es un programa*, *cómo se escribe en Rust*, *qué datos maneja*, *cómo se comprueba* y *cómo se compila y ejecuta*.

| Apartado | Contenido |
|---|---|
| [U1. Primer programa en Rust](u01/index.md) | Estructura, variables, operadores, tipos, pruebas |
| [Entorno](u01/05-entorno.md) | `rustup`, `cargo` y el compilador `rustc` |
| [Prácticas](u01/practicas.md) | Guía de prácticas de la unidad 1 |

!!! note "Qué es Rust"
    Rust es un lenguaje **compilado, de tipado estático y sin recolector de basura**: el compilador comprueba quién es dueño de cada dato (*ownership*) y así evita los errores de memoria sin perder velocidad. Se usa en sistemas, herramientas de línea de comandos, servidores, WebAssembly y, cada vez más, en partes de Windows, Linux y Android. Lo mantiene la Rust Foundation y se publica una versión nueva cada seis semanas.

!!! warning "Web en construcción"
    Por ahora está la **unidad 1**. Las demás (control, errores, colecciones, `struct` y `trait`, ficheros y bases de datos) se irán añadiendo.
