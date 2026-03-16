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

## Ataque de Força Bruta em FTP com Medusa

Para identificar senhas fracas no serviço FTP da máquina Metasploitable 2, utilizei o Medusa a partir do Kali Linux.

Primeiro, confirmei o IP do alvo (`192.168.127.5`) com um scan na rede `192.168.127.0/24` usando o Nmap. Em seguida, executei o seguinte comando:

```bash
medusa -h 192.168.127.5 -U wordlists/usuarios.txt -P wordlists/senhas.txt -M ftp -t 4
```

![Evidência de brute force em FTP](images/ftp_bruteforce.png)

## Password Spraying em SMB com Medusa

Para simular um ataque de password spraying contra o serviço SMB da máquina Metasploitable 2, reutilizei a mesma lista de usuários e defini uma lista de senhas comuns (`senhas.txt`) para serem testadas em todos os logins.

Primeiro, confirmei que as portas 21, 22, 80, 139 e 445 estavam abertas no alvo com o Nmap:

```bash
nmap -sV -p 21,22,80,445,139 192.168.127.5
```
![Evidência de NMAP](images/nmap_bruteforce.png)

Em seguida, executei o seguinte comando com o módulo `smbnt` do Medusa:

```bash
medusa -h 192.168.127.5 -U wordlists/usuarios.txt -P senhas.txt -M smbnt -t 2 -T 50
```

![Evidência de password spraying em SMB](images/medusa_smb.png)
