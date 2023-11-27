# Semana 11

## CTF - RSA

O objetivo deste CTF era fatorizar um módulo de RSA com conhecimento partcial dos primos utilizados para o calcular e usar esse módulo para desencifrar a mensagem da flag presente no servidor.

Ao aceder ao servidor com `nc ctf-fsi.fe.up.pt 6004`, foi-nos dado o módulo (n), o expoente público (e) e o ciphertext.

``` python
e = 65537

n = 359538626972463181545861038157804946723595395788461314546860162315465351611001926265416954644815072042240227759742786715317579537628833244985694861278976928056697934064926959392075601507358866602396413033604962235604422217945253079931205430205433480758529999513833490142670671432294186322786134234472462602941

ciphertext = "6435646231366566346262656637613030633839613666366439613131323266396638623262663266633531316564336164366430643133623031343533623861303630336430376661373634303066383931323761663161343064663864616236656435666634656335646435623230306236393532643836396538663862616131663531643639646536346164393763343339333534383938316132316461306635653837643330633638363766393165333737633164363430316331656665613566666238383762643538383834363036343661376633653336636636656461303564343139396239363238323531376536363432383461363362663230303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030"
```

Precisamos de descobrir os primos p e q tal que p*q = n. Com esta informação, conseguimos calcular a chave privada, decrifrar o ciphertext e obter a flag.

No enunciado do CTF é nos dada alguma informação extra sobre os valores p e q:

- p é um primo próximo de 2^512
- q é um primo próximo de 2^513

Também é nos dito que o algoritmo Miller-Rabin é útil para testar a primalidade de números. Como tal, procurámos uma implementação desse algortimo e acabamos por nos deparar com o seguinte: https://inventwithpython.com/rabinMiller.py

Agora, é só usar utilizar a função `isPrime` que obtemos no link acima e começar a testar valores em busca de `p` e `q`. Para tal, foi escolhido um intervalo arbitrário (10 000) e iremos testar todos os primos nesse raio usando 2^512 como o ponto central. Assim, obtemos valores possíveis para `p`. Agora falta testar se, para esse `p`, existe um `q` primo tal que `p*q = n`. Para isso, apenas testamos se `n` é divisível por `p` e, se for, verificamos se o resultado da sua divisão é primo. Se for, este corresponde a `q` e damos print aos resultados.

``` python
itv = 10000
p = (2**512 + 1) - itv
while p < 2**512 + itv:
    if isPrime(p):
        q, rem = divmod(n, p)
        if rem == 0 and isPrime(q):
            print(f"p = {p}")
            print(f"q = {q}")
    p += 2 
```

Ao correr este código obtemos os valores desejados:

![image](assets/s11i1.png)

``` python
p = 13407807929942597099574024998205846127479365820592393377723561443721764030073546976801874298166903427690031858186486050853753882811946569946433649006084381
q = 26815615859885194199148049996411692254958731641184786755447122887443528060147093953603748596333806855380063716372972101707507765623893139892867298012169761
```

Usando a função de decifração que nos é providenciada neste ctf, resta só calcular d, que sabemos que é o inverso multiplicativo de `e` módulo `phi`, em que `phi = (p-1)*(q-1)`. Assim, teremos todas as informações que precisamos para obter a flag.

``` python
def dec(y, d, n):
    int_y = int.from_bytes(unhexlify(y), "little")
    x = pow(int_y,d,n)
    return x.to_bytes(256, 'little')

phi = (p-1)*(q-1)
d = pow(e, -1, phi)
flag = dec(unhexlify(ciphertext), d, n)
print(flag.decode())   
```

![image](assets/s11i2.png)