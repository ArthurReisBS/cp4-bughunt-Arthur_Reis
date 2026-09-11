# Checkpoint 4 — Bug Hunt StreamFIAP

> Copie este arquivo para a raiz do seu repositório com o nome **README.md**
> e preencha todas as seções.

## Identificação

**Grupo:** Arthur_Reis

| Integrante | RM | Turma |
|---|---|---|
| Arthur Reis | RM562181 | 2CCPW |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 02 / 6 |

---

## Parte 1 — Bugs encontrados

> Uma linha por bug, na ordem em que você os encontrou. Use a numeração dos seus
> commits (`fix: bug01 ...`). Preencha TODAS as colunas — metade da nota está aqui.

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | Cadastrei um usuario pelo POST /api/usuarios mandando o nome no JSON, mas a resposta voltou com o nome nulo e no banco salvou vazio também | No construtor da classe Usuario, linha 22, estava escrito nome = nome. O parametro estava sendo atribuido a ele mesmo, entao o atributo da classe nunca recebia valor nenhum | Coloquei o this na frente, ficando this.nome = nome | Escopo de variável e uso do this. O parametro tem o mesmo nome do atributo e acaba sombreando ele dentro do metodo |
| bug02 | Tentei alugar um conteudo que estava com disponivel igual a false e a API aceitou numa boa. Debitou os creditos do usuario, respondeu 200 e ainda marcou o conteudo como indisponivel de novo | No metodo alugar da classe Usuario nao tinha nenhuma verificacao de disponibilidade. A regra do contrato simplesmente nao existia no codigo, tanto que a ConteudoIndisponivelException estava criada e com handler pronto, mas nenhuma linha do projeto lancava ela | Coloquei uma guarda no comeco do alugar que lanca a ConteudoIndisponivelException quando o conteudo nao esta disponivel. Deixei como primeira verificacao porque ela so depende do conteudo, nao precisa saber nada do usuario | Excecoes customizadas e fail fast. A regra de negocio fica no model e o GlobalExceptionHandler ja traduzia essa exception para 409 |
| bug03 | Cadastrei uma serie de 5 temporadas e o aluguel saiu por 9,90 em vez de 24,50. Qualquer serie cobrava o mesmo valor, independente do numero de temporadas | Na classe Serie o metodo estava escrito como calcularPrecoAluguel(double desconto), com parametro. Como a assinatura ficou diferente da do pai, que e calcularPrecoAluguel() sem parametro, isso virou sobrecarga e nao sobrescrita. O metodo da Serie nunca era chamado e o alugar acabava usando a versao da classe Conteudo, que devolvia 9,90 fixo | Tirei o parametro e coloquei o @Override, deixando calcularPrecoAluguel() sem argumento e usando o atributo numeroTemporadas da propria classe | Sobrescrita e sobrecarga. Com o @Override o compilador nao deixa passar um metodo que nao sobrescreve nada, e foi justamente a falta dele que deixou o bug compilar sem erro |
| bug04 | Cadastrei uma serie e ela salvou sem titulo, sem categoria, sem duracao e com classificacao zero. So o numero de temporadas aparecia certo. Como a classificacao ficava zero, qualquer usuario conseguia alugar | O construtor da Serie nao chamava super(). Ele so fazia this.numeroTemporadas = numeroTemporadas, entao os quatro atributos que moram na classe mae Conteudo nunca eram preenchidos. Compilava normal porque a classe Conteudo tem um construtor vazio protected, que o Java chama sozinho quando nao existe super explicito | Adicionei a chamada super(titulo, categoria, duracaoMinutos, classificacaoEtaria, disponivel) e inclui o parametro disponivel na assinatura, igual ja era no Filme e no Documentario. Tambem ajustei a chamada new Serie no ConteudoController, que passou a mandar o serie.isDisponivel() | Heranca e construtores. A subclasse precisa chamar o construtor da superclasse para inicializar o que pertence a ela |
| bug05 | Chamei o GET /api/conteudos/{id}/preco-promocional em um filme de 9,90 e voltou 11,88, ou seja, o preco promocional saiu mais caro que o normal. Na serie o mesmo endpoint dava desconto certinho | Na classe Filme o metodo aplicarPromocao devolvia preco * 1.2, que aumenta 20 por cento em vez de descontar. O javadoc da interface Promocionavel diz que toda classe que implementa ela deve aplicar 20 por cento de desconto, e a Serie ja fazia certo com preco * 0.8 | Troquei o 1.2 por 0.8 | Interface como contrato e polimorfismo. A regra dos 20 por cento esta duplicada nas duas implementacoes, entao foi possivel uma divergir da outra. Um metodo default na propria interface Promocionavel eliminaria essa chance de erro |
| bug06 | Chamei o GET /api/conteudos/999 com um id que nao existe e a API respondeu 200 com o corpo vazio, como se tivesse dado tudo certo. O contrato diz que tem que dar erro dizendo que nao foi encontrado, nunca resposta vazia | No metodo buscarPorId do ConteudoController o orElseThrow ja lancava a ConteudoNaoEncontradoException certinha, mas tinha um try em volta com catch (Exception e) vazio, so com um comentario TODO tratar isso depois. O catch engolia a excecao e o metodo caia num return null no final | Tirei o try catch inteiro e deixei a excecao subir. O GlobalExceptionHandler ja tinha um @ExceptionHandler para essa excecao que devolve 404 com a mensagem, entao nao precisa tratar nada dentro do controller | Tratamento de excecoes. Catch vazio engole o erro e transforma falha em sucesso silencioso. Com tratamento centralizado no @RestControllerAdvice, o controller nao precisa de try catch |
| bug07 | Cadastrei conteudos com a categoria FICCAO e chamei o GET /api/conteudos/categoria/FICCAO. Voltou lista vazia com status 200, como se nao existisse nada cadastrado naquela categoria | No listarPorCategoria do ConteudoController a comparacao estava feita com ==, na linha 47. String em Java e objeto, entao == compara se as duas variaveis apontam para o mesmo objeto na memoria, nao se o texto e igual. A categoria da URL vem montada pelo Spring e a do conteudo vem montada pelo Hibernate a partir do banco, entao sao dois objetos diferentes e a comparacao dava falso sempre | Troquei por categoria.equals(c.getCategoria()), colocando a variavel da URL na frente porque ela nunca vem nula | Comparacao de objetos e o metodo equals. O == compara referencia e o equals compara conteudo. Vale lembrar que o ConteudoRepository ja tem um findByCategoria pronto, que faria esse filtro no banco em vez de varrer tudo em memoria |
| bug08 | Cadastrei um documentario e o aluguel cobrou 9,90, sendo que pelo contrato documentario custa 0,00. Como cobrava, um usuario sem creditos levava erro de creditos insuficientes em um conteudo que era pra ser de graca | A classe Documentario nao tinha nenhum calcularPrecoAluguel proprio, entao herdava o da classe mae Conteudo, que devolvia 9,90 fixo. Esse 9,90 na mae ja estava errado por natureza, porque pela tabela do contrato nenhum tipo tem preco padrao, cada um tem o seu | Transformei o calcularPrecoAluguel da Conteudo em metodo abstrato e implementei no Documentario devolvendo 0.00. O Filme e a Serie ja tinham a implementacao deles | Classe abstrata e metodo abstrato. Metodo abstrato obriga toda subclasse a implementar, entao o compilador passa a garantir que nenhum tipo de conteudo fique sem preco proprio |
| bug09 | Tentei alugar um filme de 9,90 com um usuario que tinha 100 de credito e a API recusou dizendo creditos insuficientes. Ja um usuario com 5 de credito conseguia alugar o mesmo filme. Estava exatamente ao contrario do que devia | Na classe Usuario, linha 28, o metodo temCreditosSuficientes devolvia preco >= this.creditos. Isso responde se o preco e maior ou igual ao saldo, que e o oposto do que o nome do metodo promete | Inverti o operador para preco <= this.creditos. Deixei o menor ou igual porque se o preco for exatamente o saldo o aluguel pode acontecer, os creditos ficam em zero e zero nao e negativo | Logica de comparacao e coerencia entre o nome e o comportamento do metodo. Um metodo chamado temCreditosSuficientes tem que devolver verdadeiro quando os creditos bastam |
| bug10 | Mandei um POST /api/conteudos/filme com duracaoMinutos igual a 0 e depois com -30, e nos dois casos a API respondeu 201 e salvou o conteudo no banco. O contrato diz que duracao menor ou igual a zero tem que ser recusada com erro claro e nao pode salvar | Nao existia nenhuma validacao de duracao em lugar nenhum. Nem no construtor da Conteudo, nem nos tres metodos de cadastro do ConteudoController, que so montavam o objeto e chamavam o save | Coloquei a verificacao no construtor da classe Conteudo lancando IllegalArgumentException quando a duracao e menor ou igual a zero. Fiz no construtor porque os tres cadastros passam por ele, entao a regra fica escrita uma vez so e o objeto invalido nem chega a ser criado. Tambem acrescentei um @ExceptionHandler para IllegalArgumentException no GlobalExceptionHandler devolvendo 400, senao a excecao virava 500 sem mensagem util | Fail fast e validacao no construtor. O objeto nasce valido ou nao nasce. O handler centralizado traduz a excecao em status HTTP, e de quebra arrumou o GET /api/usuarios com id inexistente, que tambem devolvia 500 |
| bug11 | Mandei um POST /api/usuarios e nao conseguia salvar. O id nunca era gerado, diferente do cadastro de conteudo que funcionava normal | Na classe Usuario o campo id tinha so a anotacao @Id, sem o @GeneratedValue. Sem ela o JPA espera que alguem entregue o id pronto, entao o id ficava nulo e o registro nao era gravado. A classe Conteudo ja tinha as duas anotacoes, era so comparar as duas entidades lado a lado | Acrescentei @GeneratedValue(strategy = GenerationType.IDENTITY) no campo id, igual ao que ja existia na Conteudo | Mapeamento JPA e chave primaria. O @Id diz qual campo e a chave e o @GeneratedValue diz quem gera o valor. Com IDENTITY quem gera e o proprio banco, de forma sequencial |
| bug12 | Tentei alugar com um usuario de 12 anos um conteudo com classificacao 18. A API respondeu 500 Internal Server Error, sem mensagem nenhuma explicando a classificacao. A mensagem bonita da excecao existia mas nunca chegava no cliente | A ClassificacaoIndicativaException era a unica das quatro excecoes do projeto que estendia Exception em vez de RuntimeException, e nao tinha nenhum @ExceptionHandler no GlobalExceptionHandler. Sem handler o Spring nao sabia traduzir ela e devolvia 500. Por ser checked ela ainda obrigava o Usuario.alugar e o AluguelController a declararem throws, sendo que o controller nao fazia nada com ela | Troquei para extends RuntimeException, igual as outras tres excecoes do projeto, e criei o @ExceptionHandler devolvendo 403 Forbidden com a mensagem. Com ela unchecked os dois throws sairam das assinaturas e o import que sobrou no AluguelController foi removido | Excecoes checked e unchecked. Checked serve para erro que quem chama consegue tratar e se recuperar. Violacao de regra de negocio em API REST nao tem recuperacao, entao o certo e unchecked e deixar o handler centralizado traduzir em status HTTP. Usei 403 porque o pedido esta correto e o conteudo existe, mas esse usuario nao pode acessar |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | ConteudoController, linhas 96 a 100. Tinha um bloco de if inteiro comentado sobre regra de cupom, com um TODO em cima dizendo para nao apagar porque podia ser util | Codigo comentado guardado no fonte. O historico do Git ja guarda tudo que foi apagado, entao manter comentado so faz o codigo apodrecer sem ninguem ver. Tanto que esse bloco chamava usuario.temCupomAtivo, um metodo que nao existe no projeto, e chamava aplicarPromocao sem objeto e sem argumento. Se alguem descomentasse hoje nem compilava | Apaguei o bloco comentado e o TODO. Se a regra de cupom voltar, ela esta no commit anterior |
| clean02 | ConteudoController, logo depois dos metodos de cadastro. Tinha um metodo privado chamado calcularDescontoAntigo, com um comentario dizendo que era codigo do prototipo antigo, mantido caso o time de marketing voltasse atras | Codigo morto. O metodo era privado e nenhuma linha do projeto chamava ele, entao existia so ocupando espaco e confundindo quem le. Ainda por cima calculava 10 por cento de desconto, uma regra que nao existe no contrato da API e que conflita com os 20 por cento do Promocionavel | Apaguei o metodo e o comentario. O historico do Git guarda a regra antiga se ela precisar voltar |
| clean03 | No metodo alugar da classe Usuario. O parametro se chamava c e a variavel do preco se chamava p | Nomes sem significado. Uma letra sozinha nao diz o que a variavel guarda, entao quem le o metodo precisa voltar na assinatura toda hora para lembrar o que e c e o que e p. Num metodo que tem tres verificacoes de regra de negocio isso atrapalha bastante | Renomeei c para conteudo e p para precoAluguel, trocando em todas as ocorrencias do metodo |
| clean04 | | | |
| clean05 | | | |
| clean06 | | | |

---

## Parte 3 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do projeto
> como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. Injeção de dependência (Aula 13)
Os controllers recebem os repositories via `@Autowired` (ex.: `ConteudoController`
usa `ConteudoRepository`). Explique por que o Spring precisa gerenciar esses objetos
em vez de criarmos com `new ConteudoRepository()`. O que exatamente o Spring faz ao
injetar um bean, e por que isso não funcionaria com um `new` comum?

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
Na Aula 12 escrevemos um `ProdutoDAO` na mão com `Connection`, `PreparedStatement` e
`ResultSet`. Aqui o `ConteudoRepository` tem 2 linhas e faz CRUD completo. Compare as
duas abordagens: o que o Spring Data JPA automatiza, o que o JDBC/DAO ainda resolve
melhor, e como o `findByCategoria` consegue funcionar sem implementação.

### 3. Exceções checked vs unchecked (Aula 11)
A `ClassificacaoIndicativaException` estourava como um erro genérico do servidor,
sem mensagem útil para o cliente. Explique a diferença entre `extends Exception` e
`extends RuntimeException` no contexto desse bug, e como você fez a mensagem da
regra (classificação indicativa) chegar de forma clara ao cliente da API.

### 4. Sobrescrita vs sobrecarga (Aula 7)
Um dos bugs compilava sem nenhum erro: o método da `Serie` parecia sobrescrever
`calcularPrecoAluguel`, mas na verdade sobrecarregava. Explique a diferença entre
override e overload nesse caso e por que a anotação `@Override` teria impedido o bug.

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
Vimos bugs de dados inválidos aceitos (duração negativa, créditos negativos, campos
nulos). Em quais lugares (construtor, setter, método do model) cada tipo de validação
deve ficar? Justifique usando os bugs que você encontrou e explique por que validar só
em um lugar não foi suficiente.

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é abstrata e `Promocionavel` é uma interface. Explique a diferença de
propósito entre as duas nesse projeto e o que mudaria no código se o Documentário
passasse a ter promoções — quais classes/linhas seriam tocadas e quais ficariam
intactas? O que isso diz sobre o design do sistema?

---

## Parte 4 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```

```
