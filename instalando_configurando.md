Configurar um **VPS** (Virtual Private Server) com **Debian 12**, incluindo a instalação e configuração do **Apache** e a implementação de medidas de segurança, envolve diversas etapas. Vou fornecer um **checklist passo a passo** para você configurar seu servidor corretamente. Essa lista abrange desde a configuração básica do servidor até práticas recomendadas de segurança.

---

### 1. **Configurações iniciais do servidor**

#### 1.1. **Acesse seu servidor via SSH**
Antes de mais nada, acesse seu servidor VPS via SSH. Caso esteja usando o terminal:

```bash
ssh root@<seu_endereço_ip>
```

**Dica**: Se você não estiver logado como `root`, use `sudo` para executar os comandos com permissões administrativas.

#### 1.2. **Atualize o sistema**
Primeiro, atualize o sistema para garantir que você tenha os pacotes mais recentes de segurança.

```bash
apt update && apt upgrade -y
```

#### 1.3. **Crie um usuário não-root**
Evite usar a conta `root` para administração diária. Crie um novo usuário com privilégios de sudo:

```bash
adduser meu_usuario
usermod -aG sudo meu_usuario
```

#### 1.4. **Configuração de firewall básico (UFW)**
O **UFW** (Uncomplicated Firewall) é uma ferramenta simples para gerenciar firewall. Habilite e configure regras básicas.

```bash
apt install ufw
ufw allow OpenSSH        # Permite acesso SSH
ufw allow 80/tcp         # Permite tráfego HTTP
ufw allow 443/tcp        # Permite tráfego HTTPS
ufw enable               # Ativa o firewall
ufw status               # Verifica o status do firewall
```

---

### 2. **Instalação e configuração do Apache**

#### 2.1. **Instalar o Apache**
Instale o Apache no seu servidor:

```bash
apt install apache2 -y
```

#### 2.2. **Habilitar e iniciar o Apache**
Garanta que o Apache inicie automaticamente após o boot e que esteja funcionando:

```bash
systemctl enable apache2
systemctl start apache2
systemctl status apache2   # Verifica se o Apache está ativo
```

#### 2.3. **Verificar o Apache**
Abra seu navegador e acesse o IP do servidor (ex: `http://<seu_endereço_ip>`). Se tudo estiver correto, você verá a página padrão do Apache.

---

### 3. **Configurações de segurança do Apache**

#### 3.1. **Desabilitar módulos e recursos desnecessários**
Para melhorar a segurança, desative módulos que não são necessários.

```bash
a2dismod status
a2dismod autoindex
systemctl restart apache2
```

#### 3.2. **Configuração de diretórios e permissões**
Defina permissões adequadas para os diretórios do Apache para limitar o acesso a arquivos sensíveis.

```bash
chown -R www-data:www-data /var/www/html
chmod -R 755 /var/www/html
```

#### 3.3. **Habilitar SSL (HTTPS) com Let's Encrypt**
Use **Certbot** para instalar um certificado SSL gratuito do **Let's Encrypt**.

```bash
apt install certbot python3-certbot-apache -y
certbot --apache -d seu_dominio.com -d www.seu_dominio.com
```

Certifique-se de que o Apache está configurado para redirecionar o tráfego HTTP para HTTPS. O Certbot configura isso automaticamente.

---

### 4. **Configurações adicionais de segurança**

#### 4.1. **Desabilitar acesso SSH para o root**
Para aumentar a segurança, desabilite o acesso SSH direto para o usuário `root`.

1. Edite o arquivo de configuração do SSH:
   ```bash
   nano /etc/ssh/sshd_config
   ```

2. Modifique a linha:
   ```bash
   PermitRootLogin no
   ```

3. Reinicie o serviço SSH:
   ```bash
   systemctl restart sshd
   ```

#### 4.2. **Configurar autenticação de chave SSH (opcional)**
Se ainda não estiver utilizando, configure a autenticação por chave SSH para melhorar a segurança:

1. Gere uma chave SSH no seu computador local (se ainda não tiver):
   ```bash
   ssh-keygen
   ```

2. Copie a chave pública para o servidor:
   ```bash
   ssh-copy-id meu_usuario@<seu_endereço_ip>
   ```

3. Após configurar, você pode desabilitar a autenticação por senha no servidor, editando o arquivo `/etc/ssh/sshd_config`:
   ```bash
   PasswordAuthentication no
   ```

4. Reinicie o SSH:
   ```bash
   systemctl restart sshd
   ```

#### 4.3. **Instalar Fail2ban**
O **Fail2ban** ajuda a proteger seu servidor contra ataques de força bruta, como tentativas excessivas de login SSH.

```bash
apt install fail2ban -y
systemctl enable fail2ban
systemctl start fail2ban
```

---

### 5. **Outras boas práticas de segurança**

#### 5.1. **Instalar o Lynis (Auditoria de segurança)**
**Lynis** é uma ferramenta de auditoria de segurança para servidores Linux. Ela pode identificar vulnerabilidades e sugerir melhorias.

```bash
apt install lynis -y
lynis audit system
```

#### 5.2. **Monitorar o servidor com ferramentas como o `top` ou `htop`**
Use ferramentas de monitoramento como `top` ou `htop` para acompanhar o uso de recursos do servidor.

```bash
apt install htop
htop
```

#### 5.3. **Instalar e configurar o Logwatch**
O **Logwatch** permite monitorar os logs do servidor e receber relatórios diários sobre qualquer atividade suspeita.

```bash
apt install logwatch -y
```

#### 5.4. **Configurar backups regulares**
Garanta que você tenha backups regulares dos seus dados e da configuração do servidor. Você pode usar ferramentas como `rsync`, `duplicity` ou serviços de backup baseados em nuvem.

---

### 6. **Manutenção e atualizações regulares**

#### 6.1. **Habilitar atualizações automáticas**
Habilite as atualizações automáticas para manter seu servidor seguro com os últimos patches de segurança.

```bash
apt install unattended-upgrades -y
dpkg-reconfigure --priority=low unattended-upgrades
```

#### 6.2. **Revisar logs e auditorias**
Verifique regularmente os logs de sistema, Apache e outros para detectar quaisquer atividades suspeitas:

- Logs do Apache: `/var/log/apache2/access.log` e `/var/log/apache2/error.log`
- Logs do sistema: `/var/log/syslog`, `/var/log/auth.log`

---

### Conclusão
Com esse checklist, você terá configurado seu **servidor Debian 12** com o **Apache**, **segurança básica** e boas práticas de administração. Lembre-se de que a segurança de um servidor é um processo contínuo e sempre é importante manter o sistema e as aplicações atualizados, além de monitorar regularmente os logs e a atividade do servidor.

Se precisar de mais detalhes sobre qualquer um desses passos ou configurações específicas, fique à vontade para perguntar!
