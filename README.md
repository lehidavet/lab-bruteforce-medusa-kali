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
