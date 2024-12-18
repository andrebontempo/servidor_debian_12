Aqui está um checklist detalhado com os comandos necessários para configurar o servidor web Debian 12 com Apache para hospedar os sites **inmemoriambrasil.com.br** e **inmemoriambrasilteste.com.br**, utilizando **Bootstrap**, **Node.js** e **MariaDB**. Configurações de segurança serão tratadas ao final, conforme solicitado.

---

## 1. **Atualizar o sistema - Execute todos os comandos como ROOT**
Antes de começar, atualize o sistema para garantir que todos os pacotes estão atualizados.
```bash
apt update && apt upgrade -y
```

---

## 2. **Instalar o Apache**
Apache será o servidor web que servirá os sites.
```bash
apt install apache2 -y
systemctl enable apache2
systemctl start apache2
```

---

## 3. **Configurar os Virtual Hosts no Apache**
Crie dois arquivos de configuração para os sites no diretório `/etc/apache2/sites-available/`.

### Para **inmemoriambrasil.com.br**
1. Crie o arquivo de configuração:
   ```bash
   nano /etc/apache2/sites-available/inmemoriambrasil.com.br.conf
   ```
2. Adicione o seguinte conteúdo:
   ```apache
   <VirtualHost *:80>
       ServerName inmemoriambrasil.com.br
       ServerAlias www.inmemoriambrasil.com.br
       DocumentRoot /var/www/inmemoriambrasil.com.br

       <Directory /var/www/inmemoriambrasil.com.br>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/inmemoriambrasil.com.br-error.log
       CustomLog ${APACHE_LOG_DIR}/inmemoriambrasil.com.br-access.log combined
   </VirtualHost>
   ```

3. Repita o processo para **inmemoriambrasilteste.com.br**:
   ```bash
   nano /etc/apache2/sites-available/inmemoriambrasilteste.com.br.conf
   ```
   Conteúdo:
   ```apache
   <VirtualHost *:80>
       ServerName inmemoriambrasilteste.com.br
       ServerAlias www.inmemoriambrasilteste.com.br
       DocumentRoot /var/www/inmemoriambrasilteste.com.br

       <Directory /var/www/inmemoriambrasilteste.com.br>
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/inmemoriambrasilteste.com.br-error.log
       CustomLog ${APACHE_LOG_DIR}/inmemoriambrasilteste.com.br-access.log combined
   </VirtualHost>
   ```

4. Ative os Virtual Hosts:
   ```bash
   a2ensite inmemoriambrasil.com.br.conf
   a2ensite inmemoriambrasilteste.com.br.conf
   ```

5. Reinicie o Apache:
   ```bash
   systemctl restart apache2
   ```

---

## 4. **Criar os diretórios dos sites**
Crie os diretórios para os arquivos dos sites.
```bash
mkdir -p /var/www/inmemoriambrasil.com.br
mkdir -p /var/www/inmemoriambrasilteste.com.br
```

Defina as permissões:
```bash
chown -R www-data:www-data /var/www/
chmod -R 755 /var/www/
```

---

## 5. **Instalar Node.js**
Instale o Node.js e o gerenciador de pacotes npm.
```bash
apt install -y nodejs npm
```

Verifique a versão instalada:
```bash
node -v
npm -v
```

---

## 6. **Configurar os sites com Bootstrap e Node.js**
1. Entre no diretório de cada site:
   ```bash
   cd /var/www/inmemoriambrasil.com.br
   ```
2. Inicie um novo projeto Node.js:
   ```bash
   npm init -y
   ```
3. Instale os pacotes necessários para seu projeto:
   ```bash
   npm install express
   npm install bootstrap
   ```

Repita para **inmemoriambrasilteste.com.br**.

---

## 7. **Instalar e Configurar o MariaDB**
1. Instale o MariaDB:
   ```bash
   apt install mariadb-server mariadb-client -y
   ```

2. Inicie e habilite o serviço:
   ```bash
   systemctl start mariadb
   systemctl enable mariadb
   ```

3. Acesse o MariaDB:
   ```bash
   mysql
   ```

4. Crie os bancos de dados e usuários para os sites:
   ```sql
   CREATE DATABASE inmemoriambrasil;
   CREATE DATABASE inmemoriambrasilteste;

   CREATE USER 'user_inmemoriam'@'localhost' IDENTIFIED BY '1234';
   CREATE USER 'user_inmemoriamteste'@'localhost' IDENTIFIED BY '1234';

   GRANT ALL PRIVILEGES ON inmemoriambrasil.* TO 'user_inmemoriam'@'localhost';
   GRANT ALL PRIVILEGES ON inmemoriambrasilteste.* TO 'user_inmemoriamteste'@'localhost';

   FLUSH PRIVILEGES;
   EXIT;
   ```

---

## 8. **Testar o Apache, Node.js e MariaDB**
1. Verifique se os sites estão funcionando no navegador acessando `http://inmemoriambrasil.com.br` e `http://inmemoriambrasilteste.com.br`.
2. Teste a conexão com o MariaDB usando Node.js.
3. Crie o arquivo teste_db.js dentro do diretório do site, como seguinte conteúdo:

```bash
const mariadb = require('mariadb');

// Configuração da conexão com o MariaDB
const pool = mariadb.createPool({
    host: 'localhost',  // ou o IP do servidor se for remoto
    user: 'seu_usuario', // substitua pelo nome do seu usuário do MariaDB
    password: 'sua_senha', // substitua pela senha do seu usuário
    database: 'nome_do_banco', // substitua pelo nome do banco de dados
    connectionLimit: 5
});

// Função para testar a conexão
async function testConnection() {
    let conn;
    try {
        conn = await pool.getConnection();
        console.log("Conexão bem-sucedida ao MariaDB!");

        // Teste de consulta simples
        const rows = await conn.query("SELECT 1 as resultado");
        console.log("Resultado da consulta:", rows[0].resultado);

        // Adicione consultas adicionais se desejar
    } catch (err) {
        console.error("Erro ao conectar ao MariaDB:", err);
    } finally {
        if (conn) conn.end(); // Fecha a conexão
    }
}

// Executa o teste de conexão
testConnection();
```
4 - Execute o script:
```bash
   node teste_db.js
   ```
4 - Caso dê erro, execute:
npm list mariadb
npm install mariadb
Teste novamente
node teste_db.js

---

### **Próximos passos**
Após concluir a configuração básica, você pode proceder com as configurações de segurança (por exemplo, instalar certificados SSL com Let's Encrypt, configurar UFW, etc.). Informe-me quando quiser continuar!
