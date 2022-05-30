Instituto Superior Técnico, Universidade de Lisboa

**Segurança Informática em Redes e Sistemas**

# Guião de laboratório: Criptografia com OpenSSL

## Objectivo

O objetivo deste guia é aprofundar os conhecimentos sobre criptografia.
O [OpenSSL](https://www.openssl.org/) é, como o nome sugere, uma implementação aberta do protocolo SSL/TLS (*Secure Sockets Layer*/*Transport Layer Security*).
No entanto, oferece também uma biblioteca para programação e comandos de linha que permitem utilizar diversos os algoritmos criptográficos.

---

## Exercício 1 – Criptografia simétrica

O primeiro exercício consiste em cifrar e decifrar um ficheiro usando criptografia simétrica.

1. Todas as operações que vamos realizar usam o comando de linha `openssl`.
Comece por ver que está disponível o manual desse comando correndo `man openssl`.
Pode também aceder aos manuais dos subcomandos usando `man`, p.ex., `man enc` para o subcomando `openssl enc`.

2. Observe os algoritmos de criptografia simétrica disponíveis executando:  
    `openssl list -cipher-commands`  
Pode observar variações de algoritmos como AES (Advanced Encryption Standard, a norma norte-americana atual), DES, RC2, RC4, Blowfish (bf), Camellia.
Note que nem todos os resultados são realmente algoritmos criptográficos, p.ex., _base64_ é uma forma de codificação de binário em caracteres de texto e não um algoritmo criptográfico.

3. Crie um ficheiro com o nome `texto.txt` e 10 linhas todas com o conteúdo `Texto em claro!`.

4. Vamos cifrar esse ficheiro como uma cifra de blocos, AES, com uma chave de 256 bits no modo em que cada bloco é cifrado independentemente, ou seja, modo ECB (_electronic codebook_).
O OpenSSL designa esse algoritmo/modo de cifra por _aes-256-ecb._
Para cifrar qualquer coisa é preciso ter uma chave criptográfica.
Uma solução prática para obter uma chave consiste em gerá-la a partir de uma _password_ (ou _passphrase_).
O comando `openssl enc` gera uma chave a partir de uma _password_ que pede ao utilizador.
Execute o seguinte comando:  
`openssl aes-256-ecb -in texto.txt -out texto.enc`

Veja o conteúdo do ficheiro cifrado executando `cat texto.enc`.
Como pode observar, o conteúdo é incompreensível.

5. Decifre o ficheiro usando o seguinte comando, e observe como os dois ficheiros de texto são iguais:  
`openssl aes-256-ecb -d -in texto.enc -out texto-decifrado.txt`

6. Observe o conteúdo do ficheiro `texto.enc` usando o comando `hexdump` (corra `man hexdump` se não souber para que serve). 
Use a opção `-v` para ver o ficheiro completo: `hexdump -v -C texto.enc`
Veja como no ficheiro `texto.enc` há um padrão visível. Como aes é uma bijeção, um atacante consegue inferir que o conteúdo desses blocos é igual, apesar de não conseguir descobrir qual o conteúdo desses blocos.
Esta é a razão pela qual não se deve usar o modo ECB se não quisermos dar esta informção.

7. Vamos então cifrar novamente o ficheiro usando o modo CBC (*Cipher Block Chaining*).
Execute:  
`openssl aes-256-cbc -in texto.txt -out texto.enc`  
Veja o conteúdo do ficheiro `texto.enc` com o comando `hexdump` e repare como já não há um padrão observável.

8. Decifre o ficheiro executando o comando:  
`openssl aes-256-cbc -d -in texto.enc -out texto-decifrado.txt`  
Observe como os dois ficheiros de texto são iguais.

9. No modo CBC além da chave é preciso fornecer um IV (_initialization vector_).
Neste caso, este valor é também gerado a partir da _password_.
Observe a chave e o IV gerados (em hexadecimal) usando o comando:  
`openssl aes-256-cbc -P`  
Quantos bits têm a chave e o IV?
Lembre-se de que cada caracter hexadecimal representa 4 bits.

10. O uso de uma _password_ para gerar chaves criptográficas pode ser inseguro.
Por exemplo, o algoritmo AES usa uma chave de 128 ou 256 bits, o que é considerado seguro.
No entanto, se a _password_ tiver poucos bytes, p.ex., 8, o número de chaves diferentes que se podem gerar é de apenas 64^8 = 26*8 = 2^48 (considerando 64 caracteres diferentes), o que é muito menos do que o número de chaves de 128 bits disponíveis, que é de 2^128.
Pode observar quantos blocos `aes` o seu processador pode cifrar por segundo num unico core correndo:
`openssl speed aes`
Quais as implicações de usar uma chave curta? Ie, o que aconteceria se a chave tivesse apenas 8 bits?
Quanto tempo demoraria um ataque de força bruta a uma chave de 8 bytes usando uma máquina igual à sua? E 1000 máquinas?



11. Como é que se sabe se uma _password_ tem tamanho suficiente?
A regra é que o número de bits efectivo (NBE) da _password_ tem de ser maior ou igual ao número de bits da chave.
O NBE de uma _password_ é dado por:  
$NBE = log 2 (n^m)$ – sendo _n_ o número de caracteres disponíveis e _m_ o tamanho da _password._
Considerando 64 caracteres disponíveis (26 letras maiúsculas, 26 minúsculas, 10 números, mais 2 caracteres quaisquer), que tamanho deve ter a _password_ para uma chave de 128 bits?
No caso de ser usado um IV, o número de bits efectivo da _password_ deve ser maior ou igual ao número de bits da chave mais o IV.

---

## Exercício 2 - Criptografia de chave pública

Neste exercício vamos criar e verificar assinaturas digitais, o que exige o uso de criptografia de chave pública.

1. Gere um par de chaves RSA com um módulo de 1024 bits usando o seguinte comando:  
`openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:1024 -aes-256-cbc -out mykey.pem`  
Este comando guarda as chaves no ficheiro `mykey.pem` que, dada a importância da chave    privada, é cifrado usando o algoritmo AES em modo CBC com uma chave de 256 bits derivada da _password_ que é pedida.

2. Extraia a chave pública para o ficheiro `pubkey.pem` usando o comando:  
`openssl pkey -in mykey.pem -out pubkey.pem -pubout`  
Note que a _password_ diz respeito ao ficheiro `mykey.pem`;
o ficheiro da chave pública não é cifrado.

3. Observe o conteúdo dos ficheiros gerados usando o comando `cat`.
Note que as chaves estão codificadas em base 64.
Veja os diversos parâmetros que compõem as chaves usando os comandos:  
`openssl pkey -in mykey.pem -text`  
`openssl pkey -in pubkey.pem -pubin -text`

4. Para assinar um ficheiro ou uma mensagem, é necessário calcular a sua síntese criptográfica ou _hash_.
Pode obter um _hash_ do ficheiro `texto.txt` usando o seguinte comando:  
`openssl dgst -sha1 texto.txt`

5. A opção `- sha1` designa o algoritmo de _hash_ SHA1.
Podem ser usados outros. Veja quais através do comando:  
`openssl list -digest-commands`

6. O resultado do ponto anterior não é ainda uma assinatura, porque uma síntese não tem segredos.
No entanto, o comando `openssl dgst` também serve para gerar assinaturas.
Execute:  
`openssl dgst -sha1 -sign mykey.pem -out sign.bin texto.txt`  
A assinatura é guardada no ficheiro `sign.bin`.

7. Imagine que o ficheiro `texto.txt`, a assinatura `sign.bin` e a chave pública (ficheiro    `pubkey.pem`) são passados a outra entidade que pretende verificar se o ficheiro foi modificado. 
Essa entidade pode verificar a assinatura correndo o comando:  
`openssl dgst -sha1 -verify pubkey.pem -signature sign.bin texto.txt`  
Corra o comando e observe o resultado.

8. Modifique o ficheiro `texto.txt`, p.ex., apagando um caracter.
Execute novamente o comando da alínea anterior e observe o resultado.

---

## Exercício 3 – Sínteses de ficheiros

O Linux tem comandos para gerar e verificar sínteses de ficheiros.
Esses comandos são úteis para verificar se os ficheiros não foram corrompidos no disco ou quando transferidos de/para outro computador. O nome dos comandos varia com o algoritmo de _hash_: _md5sum_, _sha1sum_, _sha224sum_, _sha256sum_, _sha384sum_ e _sha512sum_.

1. Calcule a síntese MD5 do ficheiro `texto.txt` executando:  
`md5sum texto.txt > hfile`  
Veja o _hash_ fazendo `cat hfile`.

2. Verifique o _hash_ executando:  
`md5sum -c hfile`

3. Calcule a síntese SHA1 de todos os ficheiro da directoria executando:  
`sha1sum * > hfile`  
e verifique executando:  
`sha1sum -c hfile`

4. Essas sínteses permitem detetar se os ficheiros foram corrompidos.
Qual o nome dessa propriedade?
Seria possível recuperar os ficheiros baseados na síntese?

5. Assumindo que apenas um bit de um ficheiro foi trocado, proponha um algoritmo para recuperar o ficheiro original dado o *hash* original e o ficheiro com um bit trocado.
Qual a complexidade do seu algoritmo?

6. Calcule a síntese MD5 dos ficheiros `ship.jpg` e `plane.jpg`.
O que observou?

7. O resultado do cálculo anterior não foi uma coincidência.
Os ficheiros foram gerados ao mesmo tempo com o objetivo de terem o mesmo *hash*.
Que propriedade do MD5 foi quebrada?
Investigue a dificuldade deste ataque (quanto tempo demora) para o MD5 e para SHA1.

8. Como poderia usar o ataque acima para forjar uma assinatura digital?
Dado o ataque anterior, quais as implicações, no âmbito legal, de usar MD5/SHA1 em assinaturas digitais?

9. Para MD5 e SHA1 arbitrários, qual a dificuldade do melhor ataque de *preimage* conhecido atualmente?

---

## Referências

- OpenSSL, [http://www.openssl.org/](http://www.openssl.org/)

---

### Agradecimentos

A versão inicial deste guia foi escrita por Vasco Guita.
