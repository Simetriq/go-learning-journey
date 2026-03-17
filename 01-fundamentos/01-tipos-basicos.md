# Strings en Go

Los strings en Go son inmutables y se declaran con comillas dobles.

## Ejemplo básico

```go
package main

import "fmt"

func main() {
    mensaje := "Hola, mundo"
    fmt.Println(mensaje)
    fmt.Printf("El tipo es %T\n", mensaje)
}
```