Instituto Superior Técnico, Universidade de Lisboa

**Network and Computer Security**

# Lab guide: Java Cryptographic Mechanisms

## Objectivo

O objectivo do guia consiste em aprofundar os conhecimentos sobre criptografia através do uso
dos comandos de linha fornecidos pelo pacote OpenSSL. 
O OpenSSL é, como o nome sugere, uma implementação aberta do protocolo SSL/TLS. 
No entanto, oferece também uma biblioteca para programação e comandos de linha que permitem utilizar diversos algoritmos criptográficos.


## Exercício	1	– Criptografia	simétrica

O primeiro exercício consiste em cifrar e decifrar um ficheiro usando criptografia simétrica.

1. Todas as operações que vamos realizar usam o comando de linha _openssl_. Comece por ver
    que está disponível o manual desse comando correndo _man openssl_. Pode também aceder
    aos manuais dos subcomandos usando _man,_ p.ex., _man enc_ para o subcomando _openssl_
    _enc._

2. Observe os algoritmos de criptografia simétrica disponíveis executando
    _openssl list-cipher-commands_
    Pode observar variações de algoritmos como AES (o standard americano actual), DES, RC2,
    RC4, Blowfish (bf), Camellia. Note que nem todos os resultados são realmente algoritmos
    criptográficos, p.ex., _base64_ é uma forma de codificação, não um algoritmo criptográfico
    (obtenha informação na wikipedia caso não saiba para que serve a base 64).

3. Crie um ficheiro com o nome _texto.txt_ e 10 linhas todas com o conteúdo _Texto em claro!_

4. Vamos cifrar esse ficheiro como uma cifra de bloco, AES, com uma chave de 256 bits no
    modo em que cada bloco é cifrado independentemente (modo _electronic codebook_ –
    ECB). O OpenSSL designa esse algoritmo/modo de cifra por _aes- 256 - ecb._
    Para cifrar qualquer coisa é preciso ter uma chave criptográfica. Uma solução prática para
    obter uma chave consiste em gerá-la a partir de uma _password_ (ou _passphrase_ ). O
    comando _openssl enc_ gera uma chave a partir de uma _password_ que pede ao utilizador.
    Execute o seguinte comando:
    _openssl enc -aes- 256 - ecb -in texto.txt -out texto.enc_
    Veja o conteúdo do ficheiro cifrado correndo _cat texto.enc._ 
    Como pode observar, o conteúdo é incompreensível.

5. Decifre o ficheiro usando o seguinte comenado, e observe como os dois ficheiros txt são iguais.

```
openssl enc -d -aes- 256 - ecb -in texto.enc -out texto-decifrado.txt
```

6. Observe o conteúdo do ficheiro _texto.enc_ usando o comando _hexdump_ (corra _man_
    _hexdump_ se não souber para que serve). Use a opção -v para ver o ficheiro completo:
    _hexdump -v -C texto.enc_
    Veja como no ficheiro _texto.enc_ há um padrão visível. Isso é mau pois facilita a
    criptoanálise. Por isso não se deve usar o modo ECB.

7. Vamos então cifrar novamente o ficheiro usando o modo CBC (Cipher Block Chaining).
    Execute:
    _openssl enc -aes- 256 - cbc -in texto.txt -out texto.enc_
    Veja o conteúdo do ficheiro _texto.enc_ com o comando _hexdump_ e repare como já não há
    um padrão observável.

8. Decifre o ficheiro executando o comando:
    _openssl enc -d -aes- 256 - cbc -in texto.enc -out texto-decifrado.txt_
    Observe como os dois ficheiros _txt_ são iguais.

9. No modo CBC além da chave é preciso fornecer um _initialization vector_ (IV). Esse valor é
    também gerado a partir da _password_. 
    Observe a chave e o IV gerados (em hexadecimal) usando o comando:

```
    _openssl enc -aes- 256 - cbc -P_
```

   Quantos bits têm a chave e o IV (lembre-se de que cada caracter hexadecimal representa 4 bits)?

10. O uso de uma _password_ para gerar chaves criptográficas pode ser inseguro. Por exemplo,
    o algoritmo AES com uma chave de 128 ou 256 bits é considerado seguro. No entanto, se a
    _password_ tiver poucos bytes, p.ex., 8, o número de chaves diferentes que se podem gerar
    é de apenas 64^8 = 26*8 = 2^48 (considerando 64 caracteres diferentes) que é muito menos do
    que o número de chaves de 128 bits disponíveis, que é de 2^128. Observe a diferença entre
    esses dois números calculando-os, p.ex., na seguinte calculadora: [http://www.online-](http://www.online-)
    calculator.com/scientific-calculator/

11. Como é que se sabe se uma _password_ tem tamanho suficiente? A regra é que o número
    de bits efectivo (NBE) da _password_ tem de ser maior ou igual ao número de bits da chave.
    O NBE de uma _password_ é dado por:
       NBE = log 2 (nm) – sendo _n_ o nº de caracteres disponíveis e _m_ o tamanho da
       _password._
       Considerando 64 caracteres disponíveis (26 letras maiúsculas, 26 minúsculas, 10
       números, mais 2 caracteres quaisquer), que tamanho deve ter a _password_ para
       uma chave de 128 bits? No caso de ser usado um IV, o número de bits efectivo da
       _password_ deve ser maior ou igual ao número de bits da chave mais o IV.


## Exercício	2	– Criptografia	de	chave	pública

Neste exercício vamos criar e verificar assinaturas digitais, o que exige o uso de criptografia de
chave pública.

1. Gere um par de chaves RSA com um módulo de 1024 bits usando o seguinte comando:
    _openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:1024 -aes- 256 - cbc -out_
    _mykey.pem_
    Este comando guarda as chaves no ficheiro _mykey.pem_ que, dada a criticidade da chave
    privada, é cifrado usando o algoritmo AES em modo CBC com uma chave de 256 bits
    derivada da _password_ que é pedida_._

2. Extraia a chave pública para o ficheiro _pubkey.pem_ usando o comando:
    _openssl pkey -in mykey.pem -out pubkey.pem -pubout_
    Note que a _password_ diz respeito ao ficheiro _mykey.pem_ ; o ficheiro da chave pública não é
    cifrado.

3. Observe o conteúdo dos ficheiros gerados usando o comando _cat_. Note que as chaves
    estão codificadas em base 64. Veja os diversos parâmetros que compõem as chaves
    usando os comandos:
    _openssl pkey -in mykey.pem -text_
    _openssl pkey -in pubkey.pem -pubin -text_

4. Assinar um ficheiro ou uma mensagem requer calcular a sua síntese criptográfica ou _hash._
    Pode obter um _hash_ do ficheiro _texto.txt_ usando o seguinte comando:
    _openssl dgst -sha1 texto.txt_

5. A opção _- sha1_ designa o algoritmo de _hash_ SHA1. Podem ser usados outros. Veja quais
    usando o comando:
    _openssl list-message-digest-commands_

6. O resultado do ponto anterior não é ainda uma assinatura. 
   No entanto, o comando _openssl dgst_ também serve para gerar assinaturas. 
   Execute:
    _openssl dgst -sha1 -sign mykey.pem -out sign.bin texto.txt_
    A assinatura é guardada no ficheiro _sign.bin_

7. Imagine que o ficheiro _texto.txt,_ a assinatura _sign.bin_ e a chave pública (ficheiro
    _pubkey.pem_ ) são passados a outra entidade que pretende verificar se o ficheiro foi
    modificado. Essa entidade pode verificar a assinatura correndo o comando:
    _openssl dgst -sha1 -verify pubkey.pem -signature sign.bin texto.txt_
    Corra o comando e observe o resultado.

8. Modifique o ficheiro _texto.txt,_ p.ex. apagando um caracter. Execute novamente o
    comando da alínea anterior e observe o resultado.


## Exercício	3	– Sínteses	de	ficheiros

O Linux tem comandos para gerar e verificar sínteses de ficheiros. Esses comandos são úteis para
verificar se os ficheiros não foram corrompidos no disco ou quando transferidos de/para outro
computador. O nome dos comandos varia com o algoritmo de _hash: md5sum, sha1sum,
sha224sum, sha256sum, sha384sum_ e _sha512sum._

1. Calcule a síntese MD5 do ficheiro _texto.txt_ executando:
    _md5sum texto.txt > hfile_
    Veja o _hash_ fazendo _cat hfile._

2. Verifique o _hash_ executando:
    _md5sum -c hfile_

3. Calcule a síntese SHA1 de todos os ficheiro da directoria executando:
    _sha1sum * > hfile_
    e verifique executando
    _sha1sum -c hfile_

4. Essas sínteses permitem detectar se os ficheiros foram corrompidos acidentalmente.
    Servem também para detectar se os ficheiros foram corrompidos intencionalmente, por
    uma pessoa mal intencionada?

## Referências

- OpenSSL, [http://www.openssl.org/](http://www.openssl.org/)


**Acknowledgments**

This guide was written by Vasco Guita.
