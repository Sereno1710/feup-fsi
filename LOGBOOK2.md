
# Trabalho realizado nas Semanas #2 e #3

## Identificação

- CVE-2022-0847 (também chamado de Dirty Pipe) é uma vulnerabilidade que existe no kernel do Linux desde a versão 5.8.
- Esta vulnerabilidade permite a qualquer utilizador sobrescrever qualquer ficheiro "read-only", mesmo sem este possuir os privilégios necessários.
- Foi corrigida no Linux 5.16.11, 5.15.25 e 5.10.102.

## Catalogação
- Foi descoberto por Max Kellerman, após analisar tickets de suporte acerca de corrupção de ficheiros.
- A 20/02/2022, foi enviado o bug report à Linux Kernel Security Team, assim como o seu patch.
- Após reprodução do bug, a equipa lançou o patch de Kellerman em versões estáveis do Linux.
- Nível de gravidade: Alto. Este bug é fácil de explorar, já que o atacante precisa apenas de permissões de leitura.

## Exploit

- O exploit consiste em utilizar a vulnerabilidade para executar ações que o utilizador atual não deveria ter privilégios para executar.
- O Metasploit oferece uma automatização que sobrescreve binários SUID com código malicioso, executa-o, e, depois, retorna ao ficheiro original.
- 1º limitação: código malicioso não pode começar no limite de uma página (necessita um byte antes com address da pagina).
- 2º limitação: código malicioso não pode cruzar nenhum limite de página: tem que ser menor que uma página (4096 bytes).

## Ataques

- Não existe relatos de ataques bem sucedidos que utilizaram esta vulnerabilidade mas, ainda assim, possui alto potencial para causar danos.
- Um exploit pode permitir a atacantes obter privilégios no sistema afetado. Isto pode lhes oferecer controlo total sobre o mesmo.
- De modo a mitigar estes danos, é aconselhável atualizar o linux kernel para a versão mais recente, caso esteja desatualizado.
