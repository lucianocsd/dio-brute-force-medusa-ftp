# #                            DIO - CIBERSEGURANÇA

# Brute-Force com Medusa (FTP — Metasploitable2)

Este relatório documenta um exercício controlado de segurança ofensiva, com o propósito de demonstrar a eficácia de um ataque de força bruta contra o serviço FTP de um servidor vulnerável (Metasploitable2), utilizando a ferramenta Medusa.

Nosso ambiente de testes possui:

> Virtualização: Oracle VirtualBox.

> Rede: Modo Host-Only para isolar o laboratório da rede física.

> Máquina Atacante: Kali Linux (IP: 192.168.56.101).

> Máquina Alvo: Metasploitable2 (IP: 192.168.56.101).

> Serviço Alvo: Serviço FTP (Porta 21).

> Ferramenta Principal: Medusa.

---

A técnica de força bruta consiste em testar massivamente combinações de usuário e senha para passar pela autenticação. Softwares como o Medusa são essenciais para dar agilidade a esse processo, que seria impraticável de forma manual. O objetivo deste guia é demonstrar o uso prático dessas ferramentas em laboratório, simulando um cenário real para identificar vulnerabilidades.


> Antes de começar, fica o alerta: ataques de força bruta são ilegais se você não tiver autorização por escrito para fazer. O objetivo aqui é puramente educacional. Só use essas técnicas num ambiente controlado ou nas suas próprias máquinas. Sair testando em sistemas de terceiros sem permissão pode te render graves consequências legais.

---

==== Medusa

Medusa representa uma solução de código aberto para ataques de força bruta, projetada para ser ágil por meio de processamento paralelo. Criada para avaliações de segurança, possibilita que especialistas verifiquem a robustez de sistemas contra invasões por senhas, oferecendo suporte flexível a vários protocolos, incluindo FTP, SSH e RDP. A finalidade principal é descobrir senhas pouco seguras e brechas de segurança antes que possam ser utilizadas indevidamente.

==== Instalação

Para ilustrar, recorri ao Kali Linux, uma versão do sistema operacional voltada para a área de segurança, que já vem com o Medusa instalado, facilitando a realização dos testes.

Se você estiver usando outra versão do Debian, como o Ubuntu, ou se o Medusa não estiver presente no seu Kali, a instalação é bem fácil.

```bash
sudo apt update
sudo apt install medusa
```

Para ter certeza de que o Medusa está funcionando corretamente, abra o seu terminal e digite o comando "medusa". Se tudo tiver corrido bem na instalação, o programa irá mostrar o menu de ajuda. Nele, será possível encontrar as opções e parâmetros, o que indica que está tudo certo para continuar.


<img width="1016" height="627" alt="518774393-83c06f48-d071-493b-8ec9-9dab6ced4ff7" src="https://github.com/user-attachments/assets/601b8863-5728-466d-9378-48afcabea75d" />


==== Sintaxe

Sintaxe básica de uso:

```bash
medusa -h <target IP> -U <arquivo_de_usuarios> -P <arquivo__desenhas> -M <módulo> -t <threads>
```

| Flag | Ação                                                                    |
| ---- | --------------------------------------------------------------          |
| *-h* | Especifica o endereço do servidor alvo (IP ou domínio).                 |
| *-U* | Define o arquivo contendo a lista de usuários para o teste.             |
| *-P* | Define o arquivo contendo a lista de senhas para o teste.               |
| *-M* | Seleciona o módulo do protocolo a ser utilizado (ex: ftp, ssh).         |
| *-t* | Controla o número de threads para tentativas simultâneas (padrão: 1).   |

Existem outros parâmetros específicos para cada protocolo. Consulte o manual:

*Mais informações:* [*Documentation Medusa*](https://jmk-foofus.github.io/medusa/medusa.html)

---

==== Configuração do Ambiente 

Para montar um laboratório seguro para os estudos, eu criei duas máquinas virtuais no VirtualBox. 
O Kali Linux está fazendo o papel do atacante, e o Metasploitable é o nosso alvo para os testes.

==== VirtualBox 

Software gratuito e multiplataforma para criar/gerenciar máquinas virtuais. Permite testar sistemas com segurança.

É possível baixá-lo, atentando-se ao seu sistema operacional, na página a seguir.

*Mais informações:* [*VirtualBox*](https://www.virtualbox.org/)

==== Kali Linux

É uma distribuição Linux voltada para testes de penetração e análise forense digital. Pode ser usada via ISO ou importando a VM pronta para VirtualBox disponível no site.

Após criar a VM, nas configurações de **Network** configure o adaptador como **Host-Only Adapter** para que Kali e Metasploitable fiquem na mesma rede isolada.

<img width="886" height="349" alt="kali" src="https://github.com/user-attachments/assets/443debf8-97df-480d-a853-76c3d424ec57" />



 *Mais informações:* [*Kali Linux*](https://www.kali.org/get-kali/#kali-platforms)

==== Metasploitable

Metasploitable é uma máquina virtual intencionalmente vulnerável, usada para treinamentos. O arquivo baixado vem como um "disco virtual", com a extensão ".vdi ou .vmdk"
Com ele em mãos, crie uma VM no VirtualBox do tipo Linux/Ubuntu e, na aba **Storage**, substitua o disco virtual pelo `Metasploitable.vmdk`.

<img width="890" height="509" alt="disk" src="https://github.com/user-attachments/assets/f9edc4e7-3c67-4358-ac29-658ca48ccd4f" />


Na aba System, altere a ordem de boot(inicialização), deixe selecionado a opção **Hard Disk**, para que a máquina inicie pelo disco mencionado acima.

<img width="884" height="482" alt="order boot" src="https://github.com/user-attachments/assets/81593907-5b77-47f4-a1cd-d753bc827a44" />


Configure a rede da VM do Metasploitable2 também como **Host-Only** e inicie a VM. Login da Máquina Virtual = Usuário: msfadmin / Senha: msfadmin.

 *Mais informações e Download:* [*Metasploitable*](https://sourceforge.net/projects/metasploitable/)


==== Teste de Conexão

No Metasploitable, consulte o IP da VM com `ifconfig`. No meu caso: `192.168.56.101`. No Kali, teste com:

```bash
ping -c 6 192.168.56.101
```

A rede estará corretamente comunicável quando houver resposta no comando acima, exemplificado abaixo:


<img width="523" height="272" alt="resposta-ping" src="https://github.com/user-attachments/assets/240f6506-443d-4dbb-86b0-e814d6e1543f" />


---

==== Exemplo de Uso

Usei um servidor FTP existente dentro da VM Metasploitable como alvo, simulando um cenário de avaliação em um servidor antigo.

==== Enumeração

Primeira etapa: enumeração de serviços com o **nmap**. Para verificar a porta 21 (FTP):

```bash
nmap -sV -p 21 192.168.56.101
```

O parâmetro `-p 21` especifica a porta e `-sV` tenta identificar a versão do serviço.

<img width="1060" height="360" alt="nmap" src="https://github.com/user-attachments/assets/faad3b29-8d24-4a97-8640-8a2d4e2aa6ac" />

Essa informação a respeito da versão do serviço é extremamente importante, pois indica potenciais vulnerabilidades conhecidas (exemplo: [CVEs](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)) que podem ser pesquisadas. No momento, foquei apenas no ataque de força bruta, não explorei essa vulnerabilidade

> Observação: 
> Como o foco é o brute force com Medusa, não irei me aprofundar no `nmap`. Ele pode, contudo, fornecer muitas outras informações úteis. Mais informações em: [nmap.org](https://nmap.org/download)

==== Listas

O Medusa utiliza listas de usuários e senhas para realizar tentativas de combinações. Essas listas podem ser criadas manualmente ou obtidas de fontes públicas (É importante sempre respeitar a legalidade). Para este teste eu criei listas simples:

```bash
echo -e 'user\nmsfadmin\nadmin\nroot' > users.txt
echo -e '123456\npassword\nqwerty\nmsfadmin' > passwords.txt
```

==== Ataque

Com os arquivos criados, executei:

```bash
medusa -h 192.168.56.101 -U users.txt -P password -M ftp -t 6
```

<img width="1061" height="588" alt="medusa1" src="https://github.com/user-attachments/assets/450ea5d1-ac4b-4d24-b375-e6b696e21ef3" />



Com 6 threads, o Medusa testou as combinações e retornou uma entrada com `ACCOUNT FOUND` / `SUCCESS` para uma combinação válida.
<img width="1046" height="20" alt="image" src="https://github.com/user-attachments/assets/17024b2d-022d-45c2-a66e-a8f79bcc6c4a" />


Em seguida, testei o acesso FTP:

```bash
ftp 192.168.56.101
```

<img width="319" height="124" alt="ftp-conected" src="https://github.com/user-attachments/assets/090b87e0-f2f3-4982-804e-e70754cf5d9d" />



A demonstração ilustra como uma pessoa mal intencionada pode comprometer um serviço exposto e configurado inadequadamente, de maneira automatizada e relativamente rápida.

---

==== Mitigação da Falha

Algumas das recomendações possíveis para mitigar ataques de força bruta contra serviços FTP em um servidor:

1. **Desligue o FTP:** Se você realmente não precisar do serviço de FTP, a melhor coisa a fazer é desativá-lo completamente.
2. **Mude para SFTP ou FTPS**: O FTP normal é inseguro. Se você precisa transferir arquivos, migre para o SFTP (FTP Seguro). É a melhor opção! Se não puder migrar, ative o FTPS (que usa criptografia TLS/SSL) como segunda melhor alternativa.
3. **Desabilitar login anônimo** e remover contas padrão existentes.
4. **Implementar uma proteção por bloqueio de tentativas** (exemplo.: Fail2ban).
5. **Restringir o acesso por firewall** (apenas IPs confiáveis/privados).
6. **Habilitar chroot e restringir permissões** para garantir que, ao fazer login, o usuário fique "preso" no seu diretório home e não possa navegar pelo resto do sistema de arquivos. Além disso, restrinja as permissões de arquivo ao mínimo necessário..
7. **Forçar senhas fortes / usar autenticação por chave (em SFTP)**.
8. **Mantenha o software sempre atualizado** e monitore os logs regularmente para pegar qualquer atividade suspeita

---


# # Brute-Force - Formulário de login, de sistema WEB (FTP — Metasploitable2)



Aqui vamos automatizar o script no medusa, para que ele realize a tentativa "forçada" de entrada em um formulário web. 
Em nossa VM do metasploitable existe um serviço de web, no link [*192.168.56.101/dvwa/login.php*](192.168.56.101/dvwa/login.php), chamado **Damn Vulnerable Web App (DVWA)**.




Acessando esse endereço em nossa máquina atacante com o Kali Linux, e já dentro da página, acessamos a ferramenta de Desenvolvedor do navegador (atalho.: TECLA F12 - Microsoft Edge). 
Nela, podemos ter acesso a informações de conexões (entrada e saída), usando a Aba **Network**. Como por exemplo, em **Request** teremos informação exata de uma tentativa de login realizada.

No exemplo, usamos o usuário: luck, a senha: 123:




Na página é possível ver uma mensagem de erro, “Login failed”. Podemos estrategicamente configurar o Medusa para indicar que houve falha ao testar a senha.


Para explorar a vulnerabilidade, usando força bruta em formulário WEB, devemos entender alguns aspectos:


|   1   | Nem todos os formulários são tão simples como o apresentado no exemplo.      |

|   2   | Algumas aplicações poderão usar mecanismos de proteção, como o **Token CSRF**. |


---
 
==== Forçando o Ataque ao formulário, wordlist.

```bash
echo -e 'user\nmsfadmin\nadmin\nroot' > users.txt
echo -e '123456\npassword\nqwerty\nmsfadmin' > passwords.txt
```

No medusa usamos o comando:
```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'/username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```

<img width="1078" height="677" alt="form-medusa" src="https://github.com/user-attachments/assets/c97cde7e-8aca-4aa7-aed2-fa80239071fc" />



| Flag | Ação                                                                             |
| ---- | --------------------------------------------------------------                   |
| *-h* | Especifica o endereço do servidor alvo (IP ou domínio).                          |
| *-U* | Define o arquivo contendo a lista de usuários para o teste.                      |
| *-P* | Define o arquivo contendo a lista de senhas para o teste.                        |
| *-M* | Seleciona o módulo do protocolo a ser utilizado (neste caso foi 'http').         |
| *-m PAGE* | Informamos aqui o caminho do formulário.                                    |
| *-m FORM* | É o corpo da requisição, usamos cada linha do arquivo como credenciais.     |
| *-m FAIL* | Informamos ao medusa qual a resposta esperada para uma tentativa com falha. |
| *-t* | Controla o número de threads para tentativas simultâneas (atual: 6).            |

---

==== Mitigação da Falha

Algumas das recomendações possíveis para mitigar ataques de força bruta contra formulários WEB:

1. **Usar valores únicos por sessão:** O objetivo é ter certeza que cada sessão ou requisição seja única.
2. **Utilização de Captcha:** É um mecanismo de segurança muito usado nos dias de hoje, que basicamente exige uma prova de que a ação está sendo realizada por um humano e não um robô.
3. **Obrigatoriedade de Cookies:** O login só funcionará se a sessão possuir cookies válidos.

---

# Ataques com enumeração contra serviço SMB

Geralmente, o foco desses ataques são ambientes corporativos, explorando vulnerabilidade de servidores de arquivo por exemplo.

> Podemos entender o SMB como um tipo de "porta de entrada" para compartilhamentos de recursos na rede interna.

Abaixo, vamos simular uma situação onde conseguimos o acesso a uma rede interna (uma máquina física ou por vetor) e achou um servidor SMB ativo. 
Através desse ativo exposto podemos achar usuários no sistema e testar senhas relativamente "fracas" nele, tudo isso de forma discreta sem bloquear nenhuma conta. Para isso contamos com uma
técnica chamada **PASSWORD SPRAYING**. 

> Técnica furtiva e silenciosa, difícil de detectar e muito funcional em credenciais fracas ou previsíveis.

---

==== Simulação de Cenário 

>Simulando um cenário comum, de um ambiente corporativo mal configurado. 


Cenário proposto: Descobrir quais portas realmente existem e quais levam a lugares úteis para o atacante.
Indicado fazer isso, pois se realizarmos varreduras e ir testando em todas as portas disponíveis ou locais vulneráveis, iremos perder tempo e ainda ser descoberto.

A melhor estratégia é focar onde realmente é útil conseguir um ataque.



```bash
enum4linux -a 92.168.56.101 | tee enum4_output.txt
```

<img width="1004" height="540" alt="enum4linux" src="https://github.com/user-attachments/assets/fbef7eb8-d610-4c5f-aedf-8f38980f7fb0" />



| Flag  | Ação                                                            |
| ----  | --------------------------------------------------------------  |
| *-a*  | Testa todas as técnicas possíveis de enumeração.                |
| *tee* | Grava o arquivo com as informações extraídas e exibe em tela.   |


Para acessarmos o arquivo gerado, usamos:
```bash
less enum4_output.txt
```

> RID =  Identificador real de um usuário no sistema.

---

==== Criando uma wordlist com usuários e senhas (Utilizando a técnica de Password Spraying)

```bash
echo -e "user\nmsfadmin\nservice" > smb_users.txt
```


```bash
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt
```


> As senhas utilizadas que foram utilizadas são apenas um teste. No conteúdo desse arquivo poderíamos utilizar uma lista maior e mais elaborada, a fim de obter mais sucesso nas tentativas de exploração de vulnerabilidade.


---

==== Executando o Medusa novamente

```bash
medusa -h 192.168.56.101 -U smb_users.txt -P senhas.spray.txt -M smbnt -t 2 -T 50
```

<img width="1087" height="530" alt="medusa2" src="https://github.com/user-attachments/assets/773ef4b5-6bb6-40f5-beb5-92a707afe18c" />



---


==== Considerações finais (ética e escopo)

O teste proposto nestes tópicos foram unicamente testados/realizados em ambiente controlado e de testes (Metasploitable) com finalidade educacional. **Não realize varreduras ou ataques em sistemas de terceiros sem autorização prévia**. Em um portfólio, deixe explícito o escopo do teste, os limites e as autorizações.




