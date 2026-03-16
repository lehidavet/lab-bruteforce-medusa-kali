# Lab de Ataques de Força Bruta com Medusa e Kali Linux

Este repositório documenta um laboratório prático realizado para o desafio da DIO, simulando ataques de força bruta e password spraying em serviços FTP e SMB expostos em uma máquina Metasploitable 2, a partir de uma máquina atacante Kali Linux. O objetivo é entender o funcionamento desses ataques em ambiente controlado e propor medidas de mitigação.

## Ambiente de Laboratório

Para este desafio, montei um ambiente isolado em máquinas virtuais, garantindo que todos os testes fossem realizados em rede controlada.

- **Hypervisor:** Oracle VirtualBox.
- **Rede:** adaptadores configurados em modo interno/host-only, permitindo comunicação apenas entre as VMs e o host.
- **Máquina Atacante – Kali Linux**
  - IP: `192.168.127.7`
  - Principais ferramentas utilizadas: Nmap e Medusa.
- **Máquina Alvo – Metasploitable 2**
  - IP: `192.168.127.5`
  - Serviços vulneráveis habilitados por padrão, incluindo FTP e SMB.

Antes de iniciar os ataques, validei a conectividade entre as máquinas com um scan de descoberta de hosts na rede `192.168.127.0/24` utilizando o Nmap:

```bash
nmap -sP 192.168.127.0/24
```
![Evidência nmap inicial](images/nmap_inicial.png)

## Ataque de Força Bruta em FTP com Medusa

Para identificar senhas fracas no serviço FTP da máquina Metasploitable 2, utilizei o Medusa a partir do Kali Linux.

Primeiro, confirmei o IP do alvo (`192.168.127.5`) com um scan na rede `192.168.127.0/24` usando o Nmap. Em seguida, executei o seguinte comando:

```bash
medusa -h 192.168.127.5 -U wordlists/usuarios.txt -P wordlists/senhas.txt -M ftp -t 4
```

![Evidência de brute force em FTP](images/ftp_bruteforce.png)

### Validação de Acesso com smbclient

Após identificar as credenciais válidas pelo Medusa (`msfadmin/msfadmin`), validei o acesso ao compartilhamento SMB utilizando o smbclient a partir do Kali Linux:

```bash
smbclient -L 192.168.127.5 -U msfadmin
smbclient //192.168.127.5/tmp -U msfadmin
```

![Evidência Validação de smbclient](images/smbclient_validacao.png)

Com essas credenciais, consegui listar os compartilhamentos disponíveis e acessar o recurso `tmp`, confirmando na prática que a combinação de usuário e senha descoberta pelo ataque permite interação com o serviço SMB da máquina alvo.

## Password Spraying em SMB com Medusa

Para simular um ataque de password spraying contra o serviço SMB da máquina Metasploitable 2, reutilizei a mesma lista de usuários e defini uma lista de senhas comuns (`senhas.txt`) para serem testadas em todos os logins.

Primeiro, confirmei que as portas 21, 22, 80, 139 e 445 estavam abertas no alvo com o Nmap:

```bash
nmap -sV -p 21,22,80,445,139 192.168.127.5
```
![Evidência de NMAP](images/nmap_bruteforce.png)

O resultado confirmou, além do FTP na porta 21, os serviços HTTP (80/tcp) e SMB/NetBIOS (139 e 445/tcp) ativos no alvo, que foram utilizados nas etapas seguintes de teste.

Em seguida, executei o seguinte comando com o módulo `smbnt` do Medusa:

```bash
medusa -h 192.168.127.5 -U wordlists/usuarios.txt -P senhas.txt -M smbnt -t 2 -T 50
```

![Evidência de password spraying em SMB](images/medusa_smb.png)

Diferentemente da força bruta tradicional, onde muitas senhas são testadas contra um único usuário até encontrar a combinação correta, no password spraying um pequeno conjunto de senhas comuns é testado contra vários usuários. Essa abordagem reduz a chance de disparar políticas de bloqueio de conta baseadas em tentativas consecutivas de falha, ao mesmo tempo em que explora senhas fracas e reutilizadas em serviços de autenticação em rede como o SMB.

## Medidas de Mitigação

Com base nos testes realizados, algumas boas práticas para reduzir o risco de ataques de força bruta e password spraying são:

- **Fortalecer políticas de senha:** exigir senhas longas, complexas e únicas, evitando padrões fáceis como `msfadmin` ou `admin`.
- **Implementar bloqueio e delays:** configurar bloqueio temporário de contas ou aumento de delay após múltiplas falhas de login em serviços como FTP e SMB.
- **Restringir exposição de serviços:** limitar o acesso a serviços como FTP e SMB apenas a redes internas ou VPN, evitando exposição direta à internet.
- **Monitorar logs de autenticação:** coletar e analisar logs de falhas de login em busca de padrões de brute force ou spraying, integrando com soluções de SIEM/IDS.
- **Adotar autenticação adicional:** sempre que possível, complementar com MFA e revisões periódicas de contas e permissões.

## Reflexão sobre Força Bruta em Aplicações Web (DVWA)

Além dos testes em protocolos de rede como FTP e SMB, estudei também o módulo de Brute Force da Damn Vulnerable Web Application (DVWA), focado em formulários de login HTTP. Nesse cenário, o atacante automatiza tentativas sucessivas de autenticação contra um formulário web, reaproveitando listas de usuários e senhas de maneira semelhante aos testes em serviços de rede, mas adaptando o ataque para parâmetros HTTP como `username`, `password` e mensagens de erro retornadas pela aplicação.

A DVWA demonstra, em diferentes níveis de dificuldade, como mecanismos adicionais — por exemplo, delays de resposta, tokens CSRF e políticas de bloqueio após um número máximo de tentativas — podem mitigar ataques de força bruta em interfaces web modernas. Esse tipo de laboratório complementa os testes feitos com o Medusa em SMB, mostrando que a lógica de tentativa e erro é reaproveitada em múltiplas camadas (rede e aplicação) e reforçando a importância de combinar boas práticas de desenvolvimento seguro com configurações robustas de autenticação.


## Conclusão

O laboratório demonstrou na prática como ataques automatizados com Medusa podem identificar credenciais fracas em serviços comuns como FTP e SMB em poucos segundos. A experiência reforça a importância de combinar políticas de senha robustas, configuração segura de serviços e monitoramento contínuo para mitigar esse tipo de ameaça em ambientes reais.

Além de reproduzir a lógica de tentativa e erro usada por cibercriminosos em ataques reais, o laboratório reforça como credenciais fracas continuam sendo um vetor crítico mesmo em ambientes que já adotam MFA e soluções baseadas em Inteligência Artificial. O domínio de ferramentas clássicas como o Medusa continua essencial para auditar configurações, identificar falsos-positivos e validar, na prática, a eficácia dos controles de segurança.
