# 01 - Goroutines y Canales en Go

> **Objetivo:** Entender la concurrencia en Go a través de goroutines (unidades de ejecución ligeras) y canales (mecanismo de comunicación entre goroutines). La filosofía de Go: *"No comuniques compartiendo memoria; comparte memoria comunicando."*

---

## Tabla de Contenidos

1. [¿Qué es una goroutine?](#qué-es-una-goroutine)
2. [Crear goroutines](#crear-goroutines)
3. [El problema: sincronización](#el-problema-sincronización)
4. [Canales sin buffer](#canales-sin-buffer)
5. [Canales con buffer](#canales-con-buffer)
6. [Range sobre canales](#range-sobre-canales)
7. [Canales direccionales](#canales-direccionales)
8. [Select](#select)
9. [Ejercicios propuestos](#ejercicios-propuestos)

---

## ¿Qué es una goroutine?

Una **goroutine** es una función que se ejecuta de forma **concurrente** con otras goroutines en el mismo espacio de direcciones. Son extremadamente ligeras (inician con ~2KB de pila vs ~1MB de un thread del sistema operativo) y son gestionadas por el runtime de Go, no por el sistema operativo.

```
Programa principal:
  main() ──┬── goroutine A: tarea1()
            ├── goroutine B: tarea2()
            └── goroutine C: tarea3()

Todas se ejecutan concurrentemente (posiblemente en paralelo)
```

---

## Crear goroutines

```go
package main

import (
    "fmt"
    "time"
)

func tarea(nombre string, duracion time.Duration) {
    fmt.Printf("▶ Iniciando %s\n", nombre)
    time.Sleep(duracion) // simula trabajo
    fmt.Printf("✓ Terminó %s\n", nombre)
}

func main() {
    fmt.Println("=== Sin goroutines (secuencial) ===")
    inicio := time.Now()
    tarea("A", 100*time.Millisecond)
    tarea("B", 200*time.Millisecond)
    tarea("C", 150*time.Millisecond)
    fmt.Printf("Total: %v\n\n", time.Since(inicio))

    fmt.Println("=== Con goroutines (concurrente) ===")
    inicio = time.Now()
    go tarea("X", 100*time.Millisecond)
    go tarea("Y", 200*time.Millisecond)
    go tarea("Z", 150*time.Millisecond)

    // ⚠️ Problema: main puede terminar antes que las goroutines
    time.Sleep(300 * time.Millisecond) // solución temporal (mala práctica)
    fmt.Printf("Total: %v\n", time.Since(inicio))
}
```

---

## El problema: sincronización

El `time.Sleep` anterior es una mala práctica. La forma correcta de esperar goroutines es con **WaitGroup** o **canales** (que veremos en la siguiente sección).

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func trabajador(id int, wg *sync.WaitGroup) {
    defer wg.Done() // decrementar el contador al terminar

    fmt.Printf("Trabajador %d: comenzando\n", id)
    time.Sleep(time.Duration(id*100) * time.Millisecond)
    fmt.Printf("Trabajador %d: terminado\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1) // incrementar antes de lanzar la goroutine
        go trabajador(i, &wg)
    }

    wg.Wait() // bloquea hasta que el contador llegue a 0
    fmt.Println("Todos los trabajadores han terminado")
}
```

---

## Canales sin buffer

Un **canal** es un conducto por donde las goroutines se envían valores. Los canales sin buffer **sincronizan**: el envío bloquea hasta que alguien recibe, y viceversa.

```go
package main

import "fmt"

func calcularCuadrado(n int, ch chan int) {
    ch <- n * n // enviar al canal
}

func main() {
    // Crear canal con make(chan tipo)
    ch := make(chan int)

    go calcularCuadrado(7, ch)

    // Recibir del canal (bloquea hasta que llegue un valor)
    resultado := <-ch
    fmt.Println("7² =", resultado)

    // Ping-pong entre goroutines
    ping := make(chan string)
    pong := make(chan string)

    go func() {
        msg := <-ping // recibe del canal ping
        fmt.Println("Recibido:", msg)
        pong <- msg + "-pong" // envía al canal pong
    }()

    ping <- "ping"
    respuesta := <-pong
    fmt.Println("Respuesta:", respuesta)
}
```

### Patrón productor-consumidor básico

```go
package main

import "fmt"

func productor(ch chan<- int, n int) {
    for i := 0; i < n; i++ {
        ch <- i * i // produce cuadrados
    }
    close(ch) // señal: no habrá más valores
}

func main() {
    ch := make(chan int)

    go productor(ch, 6)

    // El consumidor recibe hasta que el canal se cierra
    for valor := range ch {
        fmt.Println("Consumido:", valor)
    }
    fmt.Println("Canal cerrado, fin")
}
```

---

## Canales con buffer

Un canal con buffer acepta hasta N valores sin que haya un receptor esperando. Solo bloquea cuando el buffer está lleno (envío) o vacío (recepción).

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Canal con capacidad para 3 mensajes
    mensajes := make(chan string, 3)

    // Podemos enviar sin bloquear (hay espacio en el buffer)
    mensajes <- "primero"
    mensajes <- "segundo"
    mensajes <- "tercero"
    // mensajes <- "cuarto" // ← esto bloquearía (buffer lleno)

    fmt.Println(<-mensajes) // primero
    fmt.Println(<-mensajes) // segundo
    fmt.Println(<-mensajes) // tercero

    // Ejemplo: semáforo (limitar concurrencia)
    const maxConcurrentes = 3
    semaforo := make(chan struct{}, maxConcurrentes)

    for i := 0; i < 10; i++ {
        semaforo <- struct{}{} // adquirir "permiso"
        go func(id int) {
            defer func() { <-semaforo }() // liberar al terminar
            fmt.Printf("Tarea %d ejecutándose\n", id)
            time.Sleep(50 * time.Millisecond)
        }(i)
    }

    // Esperar a que todas terminen vaciando el semáforo
    for i := 0; i < maxConcurrentes; i++ {
        semaforo <- struct{}{}
    }
    fmt.Println("Todas las tareas completadas")
}
```

---

## Range sobre canales

```go
package main

import "fmt"

func generarFibonacci(n int, ch chan int) {
    a, b := 0, 1
    for i := 0; i < n; i++ {
        ch <- a
        a, b = b, a+b
    }
    close(ch) // IMPORTANTE: cerrar el canal para que range termine
}

func main() {
    ch := make(chan int, 10)
    go generarFibonacci(10, ch)

    fmt.Print("Fibonacci: ")
    for n := range ch { // itera hasta que el canal se cierre
        fmt.Printf("%d ", n)
    }
    fmt.Println()
}
```

---

## Canales direccionales

Restringen si una función puede enviar o recibir en un canal, añadiendo seguridad:

```go
package main

import "fmt"

// chan<- string : solo puede ENVIAR
func emisor(ch chan<- string) {
    ch <- "mensaje desde emisor"
    // msg := <-ch  // ❌ error de compilación: no puede recibir
}

// <-chan string : solo puede RECIBIR
func receptor(ch <-chan string) {
    msg := <-ch
    fmt.Println("Recibido:", msg)
    // ch <- "respuesta"  // ❌ error de compilación: no puede enviar
}

func main() {
    ch := make(chan string, 1)
    go emisor(ch)
    receptor(ch)
}
```

---

## Select

`select` permite esperar en **múltiples operaciones de canal** simultáneamente, ejecutando el caso que esté listo primero.

```go
package main

import (
    "fmt"
    "time"
)

func lento(ch chan string) {
    time.Sleep(200 * time.Millisecond)
    ch <- "resultado lento"
}

func rapido(ch chan string) {
    time.Sleep(50 * time.Millisecond)
    ch <- "resultado rápido"
}

func main() {
    ch1 := make(chan string, 1)
    ch2 := make(chan string, 1)

    go lento(ch1)
    go rapido(ch2)

    // select espera al primero que llegue
    select {
    case msg := <-ch1:
        fmt.Println("ch1 ganó:", msg)
    case msg := <-ch2:
        fmt.Println("ch2 ganó:", msg)
    }

    // select con timeout
    ch3 := make(chan int)
    select {
    case v := <-ch3:
        fmt.Println("Recibido:", v)
    case <-time.After(100 * time.Millisecond):
        fmt.Println("Timeout: nadie envió en 100ms")
    }

    // select no bloqueante con default
    ch4 := make(chan string, 1)
    select {
    case msg := <-ch4:
        fmt.Println("Recibido:", msg)
    default:
        fmt.Println("Canal vacío, continuando sin bloquear")
    }
}
```

### select para cancelación con done channel

```go
package main

import (
    "fmt"
    "time"
)

func worker(done <-chan struct{}) {
    for {
        select {
        case <-done:
            fmt.Println("Worker: señal de parada recibida")
            return
        default:
            fmt.Println("Worker: trabajando...")
            time.Sleep(100 * time.Millisecond)
        }
    }
}

func main() {
    done := make(chan struct{})

    go worker(done)

    time.Sleep(350 * time.Millisecond)
    close(done) // señal de parada a TODAS las goroutines que escuchan
    time.Sleep(50 * time.Millisecond)
    fmt.Println("Main: programa terminado")
}
```

---

## Ejercicios propuestos

### Ejercicio 1 — Pipeline de procesamiento
Crea un pipeline de 3 etapas usando canales:
1. `generar(n int) <-chan int` — emite números del 1 al n
2. `cuadrar(in <-chan int) <-chan int` — eleva al cuadrado
3. `filtrarPares(in <-chan int) <-chan int` — solo deja pasar pares

Usa el pipeline para imprimir los cuadrados pares de 1 a 10.

### Ejercicio 2 — Fan-out
Crea una función que tome un canal de entrada y lo distribuya a N canales de salida (un valor por canal, round-robin). Simula procesamiento paralelo.

### Ejercicio 3 — Timeout con context
Sin usar `context`, escribe una función `ejecutarConTimeout(fn func() int, timeout time.Duration) (int, error)` que:
- Ejecute `fn` en una goroutine
- Retorne su resultado si termina antes del timeout
- Retorne un error si el timeout expira

### Ejercicio 4 — Merge de canales
Escribe `merge(cs ...<-chan int) <-chan int` que combine múltiples canales en uno solo. Usa WaitGroup para cerrar el canal de salida cuando todos los de entrada se cierren.

<details>
<summary>💡 Pista para Ejercicio 3</summary>

Usa `select` con `time.After(timeout)` como uno de los casos. El resultado de `fn` lo puedes pasar por un canal de tamaño 1.

</details>
