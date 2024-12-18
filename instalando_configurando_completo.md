Para configurar o site **inmemoriambrasil.com.br**, que utiliza **Bootstrap** no frontend e **Node.js** para o backend, com integração ao banco de dados **MariaDB**, você precisará passar por várias etapas, desde a configuração do servidor até o desenvolvimento e a integração do banco de dados. Abaixo está um **checklist detalhado** com as etapas essenciais para configurar o ambiente, integrar as tecnologias e garantir que o site funcione corretamente.

---

### 1. **Configuração do Servidor VPS (Debian 12)**

#### 1.1. **Acessar o servidor via SSH**
Primeiro, faça login no seu servidor VPS utilizando SSH:

```bash
ssh root@<seu_endereço_ip>
```

#### 1.2. **Atualizar o sistema**
Mantenha o sistema operacional atualizado para garantir que você tenha todos os pacotes de segurança mais recentes:

```bash
apt update && apt upgrade -y
```

#### 1.3. **Instalar pacotes básicos**
Instale pacotes essenciais para o desenvolvimento do servidor:

```bash
apt install curl git vim ufw -y
```

#### 1.4. **Configurar firewall básico (UFW)**
Configure um firewall para proteger seu servidor:

```bash
ufw allow OpenSSH        # Permite SSH
ufw allow 80/tcp         # Permite HTTP
ufw allow 443/tcp        # Permite HTTPS
ufw enable               # Ativa o firewall
ufw status               # Verifica o status do firewall
```

---

### 2. **Instalação do Node.js**

#### 2.1. **Instalar Node.js**
Primeiro, instale o **Node.js** no servidor. Você pode instalar a versão mais recente do Node.js ou usar o **nvm** (Node Version Manager) para gerenciar várias versões.

ANTIGO
```bash
curl -fsSL https://deb.nodesource.com/setup_16.x | bash -   # Exemplo para versão 16
apt install -y nodejs
```
Verificar no site a versão
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | bash - apt install -y nodejs
```
Verifique a instalação do Node.js:

```bash
node -v
npm -v
```

#### 2.2. **Instalar o NPM (se necessário)**
Se o NPM não for instalado junto com o Node.js, instale-o manualmente:

```bash
apt install npm -y
```

---

### 3. **Instalação do MariaDB**

#### 3.1. **Instalar MariaDB**
O **MariaDB** é um banco de dados relacional que é uma alternativa ao MySQL. Para instalá-lo, execute:

```bash
apt install mariadb-server mariadb-client -y
```

#### 3.2. **Configurar o MariaDB**
Após a instalação, inicie e habilite o MariaDB para iniciar automaticamente:

```bash
systemctl start mariadb
systemctl enable mariadb
```

#### 3.3. **Segurança do MariaDB**
Execute o script de segurança para proteger o MariaDB:

```bash
mysql_secure_installation
```

#### 3.4. **Criar banco de dados e usuário**
Crie o banco de dados e o usuário no MariaDB para o seu site:

```bash
mysql -u root -p
```

No prompt do MariaDB, execute os seguintes comandos para criar um banco de dados, um usuário e conceder permissões:

```sql
CREATE DATABASE inmemoriambrasil;
CREATE USER 'siteuser'@'localhost' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON inmemoriambrasil.* TO 'siteuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

### 4. **Configuração do Backend com Node.js**

#### 4.1. **Criar o diretório do projeto**
Crie um diretório para o seu projeto e inicialize o Node.js:

```bash
mkdir /var/www/inmemoriambrasil
cd /var/www/inmemoriambrasil
npm init -y
```

#### 4.2. **Instalar dependências essenciais**
Instale as dependências necessárias, como **Express.js** (framework para Node.js), **MySQL/MariaDB** e outras bibliotecas essenciais:

```bash
npm install express mysql2 dotenv body-parser
```

- **express**: Framework para criação de servidores web no Node.js.
- **mysql2**: Biblioteca para conectar o Node.js ao MariaDB.
- **dotenv**: Para carregar variáveis de ambiente (como credenciais do banco de dados).
- **body-parser**: Para interpretar os dados recebidos nas requisições HTTP.

#### 4.3. **Criar arquivo `.env` para configurações**
Crie um arquivo `.env` no diretório **raiz do projeto** para armazenar as credenciais do banco de dados e outras variáveis de ambiente:

```bash
touch .env
```

Exemplo de conteúdo do arquivo `.env`:

```
DB_HOST=localhost
DB_USER=siteuser
DB_PASSWORD=senha_segura
DB_NAME=inmemoriambrasil
```

#### 4.4. **Criar servidor Node.js com Express**
Crie um arquivo `server.js` para configurar o servidor básico e a conexão com o banco de dados:

```javascript
const express = require('express');
const mysql = require('mysql2');
const dotenv = require('dotenv');

dotenv.config();

const app = express();
const port = 3000;

const db = mysql.createConnection({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
});

app.get('/', (req, res) => {
  res.send('Bem-vindo ao In Memoriam Brasil!');
});

app.listen(port, () => {
  console.log(`Servidor rodando na porta ${port}`);
});
```

#### 4.5. **Testar o backend**
Execute o servidor Node.js:

```bash
node server.js
```

Acesse `http://<seu_endereço_ip>:3000` para testar o servidor.

---

### 5. **Instalação e Configuração do Apache (para redirecionamento de tráfego)**

#### 5.1. **Instalar o Apache**
Instale o Apache para servir o frontend:

```bash
apt install apache2 -y
```

#### 5.2. **Configurar o Apache como proxy reverso**
Edite a configuração do Apache para redirecionar o tráfego para o Node.js:

```bash
nano /etc/apache2/sites-available/000-default.conf
```

Adicione a seguinte configuração ao final:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@inmemoriambrasil.com.br
    DocumentRoot /var/www/inmemoriambrasil/public
    ProxyPass / http://localhost:3000/
    ProxyPassReverse / http://localhost:3000/
</VirtualHost>
```

Habilite os módulos necessários e reinicie o Apache:

```bash
a2enmod proxy
a2enmod proxy_http
systemctl restart apache2
```

---

### 6. **Desenvolvimento do Frontend com Bootstrap**

#### 6.1. **Baixar o Bootstrap**
No diretório `/var/www/inmemoriambrasil`, crie a estrutura do site e baixe o **Bootstrap**.

Exemplo para baixar via CDN no seu arquivo `index.html`:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>In Memoriam Brasil</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
  <div class="container">
    <h1 class="my-5">Bem-vindo ao In Memoriam Brasil</h1>
    <!-- Conteúdo do site aqui -->
  </div>
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

#### 6.2. **Integrar com o backend**
Use o **Express** para fornecer dados dinâmicos do MariaDB para o frontend. Por exemplo, você pode criar rotas que consultam o banco de dados e retornam os dados em formato JSON para o frontend.

---

### 7. **Configuração de SSL (HTTPS)**

#### 7.1. **Instalar Certbot para Let's Encrypt**
Instale e configure o **Certbot** para obter um certificado SSL gratuito:

```bash
apt install certbot python3-certbot-apache -y
certbot --apache -d inmemoriambrasil.com.br
```

---

### 8. **Testes Finais**

- **Testar o site**: Acesse `http://inmemoriambrasil.com.br` para verificar se o Apache redireciona para o Node.js.
- **Testar a integração com o banco de dados**: Faça consultas ao MariaDB através das rotas do Express para garantir que o banco de dados esteja funcionando corretamente.

---

### 9. **Backup e Manutenção**

- **Configurar backups regulares** do banco de dados e do site.
- **Monitoramento** do servidor (uso de recursos, tráfego, logs).

---

Com esse checklist, você terá configurado corretamente o **site inmemoriambrasil.com.br** usando **Bootstrap** no frontend, **Node.js** no backend, e **MariaDB** como banco de dados.
