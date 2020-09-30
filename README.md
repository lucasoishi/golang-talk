# Golang

[Custom foo description](##Introdução)
## Conteúdo
- [Introdução](#introdução)
	- [Um pouco sobre Go](#um-pouco-sobre-go)
	- [Golang vs Python](#golang-vs-python)
	- [Vantagens](#vantagens)
	- [Desvantagens](#desvantagens)
- [Conding in Go](#coding-em-go)
	- [Structs](#structs)
	- [Métodos](#métodos)
	- [Interfaces](#interfaces)
	- [Defer](#defer)
	- [Error Handling](#error-handling)
		- [Panic](#panic)
		- [Recover](#recover)
	- [Concorrência](#concorrência)
		- [Goroutines](#goroutines)
		- [Channels](#channels)
		- [Select](#select)
## Introdução

### Um pouco sobre Go
- Desenvolvida por Robert Griesemer, Rob Pike, e Ken Thompson em 2009 com o intuito de melhor a produtividade e escalabilidade no desenvolvimento de softwares no Google
- Sintática similar a C
- Concorrencia derivada dos princípios de CSP (Communicating Sequencial Proccess) -> forma simples de lidar com problemas de comunicação entre tasks concorrentes (evitar race conditions)

### Golang vs Python
- Tipagem Estática vs Dinâmica
- Forma de lidar com Exceções
- Suporte limitado a OO
- Simplicidade: poucas maneiras de fazer as coisas
- Concorrencia built-in e fácil de usar
- Em termos de performance, Golang tende a ser mais rápido e utilizar menos memória

### Vantagens de GO
- Linguagem simples e fácil de aprender
	- Poucas formas de se fazer as coisas
	- Introdução de poucos conceitos novos na linguagem
- Concorrência
- Ótimos pacotes padrões
- Pouca necessidade de otimização
- Compilação rápida
- Bom suporte para testes

### Desvantagens
- Não tem suporte completo para OO
- Sem Generics
- Poucas libs third-party em comparação com outras linguagens mais maduras (java, python, etc)
## Coding em GO
```
package main

import (
	"fmt"
)

func main() {
  fmt.Println(funcA(10))
}

func funcA (v int32) float32 {
  var a int32 = 12
  var b int32
  b = 10
  i := v + a + b
  return float32(i)
}
```
### Structs
Struct é uma coleção de campos ([Golang Tour](https://tour.golang.org/moretypes/2))
Structs são objetos que podem ser comparados a classes
```
package main

import (
	"fmt"
)

type S struct {
	Str string
	I   int32
}

func main() {
	s := S{"a string", 1}
	fmt.Println("Number", s.I)
	fmt.Println(s.Str)
}
```
### Métodos
Em Go, métodos estão atrelados a tipos e não a classes, sendo funções com um argumento receptor especial  ([Golang Tour](https://tour.golang.org/methods/1))
```
package main

import (
	"fmt"
)

type S struct {
	Str string
}

type X string
func (s *S) Print() {
	fmt.Println(s.Str)
}
func (x X) Print() {
	fmt.Println(x)
}
func main() {
	s := S{"a string"}
	x := X("another string")
	s.Print()
	x.Print()
}

```

### Interfaces

Um tipo interface é definido por um conjunto de métodos
"Um valor de tipo interface pode conter qualquer valor que implementa esses métodos." ([Golang Tour](https://tour.golang.org/methods/9))
"Interfaces são abstrações que definem um comportamento em particular sem especificar os detalhes de como esse comportamento implementado" ([Medium](https://medium.com/swlh/using-go-interfaces-for-testable-code-d2e11b02dea))
```
package main

import (
	"fmt"
)

type StructType struct {
	Str string
}

type StringType string

func (s *StructType) Print() {
	fmt.Println(s.Str)
}
func (x StringType) Print() {
	fmt.Println(x)
}
func main() {
	var i Printer
	s := StructType{"a string"}
	x := StringType("another string")
	i = x // a StringType implements Printer
	i.Print()
	i = &s // a StructType pointer implements Printer
	i.Print()
	i = s // a StructType does not implements Printer
}

type Printer interface {
	Print()
}
```
Interfaces também são recursos importantes para fazer testes em Go. Uma das suas utilidades é possibilitar realizar *mocks* de determinados comportamentos com facilidade

External Package
![External-Package](https://raw.githubusercontent.com/lucasoishi/golang-talk/master/external-package.png)

Foo Package
![Foo-Package](https://raw.githubusercontent.com/lucasoishi/golang-talk/master/foo-package.png)

Arquivo Teste - o teste TestController_Failure falha pois não há um mock do comportamento da struct Client 
![Test-File](https://raw.githubusercontent.com/lucasoishi/golang-talk/master/test-1.png)

Refatoração de Foo-Package (agora utiliza interface para implementação do comportamento GetData, possibilitando mockar o comportamento nos testes)
![Foo-Package-Refactor](https://raw.githubusercontent.com/lucasoishi/golang-talk/master/foo-package-better.png)

Teste refeito, com caso de falha
![Test-Redone](https://raw.githubusercontent.com/lucasoishi/golang-talk/master/test-2.png)

### Defer
Instrução que adiará uma determinada função até o retorno da função entorno ([Golang Tour](https://tour.golang.org/flowcontrol/12))
```
package main

import (
	"fmt"
)

func main() {
	defer fmt.Println("world")
	fmt.Println("hello")
}
```
Quando há mais de um defer sendo declarado, a execução ocorre na ordem inversa a declaração no código
```
package main

import (
	"fmt"
)

func main() {
	defer fmt.Println("!")     //último a executar
	defer fmt.Println("world") //2ª a executar
	defer fmt.Println("hello") //1ª a executar
}
```
### Error Handling

Go não possui uma estrutura clássica de try-catch-finally
Para lidar com erros não críticos, Go utiliza um padrão de múltiplos retornos em uma função, como no exemplo abaixo ([Golang Tour](https://tour.golang.org/methods/19)), deixando para que o desenvolvedor decida como lidar com o erro

```
package main

import (
	"fmt"
	"time"
)

type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s",
		e.When, e.What)
}

func DivideNumbers(a, b float32) (float32, error) {
	if b == 0 {
		
		return 0, &MyError{
		time.Now(),
		"Cannot divide by zero",
	}
}
	return a / b, nil
}

func main() {
	a, _ := DivideNumbers(1, 2)
	fmt.Println(a)
	_, err := DivideNumbers(1, 0)
	if err != nil {
		fmt.Println(err)
	}
}

```
#### Panic

Para erros críticos, a recomendação é utilizar a *builtin function* panic
Ao utilizar um panic, o fluxo de controle é interrompido e começa a entrar em *pânico* e, caso não tenha nenhuma tratativa, o *stack trace* será imprimido

#### Recover

*Recover* é outra *builtin function* que Go disponibiliza para lidar com erros, mais especificamente quando um pânico acontece em um função
No fluxo normal, *recover* irá retornar nulo. Mas, caso um pânico aconteça, *recover* irá capturar o vlaor do pânico e voltar a execução normal
```
package main

import "fmt"

func main() {
	f()
	fmt.Println("Returned normally from f.")
}

func f() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered in f", r)
		} // caso esse if seja removido, o stack trace será imprimido no lugar da instrução acima
	}()
	fmt.Println("Calling g.")
	g(0)
	fmt.Println("Returned normally from g.")
}

func g(i int) {
	if i > 3 {
		fmt.Println("Panicking!")
		panic(fmt.Sprintf("%v", i))
	}
	defer fmt.Println("Defer in g", i)
	fmt.Println("Printing in g", i)
	g(i + 1)
}
``` 
[The Go Blog: Defer, Panic e Recover](https://blog.golang.org/defer-panic-and-recover)

### Concorrência

[Golang Tour - Concorrência](https://tour.golang.org/concurrency/1)

#### Goroutines

Go provê uma forma simples e fácil de implementar concorrência
*Goroutines* são funcões ou métodos que rodam de forma concorrente em um código de Go
```
package main

import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}
```
#### Channels

Canais são um conduto tipado através do qual você pode enviar e receber valores com o operador de canal, `<-` ([Golang Tour](https://go-tour-br.appspot.com/concurrency/2))
Canais funcionam como filas FIFO (**F**irst **I**n **F**irst **O**ut)
A maior utilidade de canais é possibilitar a sincronização de comunicação entre *goroutines*
```
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int) \\ canais devem ser criados antes de sua utilização com a instrução make
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}	
```
Canais podem ser unbuffered (`ch := make(chan int32)`), ou *buffered*, onde é fornecido um valor do *buffer* na sua criação (`ch := make(chan int, 100)`)
Caso o código tente enviar dados a um canal *buffered* que está cheio, o Go irá subir uma exceção

#### Select

Instrução que permite uma espera na goroutine sobre as operações de comunicação múltiplas ([Golang Tour](https://go-tour-br.appspot.com/concurrency/5))
```
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 1
	}()
	fibonacci(c, quit)
}
```

## Referências/Links úteis
[Study Case GetStream](https://getstream.io/blog/switched-python-go/)
[Benchmark Go vs Python](https://benchmarksgame-team.pages.debian.net/benchmarksgame/fastest/go-python3.html)
[GoTour PT-BR](https://go-tour-br.appspot.com/welcome/1)
[Go PlayGround](https://play.golang.org/)
[HowIStart Golang](https://howistart.org/posts/go/1/)
[Go Wiki](https://github.com/golang/go/wiki)
[Medium - Using Go Interfaces for Testable Code](https://medium.com/swlh/using-go-interfaces-for-testable-code-d2e11b02dea)