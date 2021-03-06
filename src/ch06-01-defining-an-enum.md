## Definindo uma Enum

Vamos ver um caso em que enums podem ser mais apropriadas do que structs e
descobrir como elas podem ser úteis. Digamos que estamos trabalhando com
endereços IP. Atualmente, existem duas versões do protocolo IP que são mais
utilizadas: a quatro e a seis. Estas são as únicas possibilidades para um
endereço IP com que o nosso programa vai trabalhar: nós podemos *enumerar*
todos os possíveis valores, é daí que vem o nome enumeração.

Um endereço IP qualquer pode ser ou da versão quatro ou da versão seis, mas
nunca das duas ao mesmo tempo. Esta propriedade dos endereços IP faz com que a
enum seja bem apropriada para este caso, pois enums só podem assumir o valor de
uma de suas variantes. Os endereços de ambas as versões, seja quatro ou seis,
ainda são, fundamentalmente, endereços IP, e deveriam ser tratados pelo mesmo
tipo no código em situações que se aplicam a qualquer versão de endereço IP.

Podemos expressar esse conceito em código definindo uma enum `VersaoIp` e
listando os possíveis tipos de que um endereço IP pode ser: `V4` e `V6`. Estas
são as chamadas *variantes* da enum:

```rust
enum VersaoIp {
    V4,
    V6,
}
```

`VersaoIp` é um tipo de dados que agora nós podemos usar em qualquer lugar no
nosso código.

### Valores de uma Enum

Podemos criar instâncias de cada uma das duas variantes de `VersaoIp`, da
seguinte forma:

```rust
# enum VersaoIp {
#     V4,
#     V6,
# }
#
let quatro = VersaoIp::V4;
let seis = VersaoIp::V6;
```

Repare que as variantes pertencem ao _namespace_ da enum, e se usa `::` para
separar os dois. Isso é útil porque agora ambos os valores `VersaoIp::V4` e
`VersaoIp::V6` são do mesmo tipo: `VersaoIp`. Agora nós podemos, por exemplo,
definir uma função que usa qualquer `VersaoIp`.

```rust
# enum VersaoIp {
#     V4,
#     V6,
# }
#
fn rotear(versao_ip: VersaoIp) { }
```

E podemos ainda chamar esta função passando qualquer uma das variantes:

```rust
# enum VersaoIp {
#     V4,
#     V6,
# }
#
# fn rotear(versao_ip: VersaoIp) { }
#
rotear(VersaoIp::V4);
rotear(VersaoIp::V6);
```

O uso de enums tem ainda mais vantagens. Pensando mais a fundo sobre o nosso
tipo de endereço IP, ainda não temos uma forma de representar o *endereço* em
si, apenas sabemos qual a *versão* dele. Tendo em vista o que você acabou de
aprender sobre structs no Capítulo 5, você poderia abordar esse problema assim
como visto na Listagem 6-1:

```rust
enum VersaoIp {
    V4,
    V6,
}

struct EnderecoIp {
    versao: VersaoIp,
    endereco: String,
}

let local = EnderecoIp {
    versao: VersaoIp::V4,
    endereco: String::from("127.0.0.1"),
};

let loopback = EnderecoIp {
    versao: VersaoIp::V6,
    endereco: String::from("::1"),
};
```

<span class="caption">Listagem 6-1: Representação do endereço e da variante
`VersaoIp` de um endereço IP usando uma `struct`</span>

Aqui nós definimos uma struct `EnderecoIp` que tem dois membros: `versao`, do
tipo `VersaoIp` (que definimos anteriormente) e `endereco`, do tipo `String`.
Temos duas instâncias dessa struct. A primeira, `local`, tem o valor
`VersaoIp::V4` como sua `versao`, e um endereço associado igual a `127.0.0.1`.
A segunda instância, `loopback`, tem como sua `versao` a outra variante de
`VersaoIp`, `V6`, e o endereço `::1` associado a ela. Nós usamos uma struct
para encapsular os valores de `versao` e `endereco`, agora a variante está
associada ao valor.

Podemos representar o mesmo conceito de uma forma mais concisa usando apenas
uma enum, em vez de uma enum dentro de uma struct, colocando dados dentro de
cada variante da enum, diretamente. Esta nova definição da enum `EnderecoIp`
diz que ambas as variantes, `V4` e `V6`, terão uma `String` associada:

```rust
enum EnderecoIp {
    V4(String),
    V6(String),
}

let local = EnderecoIp::V4(String::from("127.0.0.1"));

let loopback = EnderecoIp::V6(String::from("::1"));
```

Podemos anexar dados a cada variante da enum diretamente, assim não existe mais
a necessidade de uma struct adicional.

Há uma outra vantagem de se usar uma enum em vez de uma struct: cada variante
pode conter dados de diferentes tipos e quantidades. Os endereços IP da versão
quatro têm sempre quatro componentes numéricas, cada uma com valor de 0 a 255.
Se quiséssemos representar endereços `V4` como quatro valores `u8`, e ao mesmo
tempo manter os endereços `V6` como uma `String`, não poderíamos usar uma
struct. Já as enums podem facilmente atender a este caso:

```rust
enum EnderecoIp {
    V4(u8, u8, u8, u8),
    V6(String),
}

let local = EnderecoIp::V4(127, 0, 0, 1);

let loopback = EnderecoIp::V6(String::from("::1"));
```

Acabamos de ver algumas possibilidades que poderíamos usar para representar
endereços IP das duas versões por meio de uma enum. Acontece que essa
necessidade de representar endereços IP, incluindo sua versão, é tão comum que
a biblioteca padrão já possui uma definição que podemos usar! ([Veja a
documentação em inglês][IpAddr]<!-- ignore -->). Vamos ver como a biblioteca
padrão define `IpAddr`: ele tem basicamente a mesma enum e as mesmas variantes
que nós definimos e usamos anteriormente, mas os dados do endereço são
embutidos dentro das variantes na forma de duas structs separadas, que são
definidas de um jeito diferente pra cada variante.

[IpAddr]: https://doc.rust-lang.org/std/net/enum.IpAddr.html

```rust
struct Ipv4Addr {
    // detalhes omitidos
}

struct Ipv6Addr {
    // detalhes omitidos
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Esse código mostra que você pode colocar qualquer tipo de dados dentro de uma
variante de enum: strings, tipos numéricos ou structs, por exemplo. Você pode
até mesmo incluir outra enum! Além disso, os tipos definidos pela biblioteca
padrão não são tão mais complicados do que o que talvez você pensaria em fazer.

Repare que, mesmo havendo um `IpAddr`definido pela biblioteca padrão, nós ainda
podemos criar e utilizar nossa própria definição (com o mesmo nome, inclusive)
sem nenhum conflito, porque não trouxemos a definição da biblioteca padrão para
dentro do nosso escopo. Falaremos mais sobre a inclusão de tipos em um escopo
no Capítulo 7.

Vamos ver outro exemplo de uma enum na Listagem 6-2: esta tem uma grande
variedade de tipos embutidos nas suas variantes:

```rust
enum Mensagem {
    Sair,
    Mover { x: i32, y: i32 },
    Escrever(String),
    MudarCor(i32, i32, i32),
}
```

<span class="caption">Listagem 6-2: Enum `Mensagem`, cujas variantes contêm,
cada uma, diferentes tipos e quantidades de dados</span>

Esta enum tem quatro variantes de diferentes tipos:

* `Sair` não tem nenhum dado associado.
* `Mover` contém uma struct anônima.
* `Escrever` contém uma única `String`.
* `MudarCor` contém três valores do tipo `i32`.

Definir uma enum com variantes iguais às da Listagem 6-2 é similar a definir
diferentes tipos de struct, exceto que a enum não usa a palavra-chave `struct`,
e todas as variantes são agrupadas dentro do tipo `Mensagem`. As structs
seguintes podem guardar os mesmos dados que as variantes da enum anterior:

```rust
struct MensagemSair; // unit struct
struct MensagemMover {
    x: i32,
    y: i32,
}
struct MensagemEscrever(String); // tuple struct
struct MensagemMudarCor(i32, i32, i32); // tuple struct
```

Mas se usarmos structs diferentes, cada uma tendo seu próprio tipo, não vamos
conseguir tão facilmente definir uma função que possa receber qualquer um
desses tipos de mensagens, assim como fizemos com a enum `Mensagem`, definida
na Listagem 6-2, que consiste em um tipo único.

Há mais uma similaridade entre enums e structs: da mesma forma como podemos
definir métodos em structs usando `impl`, também podemos definir métodos em
enums. Aqui está um método chamado `invocar`, que poderia ser definido na nossa
enum `Mensagem`:

```rust
# enum Mensagem {
#     Sair,
#     Mover { x: i32, y: i32 },
#     Escrever(String),
#     MudarCor(i32, i32, i32),
# }
#
impl Mensagem {
    fn invocar(&self) {
        // o corpo do método é definido aqui
    }
}

let m = Mensagem::Escrever(String::from("olá"));
m.invocar();
```

O corpo do método usaria o valor `self` para obter a mensagem sobre a qual o
método foi chamado. Neste exemplo, criamos a variável `m`, que contém o valor
`Mensagem::Escrever(String::from("olá"))`, e é isso que `self` vai ser no corpo
do método `invocar` quando `m.invocar()` for executado.

Vamos ver agora outra enum da biblioteca padrão que também é muito útil e
comum: `Option`.

### A Enum `Option` e Suas Vantagens Sobre Valores Nulos

Na seção anterior, vimos como a enum `EnderecoIp` nos permite usar o sistema de
tipos do Rust para codificar em nosso programa mais informação do que apenas os
dados que queremos representar. Essa seção explora um caso de estudo da
`Option`, que é outra enum definida pela biblioteca padrão. O tipo `Option` é
muito utilizado, pois engloba um cenário muito comum, em que um valor pode ser
algo ou pode não ser nada. Expressar esse conceito por meio do sistema de tipos
significa que o compilador pode verificar se você tratou, ou não, todos os
casos que deveriam ser tratados, podendo evitar _bugs_ que são extremamente
comuns em outras linguagens de programação.

O _design_ de uma linguagem de programação é geralmente tratado em termos de
quais características são incluídas, mas as que são excluídas também têm 
importância. Rust não tem o valor nulo (_null_) que outras linguagens têm. O
valor nulo quer dizer que não há nenhum valor. Em linguagens que têm essa
característica, as variáveis sempre estão em um dos dois estados: nulo ou não
nulo.

Em uma conferência, Tony Hoare, inventor do valor nulo, disse o seguinte:

> Eu o chamo meu erro de um bilhão de dólares. Naquela época, eu estava
> projetando o primeiro sistema abrangente de tipos para referências em uma
> linguagem orientada a objetos. Meu objetivo era garantir que todo uso de
> referências deveria ser absolutamente seguro, com verificação automática
> feita pelo compilador. Mas não pude resistir à tentação de colocar uma
> referência nula, simplesmente porque era tão fácil de implementar. Isso tem
> provocado inúmeros erros, vulnerabilidades, e falhas de sistemas que
> provavelmente causaram um bilhão de dólares de dor e danos nos últimos
> quarenta anos.

O problema com valores nulos é que, se você tentar usar um valor nulo como se
fosse não nulo, vai acontecer algum tipo de erro. Pelo fato dessa propriedade
de nulo e não nulo ser tão sutil, é extremamente fácil cometer esse tipo de
erro.

Porém, o conceito que o valor nulo tenta expressar ainda é útil: um valor nulo
representa algo que, por algum motivo, está inválido ou ausente no momento.

O problema, na verdade, não está no conceito, mas na implementação em
particular. Por isso, Rust não possui valores nulos, mas sim uma enum que
engloba o conceito de um valor estar presente ou ausente. Esta enum é a
`Option<T>`, que está definida na biblioteca padrão da seguinte forma:
([Veja a documentação em inglês][option]<!-- ignore -->).

[option]: https://doc.rust-lang.org/std/option/enum.Option.html

```rust
enum Option<T> {
    Some(T), // algum valor
    None, // nenhum valor
}
```

A enum `Option<T>` é tão útil que ela já vem inclusa no prelúdio: você não
precisa trazê-la explicitamente para o seu escopo. Além disso, o mesmo ocorre
com suas variantes: você pode usar `Some` e `None` diretamente sem prefixá-las
com `Option::`. `Option<T>` continua sendo uma enum como qualquer outra, e
`Some(T)` e `None` ainda são variantes do tipo `Option<T>`.

A sintaxe do `<T>` é uma característica do Rust de que não falamos ainda.
Trata-se de um parâmetro de tipo genérico, vamos abordá-lo com mais detalhe no
Capítulo 10. Por ora, tudo que você precisa saber é que `<T>` significa que a
variante `Some` da enum `Option` pode conter um dado de qualquer tipo. Aqui vão
alguns exemplos de `Option` contendo tipos de número e texto:

```rust
let algum_numero = Some(5);
let algum_texto = Some("um texto");

let numero_ausente: Option<i32> = None;
```

Se usamos `None` em vez de `Some`, precisamos dizer ao Rust qual é o tipo de
`Option<T>` que nós temos, porque o compilador não consegue inferir qual tipo
estará contido na variante `Some` apenas olhando para um valor `None`.

Quando temos um `Some`, sabemos que um valor está presente, contido dentro do
`Some`. Já quando temos um `None`, de certa forma, significa o mesmo que um
valor nulo: não temos um valor que seja válido. Então por que a `Option<T>` é
tão melhor que usar um valor nulo?

Em resumo, é porque `Option<T>` e `T` (podendo `T` ser de qualquer tipo) são
tipos diferentes, por isso, o compilador não vai permitir usar um valor do tipo
`Option<T>` como se ele definitivamente tivesse um valor válido. Por exemplo,
o código seguinte não vai compilar, porque ele está tentando somar um `i8` a um
`Option<i8>`:

```rust,ignore
let x: i8 = 5;
let y: Option<i8> = Some(5);

let soma = x + y;
```

Quando executamos esse código, temos uma mensagem de erro como essa:

```text
error[E0277]: the trait bound `i8: std::ops::Add<std::option::Option<i8>>` is
not satisfied
 -->
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + std::option::Option<i8>`
  |
```

Intenso! O que essa mensagem quer dizer é que o Rust não consegue entender como
somar um `i8` e um `Option<i8>`, porque eles são de tipos diferentes. Quando
temos um valor de um tipo como `i8` em Rust, o compilador tem certeza de que
temos sempre um valor válido. Podemos prosseguir com confiança, sem ter de
verificar se o valor é nulo antes de usá-lo. Somente quando temos um
`Option<i8>` (ou qualquer que seja o tipo com que estamos trabalhando), vamos
ter de nos preocupar com a possibilidade de não haver um valor, e o compilador
vai se certificar de que nós estamos tratando este caso antes de usar o valor.

Em outras palavras, você tem que converter um `Option<T>` em um `T` antes de
poder executar operações com ele. Geralmente, isso ajuda a detectar um dos
problemas mais comuns com valores nulos: assumir que algo não é nulo quando,
na verdade, ele é.

Só de não ter que se preocupar com a possibilidade de ter deixado um valor nulo
escapar já lhe dá mais confiança em seu código. Pra ter um valor que pode ser
nulo em algum momento, você precisa, explicitamente, marcá-lo como sendo do
tipo `Option<T>`. A partir daí, sempre que for usar o valor, você será obrigado
a tratar, de forma explícita, o caso do valor sendo nulo. Sempre que houver um
valor que não seja um `Option<T>`, você *pode* assumir, com segurança, que o
valor não é nulo. Esta foi uma decisão deliberada de projeto do Rust para
limitar as sutilezas dos valores nulos e aumentar a segurança do código.

Então, como obter o valor `T` da variante `Some` quando se tem um `Option<T>`,
para que se possa usar seu valor? A enum `Option<T>` possui diversos métodos
que são úteis em uma variedade de situações, você pode pesquisá-los na
[documentação][docs]<!-- ignore --> (em inglês). Será extremamente útil na sua
jornada com Rust se familizarizar com os métodos da enum `Option<T>`.

[docs]: https://doc.rust-lang.org/std/option/enum.Option.html

Em geral, pra usar um valor `Option<T>`, queremos ter um código que trate cada
uma das variantes. Queremos um código que só será executado quando tivermos um
valor `Some(T)`, e esse código terá permissão para usar o valor `T` que está
embutido. Queremos também um outro código que seja executado se tivermos um
valor `None`, e esse código não terá um valor `T` disponível. A expressão
`match` é uma instrução de controle de fluxo que faz exatamente isso quando
usada com enums: ela executa códigos diferentes dependendo de qual variante
tiver a enum, e esse código poderá usar os dados contidos na variante
encontrada.
