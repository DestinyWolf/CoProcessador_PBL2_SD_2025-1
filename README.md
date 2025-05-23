# CoProcessador para o segundo problema do PBL de Sistema Digitais

<div align="center">
<h1>

 [Estrutura implementada](#estrutura-do-coprocessador) | [ISA](#conjunto-de-instruções-isa) | [Barramentos](#barramentos)
</h1>
</div>
Este coprocessador foi desenvolvido como complemento para o segundo problema de Sistemas Digitais. Tem como objetivo ser um Co-Processador para a realização de operações utilizando matrizes.
<details>
<summary><h1>Estrutura do CoProcessador</h1></summary>

# Estrutura do CoProcessador

O CoProcessador desenvolvido implementa uma arquitetura simples, sem pipeline ou qualquer forma de paralelismo, assim se faz necessario esperar uma instrução se encerrar para tentar execultar uma nova. A estrutura do CoProcessador é composta por:

- **Registradores**: São as unidades de armazenamento do CoProcessador. São utilizados 3 bancos de registradores, um para a _matriz A_, ou para _matriz B_ e outro para a _matriz C_. Cada banco de registradores conta com _25 registradores_ de _8 bits_ cada, um registrador para cada elemento.
- **ULA**: Unidade Lógica e Aritmética. É responsável por realizar as operações de soma, subtração, multiplicação por escalar e multiplicação de matrizes, O coprocessador desenvolvido trabalha com numeros inteiros de 8 bits no _complemento A2_, ou seja, o MSB _(bit mais significativo)_ é utilizado para indicar o sinal, além de outras particuliaridades que podem ser vistas [aqui](https://embarcados.com.br/complemento-de-2/). Como utilizamos essa tecnica, os numeros vão de -128 até 127 em notação decimal.
- **Unidade de controle**: É responsável por gerenciar o fluxo de dados entre os componentes do CoProcessador alem de gerenciar o envio das flags de resultado para o processador _(HPS)_.
- **Decodificador**: É responsável por decodificar as instruções recebidas do processador _(HPS)_ e enviar os bits para a unidade de controle e para o demultiplexador dos bancos de registradores das matrizes A e B.

> [!NOTE]
> Como não foi utilizada a memoria RAM para armazenar as matrizes, elas são armazenadas nos bancos de registradores, por isso é necessario enviar elemento a elemento utilizando a instrução [`STORE`](#store)

<div align="center">
  <figure>
    <img src="docs/Estrutura_CoProcessador.jpg" width="600px"/>
    <figcaption>
      <p align="center">
        <b>Figura 1</b> - Diagrama de blocos da estrutura do CoProcessador desenvolvido
      </p>
    </figcaption>
  </figure>
</div>

<details>
<summary><h2>Tempo de execução das instruções</h2></summary>

## Tempo de execução das instruções

O tempo de execução das instruções é determinado pela complexidade da operação a ser execultada, as instruções aritimeticas que envolvem o uso da ULA e o armazenamento do seu resultado na matriz C são as mais demoradas. Essas instruções necessitam de 9 pulsos de clock para serem concluidas. As instruções de leitura e escrita de dados levam apenas 3 pulsos de clock. As instruções da ula sozinhas levam apenas 6 pulsos de clock, mas como não há pipeline e é necessario armazenar o resultado após a execulção da operação, são necessario mais 3 pulsos de clock para o armazenamento do resultado.

> [!NOTE]
> Para facilitar a utilização foi inserida uma flag de [done](#barramento-de-flags) que é ativada quando a instrução é concluida, assim basta aguardar o valor dessa flag ser atualizado para 1 para saber que a instrução foi concluida.

</details>
</details>

<details>
<summary><h1>Barramentos</h1></summary>

# Barramentos

O modulo do Co processador conta com dois barramentos de entrada e dois barramentos de saida

Barramento|Tipo|Tamanho
:---------|:-------|:--------
instruction|Input|18 bits
wr|Input|1 bit
DataOut|Output|8 bits
Flags|Output|3 Bits

<details>
<summary><h3>Barramento de Instruções</h3></summary>

## Barramento de instruções

Este barramento é responsavel por enviar ao Coprocessador as instruções a serem execultadas. O barramento de instruções é de 18 bits sendo 3 deles dedicados aos [8 OP Codes](#conjunto-de-instruções-isa)
que o coprocessador possui, as intruções possuem campos e formatos diferentes, sendo assim nem todas as instruções utilizam os 18 bits.

</details>

<details>
<summary><h3>Barramento de escrita (wr)</h3></summary>

## Barramento de escrita (wr)

Este barramento serve para informa ao processador que deve escrever um dado na matriz A ou B, sendo assim ele so é utilizado nas instruções que envolvem a escrita de algum dado
na matriz A ou na matriz B, para essas intruções, o seu valor deve ser alterado pra 1.
Durante a realização de qualquer outra operação seu valor deve ser 0, caso contrario o coprocessador poderá sobrescrever algum dos valores das matrizes.

> [!NOTE]
> O sinal WR é um sinal usado para sincronismo no momento da escrita de dados no [banco de registradores](https://www2.pcs.usp.br/~labdig/pdffiles_2015/banco-registradores-ula.pdf)

> [!WARNING]
> **A cada operação de escrita deve se alterar o valor para 1 apenas apos inserir a instrução no barramento de instruçõe e seu valor deve retornar a zero apos a escrita ser realizada.**

</details>

<details>
<summary><h3>Barramento de Saida (OutputData)</h3></summary>

## Barramento de Saida (OutputData)

O barramento de saida armazena o valor do endereço solicitado da matriz C pela instrução de load até que um novo valor seja solicitado ou seja realizada alguma operação aritimetica.

</details>

<details>
<summary><h3>Barramento de Flags</h3></summary>

## Barramento de Flags

O barramento de flags é responsavel por armazenar os flags de overflow, endereçamento incorreto e finalização de uma operação. No coprocessador essas flags estão separadas em tres saidas distindas, mas podem ser associadas a um unico PIO, já que cada flag se trata de um valor unitario. Quando uma das flags for ativa seu valor logico será de 1 , caso contrario seu valor será 0.

Flag|Significado
:----|:-----------
**Overflow** | Flag ativada quando a operação realizada resulta em um valor maior que 8 bits  ou que não pode ser respresentado no [complemento A2](https://embarcados.com.br/complemento-de-2/).
**Incorrect addr**| Flag ativada quando um endereço incorreto é passado para a operações de armazenamento ou carregamento de dados.
**Done** | Flag ativada quando uma operação é finalizada.

</details>
</details>

<details>
<summary><h1>Conjunto de Instruções (ISA)</h1></summary>

# Conjunto de instruções (ISA)

O coprocessador conta com um conjunto de 8 instruções que podem ser utilizadas para realizar operações aritmeticas, de armazenamento e de leitura de dados.

> [!WARNING]
> Uma instrução não é uma função, umas instrução é uma sequencia de bits que dizem ao coprocessador o que fazer, não possui um "retorno" como a chamada de uma função. O que acontece é que ao fim da execução de uma instrução, o coprocessador podera colocar um valor no [barramento de saida](#barramento-de-saida-outputdata) e/ou atualizar os valores das [flags](#barramento-de-flags), mas não são todas as intruções que possuem uma escrita no barramento de saida ou no barramento de flags.

> [!NOTE]
> Nem todas as operações utilizão todos os 18 bits disponiveis, observar os campos de cada instrução
> para evitar perda de dados ou problemas de funcionamento.

> [!WARNING]
> Os campos das instruções de dos dados **saem do mais significativo para o menos**, ou seja, o campo do opcode da instrução começa
> no bit 0 e vai até o bit 2 e o mesmo se aplica aos demais campos de acordo com a instrução a ser utilizada.

<details>
<summary><h3>Tabela de Instruções</h3></summary>

## Tabela de Instruções

 OP Code | Nome da operação | Descrição
 :------ | :-------- |:-------
 000 | [NOP](#nop) |Informa ao coprocessador nao realizar nada, usado para bolhas.
 001 | [LOAD](#load) |carrega no barramento de saida o numero da matriz C solicitado.
 010 | [STORE](#store) |Usado para guardar numeros nas matrizes A e B.
 011 | [ADD](#add) |Usado para realizar operação de soma das matrizes A e B.
 100 | [SUB](#sub) |Usado para realizar operação de subtracao das matrizes A e B.
 101 | [MULE](#mule) |Usado para realizar operação de multiplicacao da matriz A por um escalar.
 110 | [MULM](#mulm) |Usado para realizar operação de multiplicação da matriz A pela matriz B.
 111 | [RST](#rst) |Usado para limpar os valores que estão nas Matrizes A, B e C

Descrição detalhada de cada uma das instruções com seus respectivos campos e possiveis [flags](#barramento-de-flags)

> [!NOTE]
> A unica instrução capaz de retornar um valor pelo [barramento de dados](#barramento-de-saida-outputdata) é a intrução de [LOAD](#load), todas as outras não retornam ou alteram
> o valor que esta no barramento

> [!WARNING]
> As Matrizes _A_ e _B_ devem ser armazenadas elemento a elemento, pois se tratam de 25 registradores, um para cada elemento da matriz,
> sendo assim é necessario realizar 25 vezes o envio da instrução store, cada uma contendo um dos elementos da matriz.

<details>
<summary><b>NOP instruction</b></summary>

### NOP

**Campos da instrução NOP**

Nome do Campo| Descrição | tamanho |Bit final| Bit inicial
:---------|:---------|:---------|:---------|:----------
Opcode| Codigo da operação (000) | 3 bits| 2 | 0
Não usados | | 15 bits| 17| 3

**Flags que podem ser ativadas**
> Essa instrução não tiva nenhuma flag

<div align="center">
  <figure>
    <img src="docs/NopInstruction.png" width="600px"/>
    <figcaption>
      <p align="center">
        <b>Figura 2</b> - Estrutura da instrução NOP
      </p>
    </figcaption>
  </figure>
</div>

</details>

<details>
<summary><b>LOAD instruction</b></summary>

### LOAD

**Campos da instrução LOAD**

Nome do Campo| Descrição | tamanho |Bit final| Bit inicial
:---------|:---------|:---------|:---------|:----------
Opcode| Opcode da instrução (001)| 3 bits| 2 | 0
Coluna| Coluna que esta o valor desejado| 3 bits| 5 | 3
Linha| Linha que esta o valor desejado| 3 bits| 8 | 6
Não usados| | 9 bits| 17| 9

> [!NOTE]
> Para linha e coluna os valores podem ir e 0 a 100, ou de 0 a 4, respectivamente.

- **Flags que podem ser ativadas**
  - `Incorrect Addr` Endereçamento incorreto
  - `Done` Fim da execução da instrução

<div align="center">
  <figure>
    <img src="docs/LoadInstruction.png" width="600px"/>
    <figcaption>
      <p align="center">
        <b>Figura 3</b> - Estrutura da instrução LOAD
      </p>
    </figcaption>
  </figure>
</div>
</details>

<details>
<summary><b>STORE instruction</b></summary>

### STORE

**Campos da instrução STORE**

Nome do Campo| Descrição | tamanho |Bit final| Bit inicial
:---------|:---------|:---------|:---------|:----------
Opcode| Opcode da instrução (010)|3 bits| 0 | 2
Matriz| Matriz em que o valor vai ser escrito | 1 bit | 3 | 3
Coluna| Coluna que o dado vai ser escrito de 0 a 4| 3 bits| 6 | 4
Linha| Linhas que o dado vai ser escrito| 3 bits | 9 | 7
Valor| Valor que vai ser escrito de -128 a 127 ou 0 255 se for sem sinal| 8 bits | 17 | 10

> [!NOTE]
> Para linha e coluna os valores podem ir de 0 a 100, ou de 0 a 4, respectivamente.

> [!WARNING]
> Foi utilizado o [complemento A2](https://embarcados.com.br/complemento-de-2/), ou seja, o maior valor positivos é de 127 e para os negativos 128.

- **Flags que podem ser ativadas**
  - `Incorrect Addr` Endereçamento incorreto
  - `Done` Fim da execução da instrução

<div align="center">
  <figure>
    <img src="docs/StoreInstruction.png" width="600px"/>
    <figcaption>
      <p align="center">
        <b>Figura 4</b> - Estrutura da instrução STORE
      </p>
    </figcaption>
  </figure>
</div>

</details>

<details>
<summary><b>ADD instruction</b></summary>

### ADD

**Campos da instrução ADD**

Nome do Campo| Descrição | tamanho |Bit final| Bit inicial
:---------|:---------|:---------|:---------|:----------
Opcode|Opcode da instrução (011)| 3 bits| 2|0
Não usados| | 15 bits| 17|3

- **Flags que podem ser ativadas**
  - `Overflow` o resultado da operação não pode ser devidamente representado em 8 bits utilizando C2 (complemento A2)
  - `Done` Fim da execução da instrução

<div align="center">
  <figure>
    <img src="docs/AddInstruction.png" width="600px"/>
    <figcaption>
      <p align="center">
        <b>Figura 5</b> - Estrutura da instrução ADD
      </p>
    </figcaption>
  </figure>
</div>

</details>

<details>
<summary><b>SUB instruction</b></summary>

### SUB

**Campos da instrução SUB**

Nome do Campo| Descrição | tamanho |Bit final| Bit inicial
:---------|:---------|:---------|:---------|:----------
Opcode| Opcode da instrução (100)| 3 bits | 2 | 0
Não usados| | 15 bits | 17 | 3

- **Flags que podem ser ativadas**
  - `Overflow` o resultado da operação não pode ser devidamente representado em 8 bits utilizando C2 (complemento A2)
  - `Done` Fim da execução da instrução
  
<div align="center">
  <figure>
    <img src="docs/SubInstruction.png" width="600px"/>
    <figcaption>
      <p align="center">
        <b>Figura 6</b> - Estrutura da instrução SUB
      </p>
    </figcaption>
  </figure>
</div>

</details>

<details>
<summary><b>MULE instruction</b></summary>

### MULE

**Campos da instrução MULE**

Nome do Campo| Descrição | tamanho |Bit final| Bit inicial
:---------|:---------|:---------|:---------|:----------
Opcode| Opcode da instrução (101)| 3 bits | 2 | 0
Escalar| Escalar que será usado para a realização da multiplicação| 8 bits | 10 | 3
Não usados| | 7 bits | 17 | 11

- **Flags que podem ser ativadas**
  - `Overflow` o resultado da operação não pode ser devidamente representado em 8 bits utilizando C2 (complemento A2)
  - `Done` Fim da execução da instrução

<div align="center">
  <figure>
    <img src="docs/MulEInstruction.png" width="600px"/>
    <figcaption>
      <p align="center">
        <b>Figura 7</b> - Estrutura da instrução MULE
      </p>
    </figcaption>
  </figure>
</div>

</details>

<details>
<summary><b>MULM instruction</b></summary>

### MULM

**Campos da instrução MULM**

Nome do Campo| Descrição | tamanho |Bit final| Bit inicial
:---------|:---------|:---------|:---------|:----------
Opcode| Opcode da instrução (110)| 3 bits | 2 | 0
Não usado| | 15 bits | 17 | 3

- **Flags que podem ser ativadas**
  - `Overflow` o resultado da operação não pode ser devidamente representado em 8 bits utilizando C2 (complemento A2)
  - `Done` Fim da execução da instrução

<div align="center">
  <figure>
    <img src="docs/MulMInstruction.png" width="600px"/>
    <figcaption>
      <p align="center">
        <b>Figura 8</b> - Estrutura da instrução MULM
      </p>
    </figcaption>
  </figure>
</div>

</details>

<details>
<summary><b>RST instruction</b></summary>

### RST

**Campos da instrução RST**

Nome do Campo| Descrição | tamanho |Bit final| Bit inicial
:---------|:---------|:---------|:---------|:----------
Opcode| Opcode da instrução (111)| 3 bits | 2 | 0
Não usado| | 15 bits | 17 | 3

**Flags que podem ser ativadas**
> Essa instrução não tiva nenhuma flag

<div align="center">
  <figure>
    <img src="docs/ResetInstruction.png" width="600px"/>
    <figcaption>
      <p align="center">
        <b>Figura 9</b> - Estrutura da instrução RST
      </p>
    </figcaption>
  </figure>
</div>

</details>

</details>
