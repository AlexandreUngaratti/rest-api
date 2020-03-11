# rest-api
REST, RESTful, O que é isso?

REST é um acronimo para “Representational State Transfer” ou na tradução mais genérica “Estado de representação para transferência”. REST É um padrão de arquitetura web e protocolo HTTP. O protocolo REST descreve 6 (seis) restrições:

    Interface uniforme
    Cacheable
    Client-Server
    Stateless
    Code-on-Demand (opcional)
    Layered System

REST é composto por métodos tais como uma URL base, tipo de medias e etc. Aplicações RESTful usam requisições HTTP para executar operações CRUD (Create Read Update Delete).

    Você pode lêr mais a respeito de REST e RESTful neste artigo do iMaster

Configurações

Para começarmos a escrever nossa API, nós precisaremos configurar nosso ambiente, para isso faça o download da ultima versão do Go (a versão atual até a data desta publicação é 1.9), baixe por este link. Após você concluir a instalação você terá disponivel em seu sistema uma variável de ambiente para $GOPATH.
Nesta pasta ($GOPATH) é onde precisaremos criar nosso código fonte, execute a seguinte linha no seu terminal:

$ mkdir -p $GOPATH/src/github/{seu usuario}/rest-api

Este comando criará uma pasta chamada api-rest no seu diretório Go.

$ cd rest-api

A seguir criaremos um arquivo chamado main.go
Rotas

Neste tutorial nós utilizaremos um pacote chamado mux para criarmos as rotas onde receberemos as requisições. Dentro da nossa pasta rest-api utilizar o seguinte comando para pegar este pacote:

rest-api$ go get github.com/gorilla/mux

Em nosso arquivo main.go

package main

import (
    "encoding/json"
    "log"
    "net/http"
    "github.com/gorilla/mux"
)

// função principal
func main() {
    router := mux.NewRouter()
    log.Fatal(http.ListenAndServe(":8000", router))
}

Se você executar este arquivo ele não terá nenhum retorno/resposta porque ainda não registramos nenhuma rota (endpoint). Para nosso projeto nós precisaremos das seguintes rotas:

    /contato (GET) -> Todos os contatos da agenda (banco de dados)
    /contato/{id} (GET) -> Retorna um único contato
    /contato/{id} (POST) -> Cria um novo contato
    /contato/{ID} (DELETE) -> Apaga um contato

Vamos escrever nossas rotas com os seus respectivos métodos citados acima:

// função principal
func main() {
    router := mux.NewRouter()
    router.HandleFunc("/contato", GetPeople).Methods("GET")
    router.HandleFunc("/contato/{id}", GetPerson).Methods("GET")
    router.HandleFunc("/contato/{id}", CreatePerson).Methods("POST")
    router.HandleFunc("/contato/{id}", DeletePerson).Methods("DELETE")    log.Fatal(http.ListenAndServe(":8000", router))
}

Execute o código:

rest-api$ go build
rest-api$ ./rest-api

O código acima irá gerar um erro, isso porque não criamos as funções para manusear as requisições, vejamos abaixo como faremos estas funções (vazias por enquanto):

...func GetPeople(w http.ResponseWriter, r *http.Request) {}
func GetPerson(w http.ResponseWriter, r *http.Request) {}
func CreatePerson(w http.ResponseWriter, r *http.Request) {}
func DeletePerson(w http.ResponseWriter, r *http.Request) {}...

Executando o código novamente nós não veremos nenhum erro, porém os métodos estão vazios.
Cada método recebe dois parâmetros w e r que são do tipo http.ResponseWriter e *http.Request respectivamente. Estes dois parâmetros são preenchidos sempre que uma rota é chamada.
Pegando todos os dados da agenda

Vamos escrever o código que será responsável por pegar todos os dados registrados em nossa agenda (nós não temos nenhum contato ainda). Vamos cadastrar alguns contatos manualmente (banco de dados estão fora do escopo deste tutorial).
Adicione uma struct Person (pense nisso como um objeto), uma struct Address e uma variável “people” para ser preenchida.

// arquivo main.go
...
type Person struct {
    ID        string   `json:"id,omitempty"`
    Firstname string   `json:"firstname,omitempty"`
    Lastname  string   `json:"lastname,omitempty"`
    Address   *Address `json:"address,omitempty"`
}
type Address struct {
    City  string `json:"city,omitempty"`
    State string `json:"state,omitempty"`
}

var people []Person
...

Com nossas structs prontas, vamos adicionar alguns contatos manualmente ao slice de person:

// função main()
...

people = append(people, Person{ID: "1", Firstname: "John", Lastname: "Doe", Address: &Address{City: "City X", State: "State X"}})people = append(people, Person{ID: "2", Firstname: "Koko", Lastname: "Doe", Address: &Address{City: "City Z", State: "State Y"}})people = append(people, Person{ID: "3", Firstname: "Francis", Lastname: "Sunday"})

...

Com os dados definidos, vamos retornar todos os contatos que estão no slice de “people”:

//main.go
...

func GetPeople(w http.ResponseWriter, r *http.Request) {
    json.NewEncoder(w).Encode(people)
}

...

Executando novamente go build && ./rest-api e realizando uma requisição com o postman nós veremos a resposta em json . Você pode testar também através do navegador no endereço: localhost:8080/contato
Pegando apenas um contato

Agora iremos escrever o código responsável por retornar apenas um contato da nossa agenda.

// arquivo main.go
...

func GetPerson(w http.ResponseWriter, r *http.Request) {
    params := mux.Vars(r)
    for _, item := range people {
        if item.ID == params["id"] {
            json.NewEncoder(w).Encode(item)
            return
        }
    }
    json.NewEncoder(w).Encode(&Person{})
}

...

A função GetPerson realiza um loop pelo em nosso slice de pessoas para verificar se o id que foi passado pela requisição está presente em nosso slice.
Outros endpoints

Vamos finalizar nossos outros endpoints (CreatePerson, DeletePerson)

func CreatePerson(w http.ResponseWriter, r *http.Request) {
    params := mux.Vars(r)
    var person Person
    _ = json.NewDecoder(r.Body).Decode(&person)
    person.ID = params["id"]
    people = append(people, person)
    json.NewEncoder(w).Encode(people)
}func DeletePerson(w http.ResponseWriter, r *http.Request) {
    params := mux.Vars(r)
    for index, item := range people {
        if item.ID == params["id"] {
            people = append(people[:index], people[index+1:]...)
            break
        }
    json.NewEncoder(w).Encode(people)    }
}

Código completo

package main

import (
    "encoding/json"
    "github.com/gorilla/mux"
    "log"
    "net/http"
)

// "Person type" (tipo um objeto)
type Person struct {
    ID        string   `json:"id,omitempty"`
    Firstname string   `json:"firstname,omitempty"`
    Lastname  string   `json:"lastname,omitempty"`
    Address   *Address `json:"address,omitempty"`
}type Address struct {
    City  string `json:"city,omitempty"`
    State string `json:"state,omitempty"`
}

var people []Person

// GetPeople mostra todos os contatos da variável people
func GetPeople(w http.ResponseWriter, r *http.Request) {
    json.NewEncoder(w).Encode(people)
}

// GetPerson mostra apenas um contato
func GetPerson(w http.ResponseWriter, r *http.Request) {
    params := mux.Vars(r)
    for _, item := range people {
        if item.ID == params["id"] {
            json.NewEncoder(w).Encode(item)
            return
        }
    }
    json.NewEncoder(w).Encode(&Person{})
}

// CreatePerson cria um novo contato
func CreatePerson(w http.ResponseWriter, r *http.Request) {
    params := mux.Vars(r)
    var person Person
    _ = json.NewDecoder(r.Body).Decode(&person)
    person.ID = params["id"]
    people = append(people, person)
    json.NewEncoder(w).Encode(people)
}

// DeletePerson deleta um contato
func DeletePerson(w http.ResponseWriter, r *http.Request) {
    params := mux.Vars(r)
    for index, item := range people {
        if item.ID == params["id"] {
            people = append(people[:index], people[index+1:]...)
            break
        }
        json.NewEncoder(w).Encode(people)
    }
}

// função principal para executar a api
func main() {
    router := mux.NewRouter()    people = append(people, Person{ID: "1", Firstname: "John", Lastname: "Doe", Address: &Address{City: "City X", State: "State X"}})    people = append(people, Person{ID: "2", Firstname: "Koko", Lastname: "Doe", Address: &Address{City: "City Z", State: "State Y"}})    router.HandleFunc("/contato", GetPeople).Methods("GET")
    router.HandleFunc("/contato/{id}", GetPerson).Methods("GET")
    router.HandleFunc("/contato/{id}", CreatePerson).Methods("POST")
    router.HandleFunc("/contato/{id}", DeletePerson).Methods("DELETE")    log.Fatal(http.ListenAndServe(":8000", router))
}

Testando

Para testar nossa api, você pode usar tanto pelo navegador (você precisará de uma extensão para formatar o json para conseguir levar a resposta claramente) quanto pelo Postman
Conclusão

Você acabou de ver/construir uma api RESTful muito simples usando a linguagem Go. Enquanto utilizamos dados de exemplo manuais ao invés de usar o banco de dados, nós vimos como criar rotas que realizam várias operações com JSON e slices que são tipos de variáveis nativas em Go.
Esqueci algo importante? deixe um comentário!

Este artigo é uma tradução do artigo original escrito por Francis Sunday em dev.to: Building a RESTful API with Go
