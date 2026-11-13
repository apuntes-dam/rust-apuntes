# Rust

Apuntes personales para aprender **Rust**, organizados con la misma estructura que la unidad 1 de Programación: del concepto de programa hasta compilar, ejecutar y comprobar código.

!!! info "Cómo está organizado"
    Cada apartado responde a una pregunta: *qué es un programa*, *cómo se escribe en Rust*, *qué datos maneja*, *cómo se comprueba* y *cómo se compila y ejecuta*.

| Apartado | Contenido |
|---|---|
| [U1. Primer programa en Rust](u01/index.md) | Estructura, variables, operadores, tipos, pruebas |
| [U2. Estructuras de control](u02/index.md) | Condicionales (`if`, `match`), errores (`Result`, `Option`, `?`) y depuración |
| [U3. Estructuras de datos](u03/index.md) | Cadenas, vectores, mapas, conjuntos, JSON y XML |
| [U4. Structs y métodos](u04/index.md) | `struct`, `impl`, encapsulamiento, constructores y enumerados |
| [U5. Traits y polimorfismo](u05/index.md) | Traits (sin herencia), trait objects y jerarquías cerradas con `enum` |
| [U6. Diseño con traits y SOLID](u06/index.md) | Módulos, los cinco principios SOLID y documentación con `///` |
| [U7. Entrada/salida y GUI](u07/index.md) | Consola, archivos, ficheros de texto e interfaces gráficas con `egui` |
| [U9. Bases de datos relacionales](u09/index.md) | `rusqlite`: conexión, consultas, transacciones y el patrón DAO |
| [Entorno](u01/05-entorno.md) | `rustup`, `cargo` y el compilador `rustc` |
| [Prácticas](u01/practicas.md) | Guía de prácticas de la unidad 1 |

!!! note "Qué es Rust"
    Rust es un lenguaje **compilado, de tipado estático y sin recolector de basura**: el compilador comprueba quién es dueño de cada dato (*ownership*) y así evita los errores de memoria sin perder velocidad. Se usa en sistemas, herramientas de línea de comandos, servidores, WebAssembly y, cada vez más, en partes de Windows, Linux y Android. Lo mantiene la Rust Foundation y se publica una versión nueva cada seis semanas.

!!! warning "Web en construcción"
    Por ahora están las **unidades 1 a 7 y la 9** (el material original no tiene unidad 8), y falta la sección Avanzado.
