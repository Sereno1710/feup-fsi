
# Trabalho realizado nas Semanas #2 e #3

## Identificação

- CVE-2022-0847 (também chamado de Dirty Pipe) é uma vulnerabilidade no kernel do Linux desde a versão 5.8.
- Esta vulnerabilidade permite sobrescrever informação de qualquer ficheiro de leitura por parte de qualquer utilizador, mesmo sem os privilégios necessários.
- Foi corrigida no Linux 5.16.11, 5.15.25 e 5.10.102.

## Catalogação
- Foi descoberta devido a tickets de suporte sobre corrupção de ficheiros, tendo demorado mais de um ano a identificar um padrão.
- A 20/02/2022, Max Kellerman, investigador de segurança, enviou o bug report, exploit e patch à Linux Kernel Security Team.
- Após reprodução do bug, a equipa lançou o bug fix de Max Kellerman em versões estáveis do Linux.
- Nível de gravidade: Alto. Este bug é fácil de explorar, já que o atacante precisa apenas de permissões de leitura.

## Exploit

- O exploit consiste em utilizar a vulnerabilidade para executar ações que o utilizador atual não deveria ter privilégios para executar.
- O Metasploit oferece uma automatização que sobrescreve binários SUID com código malicioso, executa-o, e, depois, retorna ao ficheiro original.
- 1º limitação: código malicioso não pode começar no limite de uma página (necessita um byte antes com address da pagina).
- 2º limitação: código malicioso não pode cruzar nenhum limite de página: tem que ser menor que uma página (4096 bytes).

## Ataques

- Não existe relatos de ataques bem sucedidos que utilizaram esta vulnerabilidade mas, ainda assim, possui alto potencial para causar danos.
- Um exploit pode permitir a atacantes obter privilégios no sistema afetado. Isto pode lhes oferecer controlo total sobre esse sistema.
- De modo a mitigar estes danos, é aconselhável atualizar o linux kernel para a versão mais recente, caso esteja desatualizado.
