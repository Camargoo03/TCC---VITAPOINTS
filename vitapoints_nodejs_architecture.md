# VitaPoints - Arquitetura do Back-end Node.js com MySQL

## Visão Geral
O VitaPoints será desenvolvido com back-end em Node.js e banco de dados MySQL, mantendo as mesmas funcionalidades do sistema de doação de sangue com recompensas.

## Stack Tecnológica

### Back-end
- **Node.js** - Runtime JavaScript
- **Express.js** - Framework web
- **MySQL** - Banco de dados relacional
- **mysql2** - Driver MySQL para Node.js
- **bcryptjs** - Hash de senhas
- **express-session** - Gerenciamento de sessões
- **cors** - Suporte a CORS
- **dotenv** - Variáveis de ambiente

### Front-end (mantido)
- **HTML5, CSS3, JavaScript**
- **Tailwind CSS**
- **Font Awesome**

## Estrutura do Projeto Node.js

```
vitapoints-nodejs/
├── src/
│   ├── config/
│   │   └── database.js          # Configuração do MySQL
│   ├── models/
│   │   ├── User.js              # Modelo de usuário
│   │   ├── Donation.js          # Modelo de doação
│   │   ├── BloodStock.js        # Modelo de estoque
│   │   ├── Reward.js            # Modelo de recompensa
│   │   ├── RewardRedemption.js  # Modelo de resgate
│   │   ├── Appointment.js       # Modelo de agendamento
│   │   └── Partner.js           # Modelo de parceiro
│   ├── routes/
│   │   ├── auth.js              # Rotas de autenticação
│   │   ├── users.js             # Rotas de usuários
│   │   ├── bloodStock.js        # Rotas de estoque
│   │   ├── rewards.js           # Rotas de recompensas
│   │   ├── ranking.js           # Rotas de ranking
│   │   ├── appointments.js      # Rotas de agendamentos
│   │   └── partners.js          # Rotas de parceiros
│   ├── middleware/
│   │   ├── auth.js              # Middleware de autenticação
│   │   └── validation.js        # Middleware de validação
│   ├── utils/
│   │   ├── database.js          # Utilitários do banco
│   │   └── helpers.js           # Funções auxiliares
│   └── public/                  # Arquivos estáticos (front-end)
├── database/
│   └── schema.sql               # Script de criação do banco
├── .env                         # Variáveis de ambiente
├── package.json                 # Dependências Node.js
├── server.js                    # Arquivo principal
└── README.md                    # Documentação
```

## Esquema do Banco de Dados MySQL

### Tabela: users
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    points INT DEFAULT 0,
    total_donations INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_admin BOOLEAN DEFAULT FALSE
);
```

### Tabela: donations
```sql
CREATE TABLE donations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    blood_type VARCHAR(5) NOT NULL,
    donation_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    points_earned INT DEFAULT 100,
    confirmed BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### Tabela: blood_stock
```sql
CREATE TABLE blood_stock (
    id INT AUTO_INCREMENT PRIMARY KEY,
    blood_type VARCHAR(5) UNIQUE NOT NULL,
    current_level INT DEFAULT 0,
    max_level INT DEFAULT 100,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### Tabela: partners
```sql
CREATE TABLE partners (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    logo_url VARCHAR(255),
    website VARCHAR(255),
    active BOOLEAN DEFAULT TRUE
);
```

### Tabela: rewards
```sql
CREATE TABLE rewards (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    points_cost INT NOT NULL,
    category VARCHAR(50),
    partner_id INT,
    available BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (partner_id) REFERENCES partners(id)
);
```

### Tabela: reward_redemptions
```sql
CREATE TABLE reward_redemptions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    reward_id INT NOT NULL,
    points_spent INT NOT NULL,
    redeemed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'completed') DEFAULT 'pending',
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (reward_id) REFERENCES rewards(id)
);
```

### Tabela: appointments
```sql
CREATE TABLE appointments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    appointment_date DATETIME NOT NULL,
    status ENUM('scheduled', 'completed', 'cancelled') DEFAULT 'scheduled',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## API Endpoints (Node.js/Express)

### Autenticação
- `POST /api/auth/register` - Cadastro de usuário
- `POST /api/auth/login` - Login
- `POST /api/auth/logout` - Logout
- `GET /api/auth/profile` - Perfil do usuário
- `GET /api/auth/check` - Verificar autenticação

### Usuários
- `GET /api/users/dashboard` - Dashboard do usuário
- `GET /api/users/points` - Pontos do usuário
- `GET /api/users/history` - Histórico de doações
- `PUT /api/users/profile` - Atualizar perfil

### Estoque de Sangue
- `GET /api/blood-stock` - Estoque atual de sangue
- `PUT /api/blood-stock/:type` - Atualizar estoque (admin)
- `GET /api/blood-stock/critical` - Estoque crítico

### Recompensas
- `GET /api/rewards` - Lista de recompensas disponíveis
- `POST /api/rewards/redeem` - Resgatar recompensa
- `GET /api/rewards/history` - Histórico de resgates
- `GET /api/rewards/categories` - Categorias de recompensas

### Ranking
- `GET /api/ranking` - Ranking geral
- `GET /api/ranking/monthly` - Ranking mensal
- `GET /api/ranking/top3` - Top 3 doadores
- `GET /api/ranking/stats` - Estatísticas gerais

### Agendamentos
- `GET /api/appointments/available` - Horários disponíveis
- `POST /api/appointments` - Criar agendamento
- `GET /api/appointments/user` - Agendamentos do usuário
- `PUT /api/appointments/:id/confirm` - Confirmar doação (admin)
- `PUT /api/appointments/:id/cancel` - Cancelar agendamento

### Parceiros
- `GET /api/partners` - Lista de parceiros
- `GET /api/partners/:id` - Detalhes do parceiro

## Configuração do Ambiente

### Variáveis de Ambiente (.env)
```env
# Servidor
PORT=3000
NODE_ENV=development

# Banco de Dados MySQL
DB_HOST=localhost
DB_PORT=3306
DB_NAME=vitapoints
DB_USER=root
DB_PASSWORD=senha_mysql

# Sessão
SESSION_SECRET=vitapoints_secret_key_2023

# CORS
CORS_ORIGIN=http://localhost:3000
```

### Dependências (package.json)
```json
{
  "name": "vitapoints-backend",
  "version": "1.0.0",
  "description": "Sistema de doação de sangue com recompensas",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "init-db": "node src/utils/initDatabase.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mysql2": "^3.6.0",
    "bcryptjs": "^2.4.3",
    "express-session": "^1.17.3",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "express-validator": "^7.0.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

## Segurança e Boas Práticas

### Autenticação
- Hash de senhas com bcryptjs
- Sessões seguras com express-session
- Validação de entrada com express-validator
- Middleware de autenticação para rotas protegidas

### Banco de Dados
- Prepared statements para prevenir SQL injection
- Validação de tipos de dados
- Índices para otimização de consultas
- Backup automático (produção)

### API
- CORS configurado adequadamente
- Rate limiting (produção)
- Logs de auditoria
- Tratamento de erros padronizado

## Dados Iniciais

### Tipos Sanguíneos
```sql
INSERT INTO blood_stock (blood_type, current_level, max_level) VALUES
('A+', 75, 100),
('A-', 15, 100),
('B+', 60, 100),
('B-', 45, 100),
('AB+', 30, 100),
('AB-', 20, 100),
('O+', 80, 100),
('O-', 25, 100);
```

### Parceiros
```sql
INSERT INTO partners (name, description, website) VALUES
('Cinema Bebedouro', 'Rede de cinemas com descontos especiais', 'https://cinemabebedouro.com.br'),
('Loja do Doador', 'Loja de produtos com vale-compras', 'https://lojadodoador.com.br'),
('Café Central', 'Cafeteria no centro da cidade', 'https://cafecentral.com.br');
```

### Recompensas
```sql
INSERT INTO rewards (name, description, points_cost, category, partner_id) VALUES
('Vale-compras R$50', 'Use em lojas parceiras', 500, 'Vale-compras', 2),
('Ingresso para Cinema', 'Válido para qualquer filme', 600, 'Entretenimento', 1),
('Brinde Exclusivo', 'Caneca personalizada VitaPoints', 400, 'Brinde', NULL),
('Café Grátis', 'Um café grande na cafeteria', 300, 'Alimentação', 3),
('Desconto em Curso Online', '50% de desconto em cursos', 700, 'Educação', NULL);
```

### Usuários de Teste
```sql
INSERT INTO users (name, email, password_hash, points, total_donations, is_admin) VALUES
('Administrador', 'admin@vitapoints.com.br', '$2a$10$hash_da_senha', 0, 0, TRUE),
('Carlos Eduardo M.', 'carlos@email.com', '$2a$10$hash_da_senha', 2400, 24, FALSE),
('Ana Carolina S.', 'ana@email.com', '$2a$10$hash_da_senha', 1800, 18, FALSE),
('Roberto Alencar', 'roberto@email.com', '$2a$10$hash_da_senha', 1500, 15, FALSE);
```

## Próximos Passos

1. **Configurar ambiente Node.js**
2. **Instalar dependências**
3. **Configurar conexão MySQL**
4. **Implementar modelos de dados**
5. **Criar rotas da API**
6. **Implementar middleware de autenticação**
7. **Testar endpoints**
8. **Integrar com front-end**
9. **Testes finais**
10. **Documentação e entrega**
