Vamos configurar a segurança do servidor e garantir que os sites sejam protegidos. O foco será em HTTPS, firewall, permissões, e endurecimento do sistema. Siga os passos detalhados abaixo:

---

## 1. **Instalar Certificados SSL com Let's Encrypt**
Para proteger os sites com HTTPS, usaremos o Let's Encrypt.

1. Instale o **Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-apache -y
   ```

2. Gere os certificados SSL para ambos os sites:
   ```bash
   sudo certbot --apache -d inmemoriambrasil.com.br -d www.inmemoriambrasil.com.br
   sudo certbot --apache -d inmemoriambrasilteste.com.br -d www.inmemoriambrasilteste.com.br
   ```

3. Teste a renovação automática:
   ```bash
   sudo certbot renew --dry-run
   ```

4. Configure a renovação automática (Certbot já adiciona ao crontab, mas é bom confirmar):
   ```bash
   sudo crontab -e
   ```
   Adicione a linha:
   ```bash
   0 3 * * * /usr/bin/certbot renew --quiet
   ```

---

## 2. **Configurar o Firewall com UFW**
1. Instale o **UFW**:
   ```bash
   sudo apt install ufw -y
   ```

2. Permita conexões essenciais:
   ```bash
   sudo ufw allow OpenSSH
   sudo ufw allow 'Apache Full'
   ```

3. Ative o firewall:
   ```bash
   sudo ufw enable
   ```

4. Verifique o status:
   ```bash
   sudo ufw status
   ```

---

## 3. **Fortalecer o MariaDB**
1. Execute o script de segurança do MariaDB:
   ```bash
   sudo mysql_secure_installation
   ```
   Responda às perguntas:
   - **Set root password?** (Defina uma senha forte para o usuário root).
   - **Remove anonymous users?** Sim.
   - **Disallow root login remotely?** Sim.
   - **Remove test database?** Sim.
   - **Reload privilege tables now?** Sim.

2. Acesse o MariaDB para verificar permissões:
   ```bash
   sudo mysql
   SHOW GRANTS FOR 'user_inmemoriam'@'localhost';
   SHOW GRANTS FOR 'user_inmemoriamteste'@'localhost';
   ```

---

## 4. **Configurar Permissões no Servidor**
1. Verifique as permissões dos diretórios dos sites:
   ```bash
   sudo chmod -R 755 /var/www/inmemoriambrasil.com.br
   sudo chmod -R 755 /var/www/inmemoriambrasilteste.com.br
   sudo chown -R www-data:www-data /var/www/inmemoriambrasil.com.br
   sudo chown -R www-data:www-data /var/www/inmemoriambrasilteste.com.br
   ```

2. Bloqueie o acesso ao diretório **/etc/apache2/** e arquivos sensíveis:
   Adicione ao arquivo `/etc/apache2/apache2.conf`:
   ```apache
   <Directory /etc/apache2/>
       Require all denied
   </Directory>
   ```

3. Reinicie o Apache:
   ```bash
   sudo systemctl restart apache2
   ```

---

## 5. **Proteger o SSH**
1. Edite o arquivo de configuração do SSH:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Altere as seguintes configurações:
   ```plaintext
   PermitRootLogin no
   PasswordAuthentication no
   AllowUsers SEU_USUARIO
   ```

3. Reinicie o serviço SSH:
   ```bash
   sudo systemctl restart sshd
   ```

---

## 6. **Proteger Node.js**
1. Use um processo gerenciador como **PM2** para rodar os aplicativos Node.js:
   ```bash
   sudo npm install -g pm2
   ```

2. Inicie os aplicativos com PM2:
   ```bash
   pm2 start app.js --name "inmemoriambrasil"
   pm2 start app.js --name "inmemoriambrasilteste"
   ```

3. Configure o PM2 para iniciar na inicialização do sistema:
   ```bash
   pm2 startup
   pm2 save
   ```

---

## 7. **Desabilitar Listagem de Diretórios no Apache**
1. Edite o arquivo de configuração do Apache:
   ```bash
   sudo nano /etc/apache2/apache2.conf
   ```

2. Encontre e altere a configuração para:
   ```apache
   Options -Indexes
   ```

3. Reinicie o Apache:
   ```bash
   sudo systemctl restart apache2
   ```

---

## 8. **Monitoramento e Logs**
1. Configure logs rotativos:
   Verifique se o `logrotate` está instalado:
   ```bash
   sudo apt install logrotate -y
   ```

2. Monitore os logs regularmente:
   ```bash
   tail -f /var/log/apache2/access.log
   tail -f /var/log/apache2/error.log
   ```

---

## 9. **Configuração Adicional de Segurança**
1. **Habilitar ModSecurity (Firewall de Aplicações Web)**:
   ```bash
   sudo apt install libapache2-mod-security2 -y
   sudo a2enmod security2
   sudo systemctl restart apache2
   ```

2. **Proteger contra ataques DDoS (mod_evasive)**:
   ```bash
   sudo apt install libapache2-mod-evasive -y
   sudo nano /etc/apache2/mods-available/evasive.conf
   ```
   Configure regras básicas no arquivo:
   ```plaintext
   DOSHashTableSize 3097
   DOSPageCount 5
   DOSSiteCount 20
   DOSBlockingPeriod 10
   ```
   Ative o módulo e reinicie:
   ```bash
   sudo a2enmod evasive
   sudo systemctl restart apache2
   ```

---

Essas configurações tornam o servidor mais seguro. Você pode ajustar conforme necessário. Caso precise de mais detalhes ou suporte, é só pedir!
