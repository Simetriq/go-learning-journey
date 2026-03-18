# 02 - Patrones de Concurrencia: Worker Pools, Mutex y Sincronización

> **Objetivo:** Aprender los patrones de concurrencia más usados en Go: worker pools para procesar tareas en paralelo, protección de datos compartidos con Mutex, y coordinación avanzada con WaitGroup y Once.

---

## Tabla de Contenidos

1. [WaitGroup avanzado](#waitgroup-avanzado)
2. [Mutex: proteger datos compartidos](#mutex-proteger-datos-compartidos)
3. [RWMutex: lecturas concurrentes](#rwmutex-lecturas-concurrentes)
4. [sync.Once: inicialización única](#synconce-inicialización-única)
5. [Worker Pool](#worker-pool)
6. [Fan-out / Fan-in](#fan-out--fan-in)
7. [Pipeline encadenado](#pipeline-encadenado)
8. [Ejercicios propuestos](#ejercicios-propuestos)

---

## WaitGroup avanzado

`sync.WaitGroup` coordina la espera de múltiples goroutines. La regla de oro: llamar `Add` **antes** de lanzar la goroutine.

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Resultado struct {
    ID    int
    Valor int
    Error error
}

func procesarItem(id, valor int, wg *sync.WaitGroup, resultados chan<- Resultado) {
    defer wg.Done()

    // Simula trabajo de distinta duración
    time.Sleep(time.Duration(valor*10) * time.Millisecond)

    resultados <- Resultado{
        ID:    id,
        Valor: valor * valor,
    }
}

func main() {
    items := []int{5, 3, 8, 1, 7, 2, 9, 4, 6}
    resultados := make(chan Resultado, len(items))

    var wg sync.WaitGroup
    for i, v := range items {
        wg.Add(1)
        go procesarItem(i, v, &wg, resultados)
    }

    // Cerrar el canal cuando todas las goroutines terminen
    go func() {
        wg.Wait()
        close(resultados)
    }()

    // Recolectar resultados
    fmt.Println("Resultados (en orden de llegada):")
    for r := range resultados {
        fmt.Printf("  Item[%d]: %d² = %d\n", r.ID, items[r.ID], r.Valor)
    }
}
```

---

## Mutex: proteger datos compartidos

Cuando múltiples goroutines acceden a datos compartidos, puede ocurrir una **race condition**. `sync.Mutex` garantiza acceso exclusivo.

```go
package main

import (
    "fmt"
    "sync"
)

// ❌ INCORRECTO: race condition (ejecutar con -race para detectar)
func sinMutex() {
    contador := 0
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            contador++ // ← RACE CONDITION: lectura-modificación-escritura no es atómica
        }()
    }
    wg.Wait()
    fmt.Println("Sin mutex (resultado incorrecto):", contador)
}

// ✅ CORRECTO: usando Mutex
type ContadorSeguro struct {
    mu    sync.Mutex
    valor int
}

func (c *ContadorSeguro) Incrementar() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.valor++
}

func (c *ContadorSeguro) Valor() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.valor
}

func conMutex() {
    contador := &ContadorSeguro{}
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            contador.Incrementar()
        }()
    }
    wg.Wait()
    fmt.Println("Con mutex (resultado correcto):", contador.Valor())
}

func main() {
    sinMutex()
    conMutex()
}
```

---

## RWMutex: lecturas concurrentes

Cuando hay muchas lecturas y pocas escrituras, `sync.RWMutex` permite lecturas concurrentes pero escrituras exclusivas:

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()         // múltiples lecturas concurrentes OK
    defer c.mu.RUnlock()
    val, ok := c.data[key]
    return val, ok
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()   // escritura exclusiva
    defer c.mu.Unlock()
    c.data[key] = value
}

func main() {
    cache := &Cache{data: make(map[string]string)}

    // Escritura inicial
    cache.Set("go", "lenguaje compilado y concurrente")
    cache.Set("rust", "lenguaje de sistemas")

    var wg sync.WaitGroup

    // 10 lectores concurrentes
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            if val, ok := cache.Get("go"); ok {
                _ = val // usar el valor
            }
            time.Sleep(10 * time.Millisecond)
        }(i)
    }

    // 2 escritores ocasionales
    for i := 0; i < 2; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            cache.Set(fmt.Sprintf("nuevo_%d", id), "valor")
        }(i)
    }

    wg.Wait()
    fmt.Println("Cache size:", func() int {
        cache.mu.RLock()
        defer cache.mu.RUnlock()
        return len(cache.data)
    }())
}
```

---

## sync.Once: inicialización única

`sync.Once` garantiza que una función se ejecute **una sola vez**, independientemente de cuántas goroutines intenten llamarla. Ideal para el patrón singleton.

```go
package main

import (
    "fmt"
    "sync"
)

type Configuracion struct {
    DSN      string
    MaxConns int
}

var (
    instanciaConfig *Configuracion
    once            sync.Once
)

func obtenerConfig() *Configuracion {
    once.Do(func() {
        fmt.Println("⚙️  Inicializando configuración (solo una vez)...")
        instanciaConfig = &Configuracion{
            DSN:      "postgres://localhost:5432/midb",
            MaxConns: 10,
        }
    })
    return instanciaConfig
}

func main() {
    var wg sync.WaitGroup

    // 5 goroutines intentan obtener la configuración
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            cfg := obtenerConfig()
            fmt.Printf("Goroutine %d usa config: %s\n", id, cfg.DSN)
        }(i)
    }

    wg.Wait()
    // El mensaje de inicialización aparece UNA sola vez
}
```

---

## Worker Pool

El patrón **Worker Pool** limita la concurrencia a N trabajadores procesando una cola de tareas. Evita crear demasiadas goroutines.

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Tarea struct {
    ID   int
    Data int
}

type ResultadoTarea struct {
    TareaID  int
    Resultado int
    Error    error
}

func worker(id int, tareas <-chan Tarea, resultados chan<- ResultadoTarea, wg *sync.WaitGroup) {
    defer wg.Done()

    for tarea := range tareas {
        // Simular procesamiento costoso
        time.Sleep(time.Duration(tarea.Data*5) * time.Millisecond)

        resultados <- ResultadoTarea{
            TareaID:   tarea.ID,
            Resultado: tarea.Data * tarea.Data,
        }
        fmt.Printf("  Worker[%d] completó tarea[%d]: %d² = %d\n",
            id, tarea.ID, tarea.Data, tarea.Data*tarea.Data)
    }
}

func workerPool(numWorkers int, tareas []Tarea) []ResultadoTarea {
    tareaCh := make(chan Tarea, len(tareas))
    resultadoCh := make(chan ResultadoTarea, len(tareas))

    // Lanzar N workers
    var wg sync.WaitGroup
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go worker(i, tareaCh, resultadoCh, &wg)
    }

    // Enviar todas las tareas
    for _, t := range tareas {
        tareaCh <- t
    }
    close(tareaCh) // señal: no habrá más tareas

    // Cerrar resultados cuando todos los workers terminen
    go func() {
        wg.Wait()
        close(resultadoCh)
    }()

    // Recolectar resultados
    var resultados []ResultadoTarea
    for r := range resultadoCh {
        resultados = append(resultados, r)
    }
    return resultados
}

func main() {
    tareas := make([]Tarea, 12)
    for i := range tareas {
        tareas[i] = Tarea{ID: i + 1, Data: i + 1}
    }

    fmt.Printf("Procesando %d tareas con 3 workers...\n", len(tareas))
    inicio := time.Now()
    resultados := workerPool(3, tareas)
    fmt.Printf("\nCompletado en %v. %d resultados obtenidos.\n",
        time.Since(inicio), len(resultados))
}
```

---

## Fan-out / Fan-in

**Fan-out:** distribuir trabajo entre múltiples goroutines.  
**Fan-in:** combinar múltiples resultados en un solo canal.

```go
package main

import (
    "fmt"
    "sync"
)

// Fan-out: distribuye el trabajo
func fanOut(entrada <-chan int, numWorkers int) []<-chan int {
    salidas := make([]<-chan int, numWorkers)

    for i := 0; i < numWorkers; i++ {
        salida := make(chan int)
        salidas[i] = salida

        go func(out chan<- int) {
            for n := range entrada {
                out <- n * n // procesar (cuadrado)
            }
            close(out)
        }(salida)
    }
    return salidas
}

// Fan-in: combina múltiples canales en uno
func fanIn(canales ...<-chan int) <-chan int {
    combinado := make(chan int)
    var wg sync.WaitGroup

    reenviar := func(ch <-chan int) {
        defer wg.Done()
        for v := range ch {
            combinado <- v
        }
    }

    for _, ch := range canales {
        wg.Add(1)
        go reenviar(ch)
    }

    go func() {
        wg.Wait()
        close(combinado)
    }()

    return combinado
}

func main() {
    // Generar números del 1 al 9
    entrada := make(chan int, 9)
    for i := 1; i <= 9; i++ {
        entrada <- i
    }
    close(entrada)

    // Fan-out a 3 workers
    salidas := fanOut(entrada, 3)

    // Fan-in: combinar resultados
    combinado := fanIn(salidas...)

    fmt.Println("Cuadrados (orden no garantizado):")
    for v := range combinado {
        fmt.Printf("  %d\n", v)
    }
}
```

---

## Pipeline encadenado

Combinar las técnicas anteriores en un pipeline completo:

```go
package main

import (
    "fmt"
    "strings"
    "sync"
)

// Etapa 1: generar strings
func generar(palabras ...string) <-chan string {
    out := make(chan string)
    go func() {
        defer close(out)
        for _, p := range palabras {
            out <- p
        }
    }()
    return out
}

// Etapa 2: transformar (paralelo con N workers)
func transformar(entrada <-chan string, n int) <-chan string {
    out := make(chan string, n)
    var wg sync.WaitGroup

    for i := 0; i < n; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for palabra := range entrada {
                out <- strings.ToUpper(palabra)
            }
        }()
    }

    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}

// Etapa 3: filtrar
func filtrar(entrada <-chan string, minLen int) <-chan string {
    out := make(chan string)
    go func() {
        defer close(out)
        for s := range entrada {
            if len(s) >= minLen {
                out <- s
            }
        }
    }()
    return out
}

func main() {
    palabras := []string{"go", "python", "rust", "java", "c", "kotlin", "ts"}

    // Pipeline: generar → transformar(paralelo) → filtrar
    etapa1 := generar(palabras...)
    etapa2 := transformar(etapa1, 3) // 3 workers en paralelo
    etapa3 := filtrar(etapa2, 4)     // solo palabras de 4+ letras

    fmt.Println("Palabras de 4+ letras (mayúsculas):")
    for palabra := range etapa3 {
        fmt.Println(" ", palabra)
    }
}
```

---

## Ejercicios propuestos

### Ejercicio 1 — Worker Pool con errores
Modifica el Worker Pool para que las tareas puedan retornar errores. Las tareas con ID par fallan. Recolecta tanto los resultados exitosos como los errores.

### Ejercicio 2 — Rate Limiter
Implementa un rate limiter que permita máximo 5 operaciones por segundo usando `time.Tick` y un canal. Simula 20 requests y muestra cuándo se ejecuta cada uno.

### Ejercicio 3 — Búsqueda concurrente
Dado un slice de 1000 números, encuentra todos los números primos usando N goroutines para dividir el trabajo. Compara el tiempo con la versión secuencial.

### Ejercicio 4 — Caché concurrente
Implementa un `CacheTTL` (struct) que:
- Almacene pares clave-valor con tiempo de expiración
- Use `RWMutex` para acceso concurrente
- Tenga una goroutine de limpieza que elimine entradas expiradas cada segundo
- Exponga métodos `Set(key, value string, ttl time.Duration)` y `Get(key string) (string, bool)`

<details>
<summary>💡 Pista para Ejercicio 3</summary>

Divide el rango [2, 1000] en N segmentos iguales. Cada goroutine procesa su segmento y envía los primos por un canal. Usa fan-in para combinar resultados.

</details>
