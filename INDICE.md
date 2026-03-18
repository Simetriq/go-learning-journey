# 📚 Curso de Go — Índice General

> Documentación de aprendizaje de Go. Todos los ejemplos son ejecutables con **Code Runner** (VSCode) o con `go run archivo.go` desde la terminal.

---

## Estructura del curso

```
curso-go/
├── 01-fundamentos/          ✅ Completado
├── 02-estructuras-datos/    ✅ Generado
│   ├── 01-arrays.md
│   ├── 02-slices.md
│   ├── 03-maps.md
│   └── 04-structs-punteros.md
├── 03-funciones/            ✅ Generado
│   ├── 01-funciones-primera-clase.md
│   └── 02-variadicas-defer-panic.md
├── 04-concurrencia/         ✅ Generado
│   ├── 01-goroutines-canales.md
│   └── 02-patrones-concurrencia.md
└── 05-mini-proyectos/       ✅ Generado
    └── README.md
```

---

## 02 — Estructuras de Datos

| Archivo | Temas clave |
|---------|-------------|
| `01-arrays.md` | Declaración, inicialización, iteración, arrays multidimensionales |
| `02-slices.md` | make, append, copy, slicing, patrones (stack, filter, sort) |
| `03-maps.md` | CRUD, verificación de existencia, iteración ordenada, contadores |
| `04-structs-punteros.md` | Definición, métodos, composición, punteros, receiver types |

## 03 — Funciones

| Archivo | Temas clave |
|---------|-------------|
| `01-funciones-primera-clase.md` | Valores de función, múltiples retornos, anónimas, closures, HOF |
| `02-variadicas-defer-panic.md` | `...args`, defer (LIFO, recursos), panic, recover, patrones |

## 04 — Concurrencia

| Archivo | Temas clave |
|---------|-------------|
| `01-goroutines-canales.md` | `go`, WaitGroup, canales sin/con buffer, range, select, done channel |
| `02-patrones-concurrencia.md` | Mutex, RWMutex, Once, Worker Pool, Fan-out/Fan-in, Pipeline |

## 05 — Mini Proyectos

| Proyecto | Conceptos integrados |
|----------|----------------------|
| Gestor de Tareas CLI | structs, slices, maps, closures, errores |
| Analizador de Texto | maps, sort, funciones de orden superior |
| Caché Concurrente | goroutines, Mutex, TTL, simulación |
| Pipeline ETL | canales, worker pool, transformaciones |

---

## Cómo ejecutar los ejemplos

Cada bloque de código con `package main` y `func main()` es autocontenido:

```bash
# Copiar el código a un archivo
# Opción 1: Code Runner en VSCode (botón ▶ o Ctrl+Alt+N)

# Opción 2: terminal
go run main.go

# Opción 3: compilar y ejecutar
go build -o programa main.go
./programa

# Detectar race conditions
go run -race main.go
```

---

*Generado para aprendizaje de Go — Formato compatible con Markdown Preview (VSCode)*
