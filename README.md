# CoProcessador para o segundo problema do PBL de Sistema Digitais

Este coprocessador foi desenvolvido como complemento para o segundo problema de Sistemas Digitais. Tem como objetivo ser um Co-Processador para a realização de operações utilizando matrizes.



# entradas e saidas

O modulo do Co processador conta com dois barramentos de entrada e dois barramentos de saida

Barramento|Tipo|Tamanho
:---------|:-------|:--------
instruction|Input|18 bits
wr|Input|1 bit
DataOut|Output|8 bits
Flags|Output|3 Bits



<details>
<summary><b>Barramento de Instruções</b></summary>

### Barramento de instruções

Este barramento é responsavel por enviar ao Coprocessador as instruções a serem execultadas. O barramento de instruções é de 18 bits sendo 3 deles dedicados aos 8 OP Codes
que o coprocessador possui, as intruções possuem campos e formatos diferentes, sendo assim nem todas as instruções utilizam os 18 bits.

</details>

<details>
<summary><b>Barramento de escrita (wr)</b></summary>

### Barramento de escrita (wr)

Este barramento serve para informa ao processador que deve escrever um dado na matriz A ou B, sendo assim ele so é utilizado nas instruções que envolvem a escrita de algum dado
na matriz A ou na matriz B, para essas intruções, o seu valor deve ser alterado pra 1.
Durante a realização de qualquer outra operação seu valor deve ser 0, caso contrario o coprocessador poderá sobrescrever algum dos valores das matrizes.

> [!WARNING]
> **A cada operação de escrita deve se alterar o valor para 1 apenas apos inserir a instrução no barramento de instruçõe e seu valor deve retornar a zero apos a escrita ser realizada.**


</details>


## Conjunto de instruções

Abaixo estão listadas as instruções e seus devidos retornos

<details>
<summary><b>Tabela de Instruções</b></summary>

### Tabela de Instruções

 OP Code | Nome da operação | Descrição
 :------ | :-------- |:-------
 000 | NOP |Informa ao coprocessador nao realizar nada, usado para bolhas.
 001 | LOAD |carrega no barramento de saida o numero da matriz C solicitado.
 010 | STORE |Usado para guardar numeros nas matrizes A e B.
 011 | ADD |Usado para realizar operação de soma das matrizes A e B.
 100 | SUB |Usado para realizar operação de subtracao das matrizes A e B.
 101 | MULE |Usado para realizar operação de multiplicacao da matriz A por um escalar.
 110 | MULM |Usado para realizar operação de multiplicação da matriz A pela matriz B.

</details>
