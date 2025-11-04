# Projeto de Cadastro de Pessoas (P1nX)

Este projeto é um sistema de console (CLI) em Java para cadastro de pessoas, focado na aplicação robusta de conceitos de Programação Orientada a Objetos, validação de dados em múltiplas camadas e tratamento de exceções.

## 1. Visão Geral

O sistema permite cadastrar `Pessoa` (divididas nas subclasses `Homem` e `Mulher`). A aplicação possui duas fases de execução:

1.  **Cadastro via CLI:** A primeira pessoa é cadastrada via argumentos de linha de comando no momento da execução.
2.  **Cadastro Interativo:** Se o primeiro cadastro for bem-sucedido, o programa entra em um modo interativo (via `Scanner`) que pergunta ao usuário quantas pessoas adicionais ele deseja cadastrar.

O foco do projeto é garantir a **integridade dos dados** e a **robustez do modelo de domínio**, impedindo a criação de objetos em estado inválido através de um sistema de validação rigoroso.

---

## 2. Decisões de Design e Arquitetura

Esta seção detalha as escolhas de design e os padrões de OO aplicados no projeto.

### 2.1. Herança e Polimorfismo
O núcleo do modelo de domínio usa Herança para representar a relação "é um".

* **`Pessoa.java` (Classe Base):** Define os atributos e comportamentos comuns a todas as pessoas (nome, sobrenome, dataNasc, cpf, peso, altura).
* **`Homem.java` e `Mulher.java` (Subclasses):** Herdam de `Pessoa` e especializam o comportamento, definindo o atributo `SEXO` como uma constante (`static final`).

O **Polimorfismo** é aplicado na classe `P1nX.java`, onde um array do tipo `Pessoa[]` é usado para armazenar objetos dos tipos `Homem` e `Mulher` de forma intercambiável.

### 2.2. Encapsulamento e Integridade do Objeto
A principal diretriz do projeto é que **um objeto `Pessoa` nunca deve existir em um estado inválido**.

Isso é alcançado movendo a lógica de validação para dentro dos métodos *setters* da classe `Pessoa`.

* **Setters como Guardiões:** Métodos como `setNome()`, `setDataNasc()`, `setNumCPF()`, etc., não apenas atribuem valores, mas primeiro validam a entrada.
* **Falha Rápida (Fail-Fast):** Se uma validação falhar, o *setter* lança uma exceção customizada (ex: `InvalidCpfException`). Isso impede que um dado inválido seja atribuído ao atributo do objeto.
* **Construtores:** Os construtores da classe `Pessoa` utilizam esses *setters* para inicializar os atributos, garantindo que a validação seja executada no momento da criação do objeto.

### 2.3. Classes Utilitárias (Utility Pattern)
Para lógicas de validação complexas e puras (que não dependem do estado de um objeto), foram criadas classes utilitárias:

* **Classes:** `ValidaCPF.java` e `ValidaData.java`.
* **Design:**
    * Declaradas como `final` para impedir herança.
    * Possuem um construtor `private` para impedir instanciação (`new ValidaCPF()`).
    * Todos os seus métodos são `static`, sendo acessados diretamente (ex: `ValidaCPF.isCPF(...)`).

Isso centraliza a lógica de validação complexa em um local, tornando-a reutilizável e fácil de manter (Princípio da Responsabilidade Única - SRP).

### 2.4. Tratamento de Exceções Customizadas
O projeto utiliza exceções checadas e não checadas (Runtime) customizadas (ex: `InvalidDataException`, `InvalidNameException`) em vez de exceções genéricas (como `IllegalArgumentException`).

* **Benefício:** Permite um tratamento de erro (`try-catch`) muito mais granular e específico na classe `P1nX`. O programa pode identificar *exatamente* qual validação falhou e fornecer feedback preciso ao usuário.

### 2.5. Sobrecarga de Construtor (Overloading)
As classes `Pessoa`, `Homem` e `Mulher` possuem construtores sobrecarregados:

1.  **Construtor Básico (6 argumentos):** `genero`, `nome`, `sobrenome`, `dia`, `mes`, `ano`.
2.  **Construtor Especializado (9 argumentos):** Inclui `CPF`, `peso` e `altura`.

Isso oferece flexibilidade para a aplicação cliente (`P1nX`), que pode criar objetos `Pessoa` com diferentes níveis de detalhe, dependendo dos dados fornecidos (6 ou 9 argumentos na CLI).

---

## 3. Regras de Negócio e Validações

O sistema implementa um conjunto rigoroso de regras de validação:

| Classe | Método / Regra | Detalhe da Validação |
| :--- | :--- | :--- |
| **`ValidaCPF`** | `isCPF()` | 1. **Formato (Regex):** Aceita `11122233344`, `111.222.333-44` ou `111.222.333/44`. 2. **Dígitos Repetidos:** Rejeita CPFs com todos os dígitos iguais (ex: `111.111.111-11`). 3. **Dígitos Verificadores:** Calcula os 10º e 11º dígitos usando o algoritmo padrão (módulo 11) e os compara com os dígitos fornecidos. |
| **`ValidaData`** | `isData()` | 1. **Limites Individuais:** `dia` (1-31), `mes` (1-12), `ano` (Ano Atual - 120 anos). 2. **Suporte a Mês Extenso:** O `Mes.java` (Enum) é usado para converter `String` (ex: "Janeiro") para `int` (1). 3. **Ano Bissexto:** Verifica corretamente se o ano é bissexto para validar o dia 29 de Fevereiro. 4. **Dias do Mês:** Valida se o dia é compatível com o mês (ex: rejeita 31 de Abril). 5. **Data Futura:** Rejeita datas posteriores à data atual (`LocalDate.now()`). |
| **`Pessoa`** | `setNome()` / `setSobreNome()` | Rejeita `null`, strings vazias ou strings que não contenham apenas letras e espaços (Regex: `^[\p{L}\s]+$`). |
| **`Pessoa`** | `setPeso()` | Rejeita valores menores ou iguais a 0. |
| **`Pessoa`** | `setAltura()` | Rejeita valores menores que 0.40m (regra de negócio arbitrária). |

---

## 4. Fluxo de Execução (`P1nX.java`)

A classe `P1nX` orquestra a aplicação:

1.  **Inicialização:** O método `main` é invocado.
2.  **Verificação de Argumentos (CLI):** Verifica se `args.length` é 6 ou 9. Se não for, exibe `displayHelp()` e encerra.
3.  **Cadastro CLI:**
    * Chama `trataExcecaoCLI(args)`.
    * Este método tenta criar a `Pessoa` (`new Homem(...)` ou `new Mulher(...)`).
    * Se qualquer exceção de validação for capturada (ex: `InvalidCpfException`), ela é tratada, uma mensagem de erro específica é exibida, `displayHelp()` é chamado e o programa **encerra** (`return`).
4.  **Início do Modo Interativo:**
    * Se o cadastro CLI for bem-sucedido, o programa pergunta quantas pessoas mais cadastrar (`quantidadePessoas()`).
5.  **Loop de Cadastro Interativo:**
    * O método `capturaEntrada()` entra em um loop, pedindo cada dado (gênero, nome, etc.) via `Scanner`.
    * Chama `trataExcecaoLoop(...)` para criar a `Pessoa`.
    * **Diferença Crucial:** Se `trataExcecaoLoop` capturar uma exceção, ele apenas exibe o erro e **continua o loop** (`continue`), pedindo os dados novamente, sem abortar o programa.
6.  **Encerramento:** Ao final dos loops, `exibeInfos()` é chamado para imprimir os detalhes de todas as pessoas cadastradas no array e usa `instanceof` para contar o total de `Homem` e `Mulher`.

---

## 5. Como Executar

O programa deve ser compilado e executado a partir da classe `P1nX`, estando no diretório raiz do pacote `br`.

### 1. Cadastro Básico (CLI - 6 argumentos)

```bash
# Formato: java br.uerj.P1n.P1nX      
java br.uerj.P1n.P1nX Masculino Alexander Dungan 23 7 1993
```

### 2. Cadastro Especializado (CLI - 9 argumentos)

```bash
# Formato: ...   
java br.uerj.P1n.P1nX Feminino Maria Silva 10 Maio 1990 123.456.789-00 65.5 1.68
```

### 3. Formatos de Entrada Flexíveis

* **Gênero:** `Masculino`, `M`, `Feminino`, `F` (ignora maiúsculas/minúsculas).
* **Mês:** `5`, `05`, `Maio`, `janeiro`.
* **CPF:** `12345678900`, `123.456.789-00`, `123.456.789/00`.

---

## 6. Possíveis Melhorias Futuras

* **Testes Unitários (JUnit):** Criar testes de unidade específicos para `ValidaCPF` e `ValidaData` para garantir que todos os cenários de validação (corretos e incorretos) estão sendo cobertos.
* **Persistência de Dados:** Adicionar funcionalidade para salvar o array de `Pessoa` em um arquivo (como JSON ou XML) ao fechar o programa e carregá-lo ao iniciar.
* **Interface Gráfica (GUI):** Substituir a interface de console por uma interface gráfica usando JavaFX ou Swing.
