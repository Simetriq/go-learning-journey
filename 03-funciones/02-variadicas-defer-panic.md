# 02 - Funciones Variádicas, Defer, Panic y Recover

> **Objetivo:** Dominar funciones que aceptan número variable de argumentos, y aprender a manejar situaciones excepcionales con defer, panic y recover.

---

## Tabla de Contenidos

1. [Funciones variádicas](#funciones-variádicas)
2. [defer: aplazar ejecución](#defer-aplazar-ejecución)
3. [Orden de ejecución de defer](#orden-de-ejecución-de-defer)
4. [defer en limpieza de recursos](#defer-en-limpieza-de-recursos)
5. [panic: errores irrecuperables](#panic-errores-irrecuperables)
6. [recover: capturar un panic](#recover-capturar-un-panic)
7. [Patrón: defer + recover](#patrón-defer--recover)
8. [Ejercicios propuestos](#ejercicios-propuestos)

---

## Funciones variádicas

Una función **variádica** acepta un número variable de argumentos del mismo tipo usando `...tipo`. Internamente, Go los trata como un slice.

```go
package main

import "fmt"

// El parámetro variádico SIEMPRE es el último
func sumar(numeros ...int) int {
    total := 0
    for _, n := range numeros {
        total += n
    }
    return total
}

func main() {
    fmt.Println(sumar())          // 0
    fmt.Println(sumar(1))         // 1
    fmt.Println(sumar(1, 2, 3))   // 6
    fmt.Println(sumar(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)) // 55

    // Expandir un slice existente con ...
    numeros := []int{10, 20, 30}
    fmt.Println(sumar(numeros...)) // 60
}
```

### Variádica con parámetros fijos

```go
package main

import (
    "fmt"
    "strings"
)

// Los parámetros fijos van ANTES del variádico
func formatear(separador string, partes ...string) string {
    return strings.Join(partes, separador)
}

// Logger con nivel y mensajes múltiples
func log(nivel string, mensajes ...interface{}) {
    fmt.Printf("[%s] ", strings.ToUpper(nivel))
    fmt.Println(mensajes...)
}

func main() {
    fmt.Println(formatear("-", "Go", "es", "genial"))   // Go-es-genial
    fmt.Println(formatear(", ", "rojo", "verde", "azul")) // rojo, verde, azul
    fmt.Println(formatear("/", "usr", "local", "bin"))    // usr/local/bin

    log("info", "Servidor iniciado en puerto", 8080)
    log("error", "Conexión fallida:", "timeout")
    log("debug", "Variables:", 42, true, "texto")
}
```

### Variádica + tipo interface{}

```go
package main

import "fmt"

// Similar al fmt.Println de la biblioteca estándar
func imprimir(args ...interface{}) {
    for i, arg := range args {
        if i > 0 {
            fmt.Print(" ")
        }
        fmt.Print(arg)
    }
    fmt.Println()
}

// Máximo de cualquier cantidad de números
func max(primero int, resto ...int) int {
    maximo := primero
    for _, n := range resto {
        if n > maximo {
            maximo = n
        }
    }
    return maximo
}

func main() {
    imprimir("Go", 2024, true, 3.14)

    fmt.Println(max(5))             // 5
    fmt.Println(max(3, 7, 2, 9, 1)) // 9
}
```

---

## defer: aplazar ejecución

`defer` pospone la ejecución de una función hasta que la función que la contiene **retorne** (ya sea normalmente o por pánico).

```go
package main

import "fmt"

func ejemploBasico() {
    fmt.Println("inicio")
    defer fmt.Println("defer 1 — se ejecuta al final")
    fmt.Println("medio")
    defer fmt.Println("defer 2 — también al final")
    fmt.Println("fin")
}

func main() {
    ejemploBasico()
    // Salida:
    // inicio
    // medio
    // fin
    // defer 2 — también al final  ← LIFO: último en entrar, primero en salir
    // defer 1 — se ejecuta al final
}
```

> 💡 **Regla clave:** Los defers se ejecutan en orden **LIFO** (Last In, First Out), como una pila.

---

## Orden de ejecución de defer

```go
package main

import "fmt"

func contarAtras() {
    for i := 1; i <= 5; i++ {
        defer fmt.Println(i) // captura el valor de i en ese momento
    }
}

func main() {
    fmt.Println("Contando al revés:")
    contarAtras()
    // 5
    // 4
    // 3
    // 2
    // 1
}
```

### Los argumentos se evalúan inmediatamente

```go
package main

import "fmt"

func main() {
    x := 10
    // El valor de x (10) se captura AHORA, no cuando se ejecute el defer
    defer fmt.Println("valor capturado:", x)

    x = 99
    fmt.Println("x actual:", x)

    // Salida:
    // x actual: 99
    // valor capturado: 10  ← se capturó el 10, no el 99
}
```

---

## defer en limpieza de recursos

Este es el **uso más importante** de `defer`: garantizar que los recursos se liberen correctamente.

```go
package main

import (
    "fmt"
    "os"
)

func leerArchivo(nombre string) error {
    archivo, err := os.Open(nombre)
    if err != nil {
        return fmt.Errorf("no se pudo abrir %s: %w", nombre, err)
    }
    defer archivo.Close() // ← garantizado: se cerrará al terminar la función

    // Leer contenido...
    buf := make([]byte, 100)
    n, err := archivo.Read(buf)
    if err != nil {
        return err // archivo.Close() se ejecuta aquí también
    }

    fmt.Printf("Leídos %d bytes: %s\n", n, buf[:n])
    return nil
}

func main() {
    if err := leerArchivo("go.mod"); err != nil {
        fmt.Println("Error:", err)
    }
}
```

### Simular conexión a base de datos

```go
package main

import "fmt"

type Conexion struct{ nombre string }

func (c *Conexion) Cerrar() {
    fmt.Printf("🔌 Conexión '%s' cerrada\n", c.nombre)
}

func nuevaConexion(nombre string) *Conexion {
    fmt.Printf("✅ Conexión '%s' abierta\n", nombre)
    return &Conexion{nombre: nombre}
}

func procesarDatos() {
    conn := nuevaConexion("postgres-produccion")
    defer conn.Cerrar() // siempre se cerrará

    fmt.Println("Procesando datos...")
    // Si hubiera un error aquí, Cerrar() igualmente se ejecuta
    fmt.Println("Datos procesados correctamente")
}

func main() {
    procesarDatos()
}
```

---

## panic: errores irrecuperables

`panic` detiene la ejecución normal y desenrolla la pila de llamadas. Se usa para errores que **no deberían ocurrir** (bugs de programación, no errores esperados).

```go
package main

import "fmt"

func obtenerElemento(s []int, i int) int {
    if i < 0 || i >= len(s) {
        // panic para condiciones que indican un bug
        panic(fmt.Sprintf("índice %d fuera de rango [0, %d)", i, len(s)))
    }
    return s[i]
}

func main() {
    s := []int{10, 20, 30}
    fmt.Println(obtenerElemento(s, 1)) // 20

    // Esto causará un panic
    // fmt.Println(obtenerElemento(s, 5))
}
```

> ⚠️ **Cuándo usar panic:** Solo para condiciones que representan errores de programación (precondiciones violadas, estados imposibles). Para errores esperados (archivo no encontrado, entrada inválida), usa `error`.

---

## recover: capturar un panic

`recover` solo funciona dentro de una función llamada con `defer`. Detiene el panic y retorna el valor pasado a `panic`.

```go
package main

import "fmt"

func operacionPeligrosa() {
    panic("algo salió muy mal")
}

func ejecutarSeguros() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("🛡️  Panic capturado:", r)
            fmt.Println("El programa continúa...")
        }
    }()

    operacionPeligrosa()
    fmt.Println("Esta línea NUNCA se ejecuta")
}

func main() {
    fmt.Println("Antes")
    ejecutarSeguros()
    fmt.Println("Después — el programa no terminó")
}
```

---

## Patrón: defer + recover

El patrón más común es convertir panics en errores para no propagar el caos:

```go
package main

import "fmt"

// safeDiv convierte un posible panic (división por cero) en un error
func safeDiv(a, b int) (resultado int, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic recuperado: %v", r)
        }
    }()

    resultado = a / b // Go hace panic si b == 0
    return resultado, nil
}

// ejecutar envuelve cualquier función y captura panics
func ejecutar(nombre string, fn func()) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic en '%s': %v", nombre, r)
        }
    }()
    fn()
    return nil
}

func main() {
    // safeDiv
    if r, err := safeDiv(10, 2); err == nil {
        fmt.Println("10/2 =", r)
    }

    if _, err := safeDiv(10, 0); err != nil {
        fmt.Println("Error capturado:", err)
    }

    // ejecutar con closure
    if err := ejecutar("tarea segura", func() {
        fmt.Println("Ejecutando tarea normal...")
    }); err != nil {
        fmt.Println("Error:", err)
    }

    if err := ejecutar("tarea peligrosa", func() {
        var s []int
        _ = s[5] // panic: index out of range
    }); err != nil {
        fmt.Println("Error capturado:", err)
    }

    fmt.Println("Programa terminado correctamente")
}
```

---

## Ejercicios propuestos

### Ejercicio 1 — Printf personalizado
Escribe una función `logf(formato string, args ...interface{})` que imprima mensajes con timestamp y formato `[HH:MM:SS] mensaje`. Usa `time.Now()` para el tiempo.

### Ejercicio 2 — defer con limpieza múltiple
Simula abrir 3 "recursos" (A, B, C) con defer para cada uno. Verifica que el orden de cierre es C → B → A usando `fmt.Println`.

### Ejercicio 3 — Wrapper de recuperación
Escribe una función `mustRun(fn func() error) error` que:
- Ejecute `fn`
- Capture cualquier panic con recover y lo convierta en error
- Retorne el error de `fn` si lo hay

### Ejercicio 4 — Stack segura
Implementa una `PilaSegura` (struct con slice y mutex simulado) con métodos `Push` y `Pop` donde `Pop` en una pila vacía cause `panic`, y escribe una función `popSeguro(p *PilaSegura) (valor int, ok bool)` que use recover para retornar `(0, false)` en vez de propagar el panic.

<details>
<summary>💡 Pista para Ejercicio 3</summary>

Recuerda: `recover()` solo funciona si está dentro de una función deferida. Declara la variable `err` antes del defer para que el closure la capture.

</details>
