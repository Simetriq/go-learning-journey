# 05 - Mini Proyectos Integradores

> **Objetivo:** Aplicar los conceptos de fundamentos, estructuras de datos, funciones y concurrencia en proyectos pequeños pero completos y ejecutables. Cada proyecto incluye instrucciones paso a paso y solución guiada.

---

## Tabla de Contenidos

1. [Proyecto 1 — Gestor de Tareas CLI](#proyecto-1--gestor-de-tareas-cli)
2. [Proyecto 2 — Analizador de Texto](#proyecto-2--analizador-de-texto)
3. [Proyecto 3 — Servidor de Caché Concurrente](#proyecto-3--servidor-de-caché-concurrente)
4. [Proyecto 4 — Mini Pipeline de ETL](#proyecto-4--mini-pipeline-de-etl)
5. [Ideas para proyectos adicionales](#ideas-para-proyectos-adicionales)

---

## Proyecto 1 — Gestor de Tareas CLI

**Conceptos integrados:** structs, slices, maps, funciones, closures, manejo de errores.

### Descripción
Un gestor de tareas de línea de comandos que permite agregar, listar, completar y eliminar tareas, con persistencia en memoria durante la sesión.

### Instrucciones paso a paso

**Paso 1:** Define la estructura de datos para una tarea.

**Paso 2:** Crea el repositorio de tareas con sus operaciones CRUD.

**Paso 3:** Implementa la interfaz de menú interactivo.

**Paso 4:** Agrega filtros y ordenamiento.

### Solución completa

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "sort"
    "strconv"
    "strings"
    "time"
)

// ─── Modelo ───────────────────────────────────────────────────────────────────

type Prioridad int

const (
    Baja   Prioridad = 1
    Media  Prioridad = 2
    Alta   Prioridad = 3
)

func (p Prioridad) String() string {
    switch p {
    case Alta:
        return "🔴 Alta"
    case Media:
        return "🟡 Media"
    default:
        return "🟢 Baja"
    }
}

type Tarea struct {
    ID          int
    Titulo      string
    Completada  bool
    Prioridad   Prioridad
    CreadaEn    time.Time
    CompletadaEn *time.Time
}

func (t Tarea) String() string {
    estado := "[ ]"
    if t.Completada {
        estado = "[✓]"
    }
    return fmt.Sprintf("#%d %s %-30s %s", t.ID, estado, t.Titulo, t.Prioridad)
}

// ─── Repositorio ──────────────────────────────────────────────────────────────

type Repositorio struct {
    tareas  []*Tarea
    nextID  int
}

func nuevoRepositorio() *Repositorio {
    return &Repositorio{nextID: 1}
}

func (r *Repositorio) Agregar(titulo string, prioridad Prioridad) *Tarea {
    t := &Tarea{
        ID:        r.nextID,
        Titulo:    titulo,
        Prioridad: prioridad,
        CreadaEn:  time.Now(),
    }
    r.tareas = append(r.tareas, t)
    r.nextID++
    return t
}

func (r *Repositorio) Completar(id int) error {
    t := r.buscarPorID(id)
    if t == nil {
        return fmt.Errorf("tarea #%d no encontrada", id)
    }
    if t.Completada {
        return fmt.Errorf("tarea #%d ya está completada", id)
    }
    ahora := time.Now()
    t.Completada = true
    t.CompletadaEn = &ahora
    return nil
}

func (r *Repositorio) Eliminar(id int) error {
    for i, t := range r.tareas {
        if t.ID == id {
            r.tareas = append(r.tareas[:i], r.tareas[i+1:]...)
            return nil
        }
    }
    return fmt.Errorf("tarea #%d no encontrada", id)
}

func (r *Repositorio) buscarPorID(id int) *Tarea {
    for _, t := range r.tareas {
        if t.ID == id {
            return t
        }
    }
    return nil
}

func (r *Repositorio) Listar(soloCompletadas *bool) []*Tarea {
    resultado := make([]*Tarea, 0, len(r.tareas))
    for _, t := range r.tareas {
        if soloCompletadas == nil ||
            *soloCompletadas == t.Completada {
            resultado = append(resultado, t)
        }
    }
    // Ordenar por prioridad descendente
    sort.Slice(resultado, func(i, j int) bool {
        return resultado[i].Prioridad > resultado[j].Prioridad
    })
    return resultado
}

func (r *Repositorio) Estadisticas() map[string]int {
    stats := map[string]int{"total": len(r.tareas)}
    for _, t := range r.tareas {
        if t.Completada {
            stats["completadas"]++
        } else {
            stats["pendientes"]++
        }
    }
    return stats
}

// ─── UI ───────────────────────────────────────────────────────────────────────

func leerLinea(prompt string) string {
    fmt.Print(prompt)
    reader := bufio.NewReader(os.Stdin)
    linea, _ := reader.ReadString('\n')
    return strings.TrimSpace(linea)
}

func imprimirMenu() {
    fmt.Println("\n╔══════════════════════════════╗")
    fmt.Println("║     GESTOR DE TAREAS GO      ║")
    fmt.Println("╠══════════════════════════════╣")
    fmt.Println("║  1. Agregar tarea            ║")
    fmt.Println("║  2. Listar todas             ║")
    fmt.Println("║  3. Listar pendientes        ║")
    fmt.Println("║  4. Completar tarea          ║")
    fmt.Println("║  5. Eliminar tarea           ║")
    fmt.Println("║  6. Estadísticas             ║")
    fmt.Println("║  0. Salir                    ║")
    fmt.Println("╚══════════════════════════════╝")
}

func main() {
    repo := nuevoRepositorio()

    // Datos de ejemplo
    repo.Agregar("Aprender goroutines", Alta)
    repo.Agregar("Leer documentación de Go", Media)
    repo.Agregar("Hacer ejercicios de slices", Baja)
    _ = repo.Completar(2)

    for {
        imprimirMenu()
        opcion := leerLinea("Opción: ")

        switch opcion {
        case "1":
            titulo := leerLinea("Título: ")
            if titulo == "" {
                fmt.Println("❌ El título no puede estar vacío")
                continue
            }
            fmt.Println("Prioridad: 1=Baja, 2=Media, 3=Alta")
            pStr := leerLinea("Prioridad (default=2): ")
            p := Media
            if n, err := strconv.Atoi(pStr); err == nil {
                p = Prioridad(n)
            }
            t := repo.Agregar(titulo, p)
            fmt.Printf("✅ Tarea #%d creada\n", t.ID)

        case "2":
            tareas := repo.Listar(nil)
            if len(tareas) == 0 {
                fmt.Println("No hay tareas")
                continue
            }
            for _, t := range tareas {
                fmt.Println(t)
            }

        case "3":
            pendiente := false
            tareas := repo.Listar(&pendiente)
            fmt.Printf("Tareas pendientes (%d):\n", len(tareas))
            for _, t := range tareas {
                fmt.Println(t)
            }

        case "4":
            idStr := leerLinea("ID de la tarea: ")
            id, err := strconv.Atoi(idStr)
            if err != nil {
                fmt.Println("❌ ID inválido")
                continue
            }
            if err := repo.Completar(id); err != nil {
                fmt.Println("❌", err)
            } else {
                fmt.Printf("✅ Tarea #%d completada\n", id)
            }

        case "5":
            idStr := leerLinea("ID a eliminar: ")
            id, err := strconv.Atoi(idStr)
            if err != nil {
                fmt.Println("❌ ID inválido")
                continue
            }
            if err := repo.Eliminar(id); err != nil {
                fmt.Println("❌", err)
            } else {
                fmt.Printf("🗑️  Tarea #%d eliminada\n", id)
            }

        case "6":
            stats := repo.Estadisticas()
            fmt.Printf("📊 Total: %d | Completadas: %d | Pendientes: %d\n",
                stats["total"], stats["completadas"], stats["pendientes"])

        case "0":
            fmt.Println("¡Hasta luego!")
            return

        default:
            fmt.Println("❌ Opción no válida")
        }
    }
}
```

---

## Proyecto 2 — Analizador de Texto

**Conceptos integrados:** strings, maps, slices, funciones de orden superior, sort.

### Descripción
Analiza un texto y produce estadísticas: frecuencia de palabras, longitud media, palabras más comunes, y detección de palíndromos.

```go
package main

import (
    "fmt"
    "sort"
    "strings"
    "unicode"
)

// ─── Análisis ─────────────────────────────────────────────────────────────────

type Estadisticas struct {
    TotalPalabras    int
    PalabrasUnicas   int
    LongitudMedia    float64
    PalabraMasLarga  string
    TopN             []ParFrecuencia
    Palindromos      []string
}

type ParFrecuencia struct {
    Palabra    string
    Frecuencia int
}

func limpiar(palabra string) string {
    return strings.Map(func(r rune) rune {
        if unicode.IsLetter(r) || unicode.IsDigit(r) {
            return unicode.ToLower(r)
        }
        return -1 // eliminar el carácter
    }, palabra)
}

func esPalindromo(s string) bool {
    runes := []rune(s)
    n := len(runes)
    for i := 0; i < n/2; i++ {
        if runes[i] != runes[n-1-i] {
            return false
        }
    }
    return n > 1
}

func analizar(texto string, topN int) Estadisticas {
    palabras := strings.Fields(texto)
    frecuencia := make(map[string]int)
    totalLongitud := 0
    palabraMasLarga := ""
    var palindromos []string

    for _, p := range palabras {
        limpia := limpiar(p)
        if limpia == "" {
            continue
        }
        frecuencia[limpia]++
        totalLongitud += len(limpia)
        if len(limpia) > len(palabraMasLarga) {
            palabraMasLarga = limpia
        }
        if esPalindromo(limpia) {
            palindromos = append(palindromos, limpia)
        }
    }

    // Convertir map a slice para ordenar
    pares := make([]ParFrecuencia, 0, len(frecuencia))
    for palabra, freq := range frecuencia {
        pares = append(pares, ParFrecuencia{palabra, freq})
    }
    sort.Slice(pares, func(i, j int) bool {
        if pares[i].Frecuencia == pares[j].Frecuencia {
            return pares[i].Palabra < pares[j].Palabra
        }
        return pares[i].Frecuencia > pares[j].Frecuencia
    })

    top := pares
    if len(top) > topN {
        top = top[:topN]
    }

    longitudMedia := 0.0
    total := len(frecuencia) // Total palabras únicas
    if total > 0 {
        longitudMedia = float64(totalLongitud) / float64(len(palabras))
    }

    return Estadisticas{
        TotalPalabras:   len(palabras),
        PalabrasUnicas:  len(frecuencia),
        LongitudMedia:   longitudMedia,
        PalabraMasLarga: palabraMasLarga,
        TopN:            top,
        Palindromos:     palindromos,
    }
}

func imprimirReporte(stats Estadisticas) {
    fmt.Println("\n╔════════════════════════════════╗")
    fmt.Println("║     ANÁLISIS DE TEXTO          ║")
    fmt.Println("╚════════════════════════════════╝")
    fmt.Printf("📝 Total palabras:     %d\n", stats.TotalPalabras)
    fmt.Printf("🔤 Palabras únicas:    %d\n", stats.PalabrasUnicas)
    fmt.Printf("📏 Longitud media:     %.2f caracteres\n", stats.LongitudMedia)
    fmt.Printf("🏆 Palabra más larga:  %q\n", stats.PalabraMasLarga)

    fmt.Printf("\n📊 Top %d palabras más frecuentes:\n", len(stats.TopN))
    for i, p := range stats.TopN {
        barra := strings.Repeat("█", p.Frecuencia)
        fmt.Printf("  %2d. %-15s %s (%d)\n", i+1, p.Palabra, barra, p.Frecuencia)
    }

    if len(stats.Palindromos) > 0 {
        fmt.Printf("\n🔄 Palíndromos encontrados: %v\n", stats.Palindromos)
    }
}

func main() {
    texto := `
    Go es un lenguaje de programación compilado y concurrente.
    Go fue diseñado en Google. Go tiene un sistema de tipos estático.
    El concurrencia en Go se logra con goroutines y canales.
    Ana es un nombre palindromo, y también lo es aba y el número 121.
    Go Go Go es rápido y eficiente. La eficiencia de Go es notable.
    `

    stats := analizar(texto, 5)
    imprimirReporte(stats)
}
```

---

## Proyecto 3 — Servidor de Caché Concurrente

**Conceptos integrados:** goroutines, canales, select, mutex, structs, interfaces.

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// ─── Caché con TTL ────────────────────────────────────────────────────────────

type entrada struct {
    valor     interface{}
    expiraEn  time.Time
}

type Cache struct {
    mu      sync.RWMutex
    datos   map[string]entrada
    hits    int
    misses  int
}

func NuevaCache() *Cache {
    c := &Cache{datos: make(map[string]entrada)}
    go c.limpiezaPeriodica(30 * time.Second)
    return c
}

func (c *Cache) Set(clave string, valor interface{}, ttl time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.datos[clave] = entrada{
        valor:    valor,
        expiraEn: time.Now().Add(ttl),
    }
}

func (c *Cache) Get(clave string) (interface{}, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()

    e, ok := c.datos[clave]
    if !ok || time.Now().After(e.expiraEn) {
        c.misses++
        return nil, false
    }
    c.hits++
    return e.valor, true
}

func (c *Cache) Delete(clave string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    delete(c.datos, clave)
}

func (c *Cache) limpiezaPeriodica(intervalo time.Duration) {
    ticker := time.NewTicker(intervalo)
    defer ticker.Stop()
    for range ticker.C {
        c.mu.Lock()
        ahora := time.Now()
        for k, e := range c.datos {
            if ahora.After(e.expiraEn) {
                delete(c.datos, k)
            }
        }
        c.mu.Unlock()
    }
}

func (c *Cache) Stats() (hits, misses, size int) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    return c.hits, c.misses, len(c.datos)
}

// ─── Simulación de uso concurrente ────────────────────────────────────────────

func simularCliente(id int, cache *Cache, wg *sync.WaitGroup) {
    defer wg.Done()

    claves := []string{"usuario:1", "usuario:2", "producto:42", "config:app"}

    for i := 0; i < 5; i++ {
        clave := claves[i%len(claves)]

        if val, ok := cache.Get(clave); ok {
            fmt.Printf("Cliente[%d] HIT  %s = %v\n", id, clave, val)
        } else {
            // Cache miss: "buscar en base de datos" y guardar
            fmt.Printf("Cliente[%d] MISS %s → fetching...\n", id, clave)
            time.Sleep(10 * time.Millisecond) // simula latencia DB
            cache.Set(clave, fmt.Sprintf("datos_%s_%d", clave, id), 2*time.Second)
        }

        time.Sleep(20 * time.Millisecond)
    }
}

func main() {
    cache := NuevaCache()

    // Pre-cargar algunos valores
    cache.Set("config:app", map[string]string{"env": "produccion", "version": "1.2"}, 1*time.Minute)

    // Lanzar clientes concurrentes
    var wg sync.WaitGroup
    numClientes := 5

    fmt.Printf("Iniciando %d clientes concurrentes...\n\n", numClientes)
    for i := 1; i <= numClientes; i++ {
        wg.Add(1)
        go simularCliente(i, cache, &wg)
    }

    wg.Wait()

    // Estadísticas finales
    hits, misses, size := cache.Stats()
    hitRate := 0.0
    if total := hits + misses; total > 0 {
        hitRate = float64(hits) / float64(total) * 100
    }

    fmt.Printf("\n📊 Estadísticas del caché:\n")
    fmt.Printf("   Hits:    %d\n", hits)
    fmt.Printf("   Misses:  %d\n", misses)
    fmt.Printf("   Hit rate: %.1f%%\n", hitRate)
    fmt.Printf("   Tamaño actual: %d entradas\n", size)
}
```

---

## Proyecto 4 — Mini Pipeline de ETL

**Conceptos integrados:** goroutines, canales, funciones variádicas, structs, manejo de errores.

ETL = Extract, Transform, Load. Un patrón muy común en procesamiento de datos.

```go
package main

import (
    "fmt"
    "strings"
    "sync"
    "unicode"
)

// ─── Tipos ────────────────────────────────────────────────────────────────────

type Registro struct {
    ID     int
    Nombre string
    Email  string
    Edad   int
    Pais   string
}

type Error struct {
    RegistroID int
    Campo      string
    Mensaje    string
}

type Resultado struct {
    Registro *Registro
    Errores  []Error
}

// ─── Extract ──────────────────────────────────────────────────────────────────

func extraer(datos []Registro) <-chan Registro {
    out := make(chan Registro, len(datos))
    go func() {
        defer close(out)
        for _, r := range datos {
            out <- r
        }
    }()
    return out
}

// ─── Transform ────────────────────────────────────────────────────────────────

type Transformacion func(*Registro) []Error

func normalizar(r *Registro) []Error {
    r.Nombre = strings.TrimSpace(r.Nombre)
    r.Email = strings.ToLower(strings.TrimSpace(r.Email))
    r.Pais = strings.ToUpper(strings.TrimSpace(r.Pais))
    return nil
}

func validar(r *Registro) []Error {
    var errs []Error

    if r.Nombre == "" {
        errs = append(errs, Error{r.ID, "nombre", "no puede estar vacío"})
    }

    emailValido := strings.Contains(r.Email, "@") && strings.Contains(r.Email, ".")
    if !emailValido {
        errs = append(errs, Error{r.ID, "email", "formato inválido: " + r.Email})
    }

    if r.Edad < 0 || r.Edad > 150 {
        errs = append(errs, Error{r.ID, "edad", fmt.Sprintf("valor %d fuera de rango", r.Edad)})
    }

    return errs
}

func enriquecer(r *Registro) []Error {
    // Extraer dominio del email como "empresa"
    if parts := strings.Split(r.Email, "@"); len(parts) == 2 {
        dominio := parts[1]
        // Capitalizar nombre si está todo en minúsculas
        if r.Nombre == strings.ToLower(r.Nombre) {
            runes := []rune(r.Nombre)
            if len(runes) > 0 {
                runes[0] = unicode.ToUpper(runes[0])
                r.Nombre = string(runes)
            }
        }
        _ = dominio
    }
    return nil
}

func transformar(entrada <-chan Registro, workers int, transforms ...Transformacion) <-chan Resultado {
    out := make(chan Resultado, workers)
    var wg sync.WaitGroup

    for i := 0; i < workers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for registro := range entrada {
                r := registro // copia para no modificar el original
                var todosErrores []Error
                for _, transform := range transforms {
                    if errs := transform(&r); len(errs) > 0 {
                        todosErrores = append(todosErrores, errs...)
                    }
                }
                out <- Resultado{Registro: &r, Errores: todosErrores}
            }
        }()
    }

    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}

// ─── Load ─────────────────────────────────────────────────────────────────────

type Reporte struct {
    Exitosos []Registro
    Fallidos []Resultado
}

func cargar(resultados <-chan Resultado) Reporte {
    var reporte Reporte
    for r := range resultados {
        if len(r.Errores) == 0 {
            reporte.Exitosos = append(reporte.Exitosos, *r.Registro)
        } else {
            reporte.Fallidos = append(reporte.Fallidos, r)
        }
    }
    return reporte
}

// ─── Main ─────────────────────────────────────────────────────────────────────

func main() {
    datos := []Registro{
        {1, "  ana garcia  ", "ANA@EJEMPLO.COM", 28, "  ar"},
        {2, "bruno", "bruno-sin-arroba", 34, "mx"},
        {3, "Carlos Ruiz", "carlos@empresa.org", -5, "co"},
        {4, "diana", "diana@test.io", 25, "es"},
        {5, "", "sin-nombre@test.com", 40, "pe"},
        {6, "elena martinez", "elena@go.dev", 31, "uy"},
    }

    fmt.Println("🚀 Iniciando pipeline ETL...")
    fmt.Printf("   Registros de entrada: %d\n\n", len(datos))

    // Pipeline
    stage1 := extraer(datos)
    stage2 := transformar(stage1, 3, normalizar, validar, enriquecer)
    reporte := cargar(stage2)

    // Reporte final
    fmt.Printf("✅ Registros exitosos (%d):\n", len(reporte.Exitosos))
    for _, r := range reporte.Exitosos {
        fmt.Printf("   #%d %-20s %-25s Edad:%-3d País:%s\n",
            r.ID, r.Nombre, r.Email, r.Edad, r.Pais)
    }

    fmt.Printf("\n❌ Registros con errores (%d):\n", len(reporte.Fallidos))
    for _, r := range reporte.Fallidos {
        fmt.Printf("   #%d %s\n", r.Registro.ID, r.Registro.Nombre)
        for _, e := range r.Errores {
            fmt.Printf("      → %s: %s\n", e.Campo, e.Mensaje)
        }
    }
}
```

---

## Ideas para proyectos adicionales

### 🎮 Juego de Adivinanzas
Usa `bufio`, `math/rand` y structs para un juego donde el usuario adivina un número con pistas. Agrega un sistema de puntuación y niveles de dificultad.

### 📁 Indexador de Archivos
Usa `os.ReadDir`, goroutines y un WaitGroup para indexar archivos en un directorio recursivamente, contando extensiones y tamaños.

### 🌡️ Convertidor de Unidades con Historial
Structs, maps y closures para un conversor de temperatura/longitud/peso que guarda el historial de conversiones y puede exportarlo.

### 📊 Mini Base de Datos en Memoria
Implementa una base de datos clave-valor con soporte para transacciones (commit/rollback) usando maps anidados, Mutex y el patrón Command.

### 🕐 Scheduler de Tareas
Usa goroutines, time.Ticker y canales para un scheduler que ejecute tareas a intervalos configurables con soporte de cancelación via context.

---

> 💡 **Tip final:** Para ejecutar cualquiera de estos proyectos con Code Runner en VSCode, guarda el archivo como `main.go` en una carpeta, abre una terminal integrada y ejecuta `go run main.go`. Para proyectos con múltiples archivos, usa `go run .`.
