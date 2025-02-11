# CTF-PaddingOracle
---

## **Enunciado do Problema**  
Conexão ao serviço vulnerável:  
`nc crypto.utctf.live 4356`

O problema se baseia em vulnerabilidades da cifra **AES-CBC**, especificamente no ataque **Padding Oracle**.

---

## **Introdução ao Tema**  
O Padding Oracle Attack é uma técnica de exploração de vulnerabilidades criptográficas que pode
ser usada contra sistemas que implementam a cifra AES (Advanced Encryption Standard) em modos
de operação como o CBC (Cipher Block Chaining).
Esse tipo de ataque explora falhas no processo de verificação de preenchimento (padding) de blocos de dados durante a descriptografia, baseado no padrão PKCS#7. 

O problema Delphi é um exemplo prático de um sistema vulnerável à esse ataque, onde um atacante pode recuperar dados sensíveis sem precisar conhecer a chave de criptografia, apenas manipulando a forma como o
padding é verificado e interpretado pelo oracle.

---

## **Como o ataque funciona?**  
Digamos que queremos saber o texto claro P2, nós sabemos que ele é calculado dessa forma:
P2 = AES(C2) ⊕ C1 

Onde C1 é o ciphertext anterior ao  que queremos decifrar 
(ao invés de C1 poderíamos estar trabalhando com o IV), e C2 é o ciphertext que queremos
decifrar para chegar ao seu texto claro (P2).

Assim, o ataque consiste em manipular C1 de forma a receber do oracle uma confirmação 
de padding válido. De forma que o atacante vai testando todas as possiblidades possíveis 
(0 à 255) o valor de cada byte em C1 que faz o oracle retornar True.

Dessa forma, quando o atacante acha um byte que o oracle valide o padding, ele faz a seguinte
operação:

P2[16] = C1’[16] ⊕ (C1[16]  ⊕ ( padding)) - Para o caso do último byte

Realizando a mesma operação para descobrir os 16 bytes do bloco de P2, 
apenas incrementando o valor do padding (o próximo, por exemplo, será 0x02/0x02). 
Realizando esse processo para os n-blocos que foram necessários para cifrar o plaintext,
decifrando assim a palavra requerida.

---

## **AES-GCM como solução**  
O **AES-GCM (Galois Counter Mode)** combina criptografia e autenticação, mitigando ataques como o **Padding Oracle**. Ele:  
- Usa **autenticação integrada** para garantir que os dados não foram alterados.  
- Não utiliza **padding**, eliminando a exploração dessa vulnerabilidade.  

Para mitigar ataques do tipo **Padding Oracle**, é recomendado:  
- **Tratar erros de forma genérica**, sem revelar detalhes sobre falhas no padding.  
- **Utilizar AES-GCM** em vez de AES-CBC.  

---

## **SSL e TLS**  
O **SSL (Secure Sockets Layer)** e seu sucessor **TLS (Transport Layer Security)** foram criados para proteger dados transmitidos na Internet.  

O **SSL/TLS** funciona por meio de um **handshake** que autentica e criptografa a comunicação entre cliente e servidor.  

### **Problema do SSL/TLS**  
Em versões antigas, **o modo CBC era utilizado**, tornando o protocolo vulnerável ao **Padding Oracle Attack**.  

Pesquisadores demonstraram que, analisando **o tempo de resposta do servidor**, era possível inferir se o padding estava correto, quebrando a criptografia do SSL/TLS.

---

## **Ataques baseados em tempo e múltiplas sessões**  

### **Ataque de Timing**  
O servidor demora mais para processar um padding correto do que um incorreto, pois:  
- Se o padding está **correto**, ele verifica o **MAC (Message Authentication Code)**, levando mais tempo.  
- Se o padding está **incorreto**, a verificação para imediatamente.  

Dessa forma, um atacante pode medir o tempo de resposta e inferir **se o padding estava correto**.

### **Ataque de Múltiplas Sessões**  
1. O atacante intercepta e armazena múltiplos blocos cifrados ao longo de várias sessões.  
2. Com um "oracle" analisando mensagens de erro, ele deduz informações sobre o texto claro.  
3. Se um bloco constante for repetido (como uma senha), ele pode ser reconstruído gradualmente.  

### **Ataque de Múltiplas Sessões com Dicionário**  
1. O atacante assume que o texto claro segue padrões previsíveis (como palavras de um idioma).  
2. Ele utiliza um dicionário de senhas comuns para testar possibilidades rapidamente.  
3. Esse método **acelera** o processo de descriptografia.  

---

## **Contramedidas**  
Para evitar esses ataques, as soluções recomendadas incluem:  
- **Verificar o MAC antes do padding**, prevenindo ataques ativos.  
- **Respostas uniformes**: garantir que mensagens de erro tenham tempos de resposta idênticos.  
- **Introduzir atrasos aleatórios** para mascarar diferenças no tempo de resposta.  
- **Monitorar erros de sessão** para identificar possíveis ataques.  

---

## **Conclusão**  
O ataque **Padding Oracle** explora falhas na criptografia **AES-CBC**, permitindo que um atacante **recupere dados cifrados sem conhecer a chave secreta**.  

A melhor defesa contra esse tipo de ataque é **migrar para AES-GCM** e **adotar práticas de segurança robustas no tratamento de erros**.

---

## **Referências**  
- [Robert Heaton - Padding Oracle Attack](https://robertheaton.com/2013/07/29/padding-oracle-attack/)  
- [Oracle Padding Attack - Medium](https://medium.com/@masjadaan/oracle-padding-attack-a61369993c86)  
- [Vídeo: Padding Oracle Attack](https://www.youtube.com/watch?v=lkPBTJ3yiCI)  
- [Cloudflare: O que é SSL/TLS?](https://www.cloudflare.com/pt-br/learning/ssl/what-is-ssl/)  
- [UTCTF 2021 - Desafio Delphi](https://github.com/utisss/UTCTF-21/tree/main/crypto-delphi)  

