# Checkpoint 5 — Bug Hunt PetFiap

> Copie este arquivo para a raiz do seu repositório com o nome **README.md**
> e preencha todas as seções.

## Identificação

**Grupo:** 35

| Integrante                        | RM     | Turma |
| --------------------------------- | ------ | ----- |
| Thiago Ono Sakai                  | 563448 | 2CCPO |
| Gabriel Nacarelli                 | 565298 | 2CCPO |
| Luiz Claro Lima                   | 563014 | 2CCPO |
| Lucas Franco de Godoy Fortes      | 561723 | 2CCPO |
| Rafael Silva Oliveira Nascimento  | 565415 | 2CCPO |
| Pedro Mitsuo Risardi Nisiaymamoto | 561710 | 2CCPO |
| Pedro Noronha dos Santos          | 564572 | 2CCPO |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |
| **Total de testes novos escritos** | 6 / 6 |
| **Suíte final (Run As → JUnit Test)** | 26 testes, 0 falhas |

---

## Parte 1 — Bugs encontrados

> Uma linha por bug, na ordem em que você os encontrou. Use a numeração dos seus
> commits (`fix: bug01 ...`). Preencha TODAS as colunas — metade da nota está aqui.

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | `AtendimentoBuilderTest.deveMontarAtendimentoCompleto` falhou com `expected: <Rex> but was: <null>`. O porte e o tutor chegavam; o nome do pet não. | `AtendimentoBuilder.comPet` (por volta da linha 24) fazia `petNome = petNome`. O parâmetro sombreava o campo, então a atribuição não gravava `this.petNome`. | Passei a atribuir `this.petNome = petNome`. | Encapsulamento e escopo: `this` distingue o atributo do parâmetro homônimo (Aula 7). |
| bug02 | `deveRecusarMontagemSemNomeDoPet` e `deveRecusarMontagemSemPorte` esperavam `IllegalArgumentException` e nada era lançado. O `construir()` devolvia um atendimento inválido. | `AtendimentoBuilder.construir` (por volta da linha 41) não validava nada. O comentário ainda jogava essa regra para o controller. | `construir()` lança `IllegalArgumentException` quando nome ou porte estão ausentes (nulos ou em branco). O objeto só nasce válido. | Validação no ponto de criação do objeto e exceção unchecked para pré-condição violada (Aulas 11 e 14, Builder). |
| bug03 | `deveCriarTosaQuandoTipoForTosa` falhou com `expected: <Tosa> but was: <Banho>`. | `AtendimentoFactory.criar`, no `case "TOSA"` (por volta da linha 17), instanciava `new Banho(...)`. | O case `"TOSA"` passou a instanciar `new Tosa(...)`. | Factory Method: só a factory conhece a subclasse concreta (Aula 14). Polimorfismo depende de nascer o tipo certo. |
| bug04 | `devePreencherOsDadosDoPetNaConsulta` falhou com `expected: <Mimi> but was: <null>` (porte e tutor também nulos). | O construtor de `ConsultaVeterinaria` (por volta da linha 17) chamava `super()` vazio e descartava protocolo, pet, porte, tutor e data. O status nem virava `AGENDADO`. | O construtor chama `super(protocolo, petNome, petPorte, tutorNome, dataHora)`. | Herança e encadeamento de construtores: a subclasse precisa inicializar o estado definido na superclasse (Aula 7). |
| bug05 | `deveManterUmaUnicaInstancia` comparou dois objetos diferentes (`assertSame`). `deveGerarProtocolosSequenciais` recebeu `1` de novo onde esperava `2` (`expected: <2> but was: <1>`). O console imprimiu "GeradorProtocolo criado!" a cada chamada. | `GeradorProtocolo.getInstancia` (por volta da linha 18) fazia `return new GeradorProtocolo()` sem guardar em `instancia`. Cada chamada zerava o contador. | A instância é atribuída ao campo estático e o método é `synchronized`, como o próprio comentário da classe promete para uso concorrente. | Singleton: uma instância e um contador global (Aula 14). Sem a atribuição, o padrão não existe. |
| bug06 | `deveRecusarAgendamentoComHorarioJaOcupado` esperava `HorarioOcupadoException` e recebeu `NullPointerException` em `salvo.getProtocolo()`, porque o `save` do mock devolveu null. O conflito não foi visto. | `AgendaService.agendar` (por volta da linha 23) comparava nome e `LocalDateTime` com `==`. O teste monta o mesmo horário com `LocalDateTime.parse`, outro objeto. A referência é diferente, então o `if` era falso e o fluxo seguia para o `save`. | As duas comparações passaram a usar `.equals()`. | `==` compara referência; `.equals()` compara valor (Aula 7). Literal `"Rex"` pode funcionar por acaso por causa do pool de strings; `LocalDateTime` criado por parse não. |
| bug07 | `deveLancarExcecaoQuandoAtendimentoNaoExiste` esperava `AtendimentoNaoEncontradoException` e o método terminava normal, devolvendo null. | `AgendaService.buscarPorId` (por volta da linha 37) tinha `orElseThrow` correto, mas um `catch (Exception)` engolia a exceção e dava `return null`. | Removi o try/catch. O `orElseThrow` propaga `AtendimentoNaoEncontradoException`. | Exceção unchecked não pode ser engolida (Aula 11). `Optional.orElseThrow` é o contrato de "nunca retornar null". |
| bug08 | Escrevi `BanhoTest.deveCalcularPrecoConformeOPorteQuandoForBanho`. Ficou vermelho: porte `PEQUENO` custava `100.0` em vez de `60.0`. | `Banho.calcularPreco` (por volta da linha 27) invertia os extremos: `PEQUENO` retornava 100 e o `else` (grande) retornava 60. `MEDIO` já estava em 80. | `PEQUENO` retorna 60, `MEDIO` continua 80 e o restante (grande) retorna 100. | Regra de negócio no model, via polimorfismo (Aula 14). O preço não pode ficar invertido entre os portes. |
| bug09 | Escrevi `TosaTest.deveDurar60MinutosQuandoForTosa`. `getDuracaoMinutos()` devolveu `30`, o padrão da superclasse. O código compilava. | `Tosa` declarava `getDuracaoMinutos(String porte)` (por volta da linha 40). Isso é sobrecarga, não sobrescrita. A chamada sem argumento caía em `Atendimento.getDuracaoMinutos()`. | O método ficou `getDuracaoMinutos()` com `@Override` e retorna 60. | Sobrescrita x sobrecarga (Aula 7). `@Override` faria o código com a assinatura errada nem compilar. |
| bug10 | Escrevi `AgendaServiceTest.deveRecusarCancelamentoQuandoAtendimentoJaEstiverConcluido`. O status ia para `CANCELADO` e o `save` era chamado. Nenhum `StatusInvalidoException`. | `Atendimento.cancelar` (por volta da linha 63) atribuía `status = "CANCELADO"` sem olhar o status atual. Concluído e já cancelado também eram aceitos. | Só cancela se o status for `AGENDADO`. Qualquer outro estado lança `StatusInvalidoException` e o service não chega no `save`. | Máquina de estados no model e exceção de domínio (Aula 11). A mesma guarda de `concluir()`. |
| bug11 | Escrevi `AgendaServiceTest.deveRecusarAgendamentoQuandoDataHoraEstiverNoPassado`. Não saía `IllegalArgumentException`, e o repositório era consultado. | `AgendaService.agendar` não tinha regra de passado. Partia direto para `findByPetNome`. | Se `dataHora` é anterior a `LocalDateTime.now()`, lanço `IllegalArgumentException` antes de qualquer chamada ao repositório. | Pré-condição de agendamento no service, com falha rápida, sem I/O (Aulas 11 e 15). |
| bug12 | Nenhum teste de unidade quebrou. Lendo `Atendimento` como review, o `@Id` não tinha geração. No `POST`, o `save` real receberia id nulo e o JPA recusaria a inclusão ("ids must be manually assigned"). O mock esconde isso. | `Atendimento` (por volta da linha 14): `@Id private Long id` sem `@GeneratedValue`. O controller também não preenche o id; o protocolo é outro número, o do singleton. | `@GeneratedValue(strategy = GenerationType.IDENTITY)` no id. O Hibernate passa a gerar a chave na tabela `atendimentos`. | Mapeamento JPA da chave primária (Aulas 12 e 13). Protocolo de negócio e id de persistência são coisas diferentes. |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `AtendimentoFactory.criar(int p, String t, String n, String po, String tu, LocalDateTime d)` | Nomes significativos. Parâmetro de uma letra obriga o leitor a decifrar a posição. | Renomeei para `protocolo`, `tipo`, `petNome`, `petPorte`, `tutorNome` e `dataHora`. O comportamento do switch não mudou. |
| clean02 | Construtor de `GeradorProtocolo`, `System.out.println("GeradorProtocolo criado!")` | Efeito colateral. Classe de domínio não escreve no console; o log vazou até na suíte. | Removi o `println`. O construtor só zera o contador. |
| clean03 | `AgendaService.agendar`, `System.out.println("Recibo: ...")` | Responsabilidade única. A regra de agenda não é camada de apresentação. O recibo no stdout aparecia no teste que agendava o Rex. | O método apenas devolve `repository.save(novo)`. |
| clean04 | `AtendimentoController.calcularDescontoFidelidade` e o bloco "Fidelidade (futuro)" | Código morto / YAGNI. Método privado sem uso, e a fórmula (`pontos * 0.1`) nem implementava "10% acima de 500 pontos". | Apaguei o método e o comentário. A feature não faz parte do contrato. |
| clean05 | `ResponseEntity.status(201)` e `status(409)` no controller | Número mágico. O restante da classe já usava `ok()`, `badRequest()` e `notFound()`. | `HttpStatus.CREATED` e `HttpStatus.CONFLICT`. O status HTTP continua 201 e 409. |
| clean06 | `@Autowired` em campo em `AgendaService` e `AtendimentoController` | Injeção por campo esconde a dependência e impede `final`. O construtor deixa o contrato da classe explícito. | Construtor com `@Autowired` e dependência `final`. O Mockito continua injetando o mock pelo construtor no `@InjectMocks`. |

## Parte 3 — Testes novos (regras que estavam sem cobertura)

> Uma linha por teste novo (`test: ...`). "Regra coberta" é o comportamento do
> contrato (seção 3 do enunciado) que o teste protege. Em "Resultado", diga se o
> teste ficou vermelho ao ser escrito (revelou bug — qual?) ou verde de cara
> (regra já estava correta).

| # | Teste escrito (classe.método) | Regra coberta | Resultado ao escrever (vermelho/verde) |
|---|---|---|---|
| teste01 | `BanhoTest.deveCalcularPrecoConformeOPorteQuandoForBanho` | Banho custa 60 / 80 / 100 conforme o porte PEQUENO, MEDIO e GRANDE. | Vermelho. PEQUENO vinha 100. Revelou o bug08. |
| teste02 | `TosaTest.deveDurar60MinutosQuandoForTosa` | Tosa dura 60 minutos. | Vermelho. A chamada sem argumento devolvia 30. Revelou o bug09. |
| teste03 | `AgendaServiceTest.deveRecusarCancelamentoQuandoAtendimentoJaEstiverConcluido` | `cancelar()` em atendimento CONCLUIDO recusa. Nada é salvo. | Vermelho. O status virava CANCELADO. Revelou o bug10. A mesma correção também recusa quem já está CANCELADO. |
| teste04 | `AgendaServiceTest.deveRecusarAgendamentoQuandoDataHoraEstiverNoPassado` | Data/hora no passado lança `IllegalArgumentException` e o banco nem é consultado. | Vermelho. Não havia a checagem. Revelou o bug11. |
| teste05 | `ConsultaVeterinariaTest.deveCobrar150ReaisQuandoPorteNaoInfluenciarOPreco` | Consulta custa 150 fixos; o porte não muda o preço. | Verde de cara. `calcularPreco()` já retornava 150.0 para qualquer porte. |
| teste06 | `AgendaServiceTest.deveCancelarQuandoAtendimentoEstiverAgendado` | `cancelar()` em AGENDADO muda o status para CANCELADO e persiste. | Verde de cara. O método já atribuía CANCELADO; faltava a trava dos outros estados, coberta pelo teste03. |

---

## Parte 4 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do
> projeto como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. A suíte como contrato (Aula 15)
O projeto chegou com 20 testes, 9 vermelhos. Descreva como você usou as
mensagens de falha (ex.: `expected: <Rex> but was: <null>`) para caçar os bugs.
O que a suíte de testes tem de melhor do que testar tudo na mão com curl?

A mensagem já entregava o sintoma e o método. Em `deveMontarAtendimentoCompleto`, `expected: <Rex> but was: <null>` na linha do `getPetNome()` me levou ao `comPet`: porte e tutor eram preenchidos, então o buraco era só o nome. Lá estava `petNome = petNome`. Em `deveCriarTosaQuandoTipoForTosa`, `expected: <Tosa> but was: <Banho>` apontou o `case "TOSA"` da factory sem eu precisar chutar qual `if` estava errado. `expected: <2> but was: <1>` no protocolo, junto com cinco linhas "GeradorProtocolo criado!" no console do Maven, mostrou que cada `getInstancia()` nascia de novo e o contador zerava. `deveLancarExcecaoQuandoAtendimentoNaoExiste` disse o contrário do que eu esperaria de um null silencioso: "Expected AtendimentoNaoEncontradoException to be thrown, but nothing was thrown". Fui no `buscarPorId` e achei o `catch` que devolvia null. Curl na API exigiria Oracle, subiria o Spring e misturaria controller, service e banco na mesma resposta HTTP. A suíte isola a classe, roda em segundos e o assert diz o valor esperado. Quando o teste está certo, falha vermelha é bug de produção, não dúvida de como eu cliquei.

### 2. Mock e injeção de dependência (Aulas 13 a 15)
No `AgendaServiceTest`, o `@Mock` cria um `AtendimentoRepository` falso e o
`@InjectMocks` o injeta no service. Explique a relação disso com o `@Autowired`
que o Spring faz em produção — quem "injeta" em cada mundo, e por que o teste
consegue rodar sem banco e sem subir o Spring?

Em produção, `AgendaService` é um `@Service`. Quem cria o objeto é o container do Spring, e o `@Autowired` no construtor pede um `AtendimentoRepository`. O Spring entrega o proxy do Spring Data, que abre conexão com o Oracle e traduz `findByPetNome` em SQL. No teste, não existe container. `@ExtendWith(MockitoExtension.class)` sobe só o Mockito. `@Mock` fabrica um repositório que não implementa JDBC nenhum, e `@InjectMocks` chama o construtor de `AgendaService` passando esse falso — o mesmo encaixe que o `@Autowired` faria, só que o "container" aqui é o Mockito. Por isso `mvn test` não lê `application.properties` e não precisa de `SEU_RM`. Em `deveRecusarAgendamentoComHorarioJaOcupado`, `when(repository.findByPetNome("Rex")).thenReturn(List.of(existente))` decide o retorno. Se o service chama `save`, `verify(repository, never()).save(any())` acusa. O teste prova a regra de conflito, não o Oracle.

### 3. `==` vs `.equals()` (Aula 7)
Um dos bugs fazia o agendamento duplicado passar pela verificação de conflito.
Explique por que `==` entre Strings e `LocalDateTime` falhou aqui, por que ele
"funciona por sorte" com literais como `"Rex"`, e o que a sua correção mudou.

O `if` de `AgendaService.agendar` era `a.getPetNome() == novo.getPetNome() && a.getDataHora() == novo.getDataHora()`. `==` em objeto compara se é o mesmo endereço de memória, não se o conteúdo é igual. O teste cria o segundo horário com `LocalDateTime.parse(existente.getDataHora().toString())`: os campos são os mesmos, o objeto é outro, então `==` dava false, o conflito era ignorado e o fluxo caía no `save`. O mock não tinha `when` para `save`, devolvia null, e o `println` do recibo estourava `NullPointerException` em `salvo.getProtocolo()` — a falha que o Maven mostrou no lugar de `HorarioOcupadoException`. Com a String `"Rex"`, os dois construtores usam o mesmo literal. O Java guarda esse literal no pool e as duas referências podem ser a mesma, então `==` "passa" sem o código estar certo. Uma String vinda de request, concatenação ou banco já não está no pool e o `==` mente. Troquei as duas comparações para `.equals()`. Aí o horário parseado é igual ao horário guardado, a exceção sai e o `save` não é chamado.

### 4. Sobrescrita vs sobrecarga (Aula 7)
Um dos bugs compilava sem nenhum erro: um método parecia sobrescrever
`getDuracaoMinutos`, mas na verdade criava uma assinatura nova. Explique a
diferença entre override e overload nesse caso e por que a anotação `@Override`
teria impedido o bug.

`Atendimento.getDuracaoMinutos()` não recebe parâmetro e devolve 30. Esse 30 é o certo para a consulta, que não sobrescreve o método. `Banho` declara `getDuracaoMinutos()` com a mesma assinatura e `@Override`, então uma variável `Atendimento` que aponta para um `Banho` executa 45. `Tosa` tinha `getDuracaoMinutos(String porte)`. O nome é igual, a lista de parâmetros não. Isso é sobrecarga: um método novo, que convive com o da superclasse. O compilador aceita porque não prometi substituir nada. Quem chama `getDuracaoMinutos()` sem argumento — o teste e o endpoint de resumo — continua no método do `Atendimento` e recebe 30, não 60. O parâmetro `porte` nem era usado. Sobrescrita exige a mesma assinatura e, em tempo de execução, o método da subclasse no lugar do da pai (polimorfismo). Se o método da tosa tivesse `@Override` com o `String porte`, a compilação falharia com "method does not override or implement a method from a supertype". A anotação transforma o deslize em erro de build. A correção foi apagar o parâmetro, manter `@Override` e retornar 60.

### 5. Singleton manual vs bean do Spring (Aula 14)
O `GeradorProtocolo` é um Singleton escrito à mão e causou um dos bugs.
Explique o que ele garante, qual foi o bug, e por que o `AgendaService`
(`@Service`) não corre o mesmo risco no container do Spring.

O singleton manual garante uma única instância e, neste projeto, um contador único de protocolo: o primeiro `proximo()` devolve 1, o seguinte 2, e assim por diante, não importa quantas vezes o controller peça `getInstancia()`. O bug estava em `getInstancia`: no `if (instancia == null)` o código dava `return new GeradorProtocolo()` e não fazia `instancia = ...`. O campo estático seguia null, cada chamada construía outro objeto com `contador = 0`, e o `assertSame` via dois objetos. O `synchronized` entrou junto porque o comentário da classe fala em uso concorrente: sem o lock, duas threads podiam passar juntas pelo `if` e criar duas instâncias mesmo depois da atribuição. `AgendaService` não tem `getInstancia`. O `@Service` pede ao Spring uma instância, e o escopo padrão do container já é singleton: o mesmo bean é injetado em todo mundo que depender dele. Quem guarda a referência é o container, não um `if` escrito por nós. O risco de "esqueci de atribuir o campo estático" não existe nesse caminho. O gerador de protocolo continuou manual porque a numeração precisa sobreviver entre requests como estado da aplicação, e o contrato pede o singleton da Aula 14, não um `@Component`.

### 6. Cobertura de testes: onde parar? (Aula 15)
Dos 6 testes novos que você escreveu, alguns ficaram vermelhos (revelaram
bugs) e outros verdes de cara (regras já corretas). Vale a pena manter os que
ficaram verdes? Em um projeto real com prazo, o que você priorizaria testar:
caminho feliz, caminhos de erro, ou 100% de cobertura? Justifique.

Quatro testes nasceram vermelhos e acharam bug de verdade: preço do banho invertido, duração da tosa caindo no método da superclasse, cancelamento de consulta já concluída e agendamento no passado. Dois nasceram verdes: consulta a 150 para qualquer porte, e cancelar um atendimento que ainda está `AGENDADO`. Eu manteria os verdes. O teste da consulta trava a regra "porte não muda o preço"; se alguém amanhã copiar o `if` do banho para dentro de `ConsultaVeterinaria.calcularPreco`, a suíte quebra na hora. O de cancelar o agendado trava o caminho feliz depois que o bug10 passou a recusar os outros estados — sem ele, uma trava escrita errada (`if ("CONCLUIDO".equals(status))`) poderia fazer o teste03 continuar verde e o cancelamento normal quebrar. Com prazo, eu não perseguiria 100% de linha. Getter, setter e o `switch` do tipo `BANHO`, que já tem teste, rendem pouco. Eu priorizaria regra que mexe em dinheiro, status e o que o sistema deve recusar: foi exatamente aí que o legado mentia com o projeto compilando. Caminho feliz sozinho não teria pego o `catch` que devolvia null nem o `==` do horário. Os testes verdes que protegem uma regra do contrato custam pouco e ficam; cobertura de vaidade em código sem decisão, não.

---

## Parte 5 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```
A identificação do grupo (nomes, RM e turma) ficou em branco de propósito:
esses dados não estão no repositório e não devem ser inventados. Falta preencher
a tabela do topo antes de entregar.

Os 20 testes originais não foram editados. Os 6 novos seguem o mesmo estilo
(AAA, deve...Quando..., Mockito só no service). A suíte final que o Maven
rodou daqui tem 26 testes e 0 falhas.

O bug12 não aparece em teste de unidade: o repositório é mock e o save nem
chega no JPA. Só aparece lendo o @Id. A API não foi subida contra o Oracle
da FIAP porque as credenciais ainda são SEU_RM / SUA_SENHA.

Depois da revisão, fechei três furos que a suíte não vê: proximo() passou a
ser synchronized; o controller manda protocolo 0 e o AgendaService só chama
o singleton depois que data, porte e horário passam; agendar inteiro roda
dentro de um lock, para duas requests não gravarem o mesmo horário. Porte
fora de PEQUENO, MEDIO e GRANDE e data ausente agora lançam
IllegalArgumentException. A herança JPA foi marcada como SINGLE_TABLE.
```
