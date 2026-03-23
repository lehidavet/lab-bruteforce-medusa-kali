# 🔐 Lab de Ataques de Força Bruta com Medusa e Kali Linux

[![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=flat&logo=kalilinux&logoColor=white)]()
[![Medusa](https://img.shields.io/badge/Medusa-Brute_Force-red?style=flat)]()
[![Metasploitable](https://img.shields.io/badge/Alvo-Metasploitable_2-orange?style=flat)]()
[![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-brightgreen?style=flat)]()

Este repositório documenta um laboratório prático simulando ataques de força bruta e password spraying em serviços FTP e SMB expostos em uma máquina Metasploitable 2, a partir de uma máquina atacante Kali Linux. O objetivo é entender o funcionamento desses ataques em ambiente controlado e propor medidas de mitigação.

> ⚠️ **Aviso Ético:** Este laboratório foi realizado **exclusivamente** em ambiente controlado, em máquinas sob meu próprio controle, para fins de **estudo e prática em Cibersegurança**. Nunca utilize estas técnicas em sistemas para os quais você não tenha autorização explícita.

---

## 🧱 Ambiente de Laboratório

Ambiente isolado em máquinas virtuais, sem conexão com redes externas.

- **Hypervisor:** Oracle VirtualBox
- **Rede:** Host-Only (comunicação apenas entre VMs e host)
- **Máquina Atacante:** Kali Linux — IP: `192.168.127.7` — Ferramentas: Nmap, Medusa
- **Máquina Alvo:** Metasploitable 2 — IP: `192.168.127.5` — Serviços: FTP, SMB

Scan inicial de descoberta:
```bash
nmap -sP 192.168.127.0/24
```

[Evidência nmap inicial](https://github.com/lehidavet/lab-bruteforce-medusa-kali/blob/main/images/nmap_inicial.png)

---

## 🎯 Ataque de Força Bruta em FTP com Medusa

Primeiro, confirmei o IP do alvo (`192.168.127.5`). Em seguida, executei:

```bash
medusa -h 192.168.127.5 -U wordlists/usuarios.txt -P wordlists/senhas.txt -M ftp -t 4
```

[Evidência de brute force em FTP](https://github.com/lehidavet/lab-bruteforce-medusa-kali/blob/main/images/ftp_bruteforce.png)

### Validação de Acesso com smbclient

Após identificar as credenciais válidas (`msfadmin/msfadmin`) pelo Medusa, validei o acesso ao compartilhamento SMB:

```bash
smbclient -L 192.168.127.5 -U msfadmin
smbclient //192.168.127.5/tmp -U msfadmin
```

[Evidência Validação de smbclient](https://github.com/lehidavet/lab-bruteforce-medusa-kali/blob/main/images/smbclient_validacao.png)

---

## 🔁 Password Spraying em SMB com Medusa

Scan de confirmação de portas abertas:

```bash
nmap -sV -p 21,22,80,445,139 192.168.127.5
```

[Evidência de NMAP](https://github.com/lehidavet/lab-bruteforce-medusa-kali/blob/main/images/nmap_bruteforce.png)

Ataque de password spraying:

```bash
medusa -h 192.168.127.5 -U wordlists/usuarios.txt -P senhas.txt -M smbnt -t 2 -T 50
```

[Evidência de password spraying em SMB](https://github.com/lehidavet/lab-bruteforce-medusa-kali/blob/main/images/medusa_smb.png)

Diferentemente da força bruta tradicional, no **password spraying** um pequeno conjunto de senhas comuns é testado contra vários usuários. Isso reduz a chance de disparar políticas de bloqueio de conta.

---

## 🛡️ Perspectiva Blue Team / SOC

Como um analista de SOC veria esses ataques nos logs:

| Ataque | O que aparece nos logs | Alerta gerado |
|---|---|---|
| **Nmap scan** | Múltiplas conexões TCP de um único IP em curto intervalo | Port Scan / Host Discovery Alert |
| **Brute force FTP** | Centenas de tentativas de login falhas no log FTP (`/var/log/vsftpd.log`) | Brute Force Alert / Account Lockout |
| **Password spraying SMB** | Múltiplos logins falhos de IPs diferentes ou mesmo IP em contas variadas | Anomalia de autenticação / SIEM Alert |
| **smbclient com credencial válida** | Login bem-sucedido no SMB após série de falhas | Lateral Movement / Acesso não usual |

> 💡 Identificar padrões de brute force em logs é uma das tarefas mais comuns em um SOC — e esse lab simula exatamente o que um analista precisaria detectar.

---

## 🛡️ Medidas de Mitigação

Com base nos testes realizados:

- **Fortalecer políticas de senha:** exigir senhas longas, complexas e únicas
- **Implementar bloqueio e delays:** bloquear contas temporariamente após múltiplas falhas
- **Restringir exposição de serviços:** limitar FTP e SMB a redes internas ou VPN
- **Monitorar logs de autenticação:** integrar com SIEM/IDS para detectar padrões de brute force
- **Adotar MFA:** autenticacao multifator como camada adicional de defesa

---

## 🌐 Reflexão: Força Bruta em Aplicações Web (DVWA)

Além dos testes em FTP e SMB, estudei também o módulo de Brute Force da DVWA focado em formulários de login HTTP. A DVWA demonstra como mecanismos adicionais — delays, tokens CSRF e políticas de bloqueio — podem mitigar ataques de força bruta em interfaces web modernas, reforçando a importância de combinar boas práticas de desenvolvimento seguro com configurações robustas de autenticação.

---

## 💡 O que aprendi neste lab

- Como estruturar um ambiente **ético e isolado** para testes de segurança em máquinas virtuais
- Diferença prática entre **brute force** e **password spraying** e quando cada técnica é aplicada
- Uso real do **Medusa** para testar credenciais em serviços FTP e SMB
- Como usar **Nmap** para descoberta de hosts e identificação de serviços e portas abertas
- Validação de acesso com **smbclient** após obtenção de credenciais
- Como esses ataques aparecem nos logs e quais alertas seriam gerados em um SOC
- Importância de políticas de senha robustas, bloqueio de conta, MFA e monitoramento de logs

---

## ✅ Conclusão

O laboratório demonstrou na prática como ataques automatizados com Medusa podem identificar credenciais fracas em serviços como FTP e SMB em poucos segundos. A experiência reforça a importância de combinar políticas de senha robustas, configuração segura de serviços e monitoramento contínuo para mitigar esse tipo de ameaça em ambientes reais.

---

## 💬 Autor

**Lehi Davet** – Profissional em transição para Cibersegurança | Engenheiro Mecânico | Perito Judicial

[![LinkedIn](https://img.shields.io/badge/LinkedIn-lehidavet-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/lehidavet)
[![GitHub](https://img.shields.io/badge/GitHub-lehidavet-black?style=flat&logo=github)](https://github.com/lehidavet)
