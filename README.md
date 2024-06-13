## Sistema de Gerenciamento de Chamados para RH/DP

### Descrição do Projeto

Este projeto é um sistema de gerenciamento de chamados para o departamento de Recursos Humanos (RH) e Departamento Pessoal (DP). Permite que funcionários abram chamados para solicitar informações, esclarecer dúvidas ou solicitar serviços relacionados a RH/DP.

### Funcionalidades Principais

- **Abertura de Chamados**: Funcionários podem abrir chamados categorizados.
- **Acompanhamento e Notificações**: Funcionários recebem notificações sobre atualizações em seus chamados.
- **Centralização de Documentos**: Possibilidade de anexar documentos relevantes aos chamados.
- **Chat Integrado**: Funcionários podem iniciar conversas em tempo real com membros do RH/DP.
- **Relatórios e Métricas**: Geração de relatórios sobre volume de chamados e tempo médio de resposta.

### Tecnologias Utilizadas

### Backend
- Node.js
- Express.js
- Sequelize ORM
- SQL Server
- Docker

### Frontend
- React.js

## Como Instalar e Executar Localmente

1. Certifique-se de ter o Node.js e o Docker instalados.
2. Clone este repositório: `git clone https://github.com/seu-usuario/ticket-management-system.git`
3. Navegue até o diretório `backend` e instale as dependências: `npm install`
4. Navegue até o diretório `frontend` e instale as dependências: `npm install`
5. Inicie o servidor backend: `npm run dev` (no diretório `backend`)
6. Inicie o servidor de desenvolvimento do frontend: `npm start` (no diretório `frontend`)

## Documentação da Licença

Este projeto está sujeito a uma licença de compra. Para mais informações sobre a licença e para adquirir uma licença, entre em contato com:

- **Nome do Contato**: [Igor Hilario]
- **Email**: [igorsilva.mg@outlook.com]

Retirar após a consturação

Estrutura do Projeto
O projeto será organizado da seguinte forma:

plaintext
Copy code
ticket-management-system/
├── backend/
│   ├── config/
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   ├── services/
│   ├── uploads/
│   ├── app.js
│   ├── package.json
│   └── Dockerfile
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── App.js
│   │   ├── index.js
│   │   └── styles.css
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
└── README.md
Backend (Node.js com Express e SQL Server)
1. Configuração do SQL Server com Docker
Crie o arquivo docker-compose.yml na raiz do projeto:

yaml
Copy code
version: '3.7'
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2019-latest
    container_name: sqlserver
    environment:
      SA_PASSWORD: "YourStrong!Passw0rd"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
    volumes:
      - sqlserverdata:/var/opt/mssql
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    volumes:
      - ./backend:/app
    depends_on:
      - sqlserver
    environment:
      - DB_SERVER=sqlserver
      - DB_USER=sa
      - DB_PASSWORD=YourStrong!Passw0rd
      - DB_NAME=TicketDB
  frontend:
    build: ./frontend
    ports:
      - "3001:3000"
    volumes:
      - ./frontend:/app
    depends_on:
      - backend
volumes:
  sqlserverdata:
2. Estrutura do Backend
a. Configuração
Crie o arquivo backend/config/dbConfig.js:

javascript
Copy code
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize('TicketDB', 'sa', 'YourStrong!Passw0rd', {
  host: 'sqlserver',
  dialect: 'mssql',
  logging: false,
});

const connectDB = async () => {
  try {
    await sequelize.authenticate();
    console.log('Connection has been established successfully.');
  } catch (error) {
    console.error('Unable to connect to the database:', error);
  }
};

module.exports = { sequelize, connectDB };
b. Modelos
Crie o arquivo backend/models/Ticket.js:

javascript
Copy code
const { Sequelize, DataTypes } = require('sequelize');
const { sequelize } = require('../config/dbConfig');

const Ticket = sequelize.define('Ticket', {
  id: {
    type: DataTypes.INTEGER,
    autoIncrement: true,
    primaryKey: true,
  },
  title: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  description: {
    type: DataTypes.TEXT,
    allowNull: false,
  },
  category: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  status: {
    type: DataTypes.STRING,
    defaultValue: 'open',
  },
}, {
  timestamps: true,
});

module.exports = Ticket;
c. Controladores
Crie o arquivo backend/controllers/ticketController.js:

javascript
Copy code
const Ticket = require('../models/Ticket');

const createTicket = async (req, res) => {
  try {
    const { title, description, category } = req.body;
    const ticket = await Ticket.create({ title, description, category });
    res.status(201).json(ticket);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

const getTickets = async (req, res) => {
  try {
    const tickets = await Ticket.findAll();
    res.status(200).json(tickets);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

module.exports = { createTicket, getTickets };
d. Rotas
Crie o arquivo backend/routes/ticketRoutes.js:

javascript
Copy code
const express = require('express');
const { createTicket, getTickets } = require('../controllers/ticketController');

const router = express.Router();

router.post('/tickets', createTicket);
router.get('/tickets', getTickets);

module.exports = router;
e. Configuração do Servidor
Crie o arquivo backend/app.js:

javascript
Copy code
const express = require('express');
const bodyParser = require('body-parser');
const { connectDB } = require('./config/dbConfig');
const ticketRoutes = require('./routes/ticketRoutes');

const app = express();

app.use(bodyParser.json());
app.use('/api', ticketRoutes);

const PORT = process.env.PORT || 3000;

const startServer = async () => {
  await connectDB();
  app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
  });
};

startServer();
f. Dockerfile do Backend
Crie o arquivo backend/Dockerfile:

Dockerfile
Copy code
FROM node:14

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm install

COPY . .

CMD ["npm", "run", "dev"]
Frontend (React.js)
1. Estrutura do Frontend
a. Configuração Inicial
Crie um novo projeto React no diretório frontend:

bash
Copy code
npx create-react-app frontend
b. Componentes
Crie os componentes necessários em frontend/src/components.

i. Componente para Criar Chamado (CreateTicket.js):
javascript
Copy code
import React, { useState } from 'react';
import axios from 'axios';

const CreateTicket = () => {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const [category, setCategory] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await axios.post('http://localhost:3000/api/tickets', {
        title,
        description,
        category,
      });
      console.log(response.data);
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Title:</label>
        <input type="text" value={title} onChange={(e) => setTitle(e.target.value)} required />
      </div>
      <div>
        <label>Description:</label>
        <textarea value={description} onChange={(e) => setDescription(e.target.value)} required />
      </div>
      <div>
        <label>Category:</label>
        <input type="text" value={category} onChange={(e) => setCategory(e.target.value)} required />
      </div>
      <button type="submit">Create Ticket</button>
    </form>
  );
};

export default CreateTicket;
ii. Componente para Listar Chamados (TicketList.js):
javascript
Copy code
import React, { useEffect, useState } from 'react';
import axios from 'axios';

const TicketList = () => {
  const [tickets, setTickets] = useState([]);

  useEffect(() => {
    const fetchTickets = async () => {
      try {
        const response = await axios.get('http://localhost:3000/api/tickets');
        setTickets(response.data);
      } catch (error) {
        console.error(error);
      }
    };

    fetchTickets();
  }, []);

  return (
    <div>
      <h2>Ticket List</h2>
      <ul>
        {tickets.map((ticket) => (
          <li key={ticket.id}>
            <h3>{ticket.title}</h3>
            <p>{ticket.description}</p>
            <p>{ticket.category}</p>
            <p>Status: {ticket.status}</p>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TicketList;
c. Páginas
Crie a página principal em frontend/src/pages.

i. Página Inicial (Home.js):
javascript
Copy code
import React from 'react';
import CreateTicket from '../components/CreateTicket';
import TicketList from '../components/TicketList';

const Home = () => {
  return (
    <div>
      <h1>Chamados RH/DP</h1>
      <CreateTicket />
      <TicketList />
    </div>
  );
};

export default Home;
d. Configuração do App
Modifique frontend/src/App.js para incluir a página inicial:

javascript
Copy code
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Home from './pages/Home';

const App = () => {
  return (
    <Router>
      <div>
        <Switch>
          <Route exact path="/" component={Home} />
        </Switch>
      </div>
    </Router>
  );
};

export default App;
``
Terminou?
Para mim parou nesta parte: d. Configuração do App
Modifique frontend/src/App.js para incluir a página inicial:
ChatGPT
Ah, entendi. Aqui está a continuação do arquivo frontend/src/App.js com a inclusão da página inicial:

javascript
Copy code
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Home from './pages/Home';

const App = () => {
  return (
    <Router>
      <div>
        <Switch>
          <Route exact path="/" component={Home} />
          {/* Adicione outras rotas aqui, se necessário */}
        </Switch>
      </div>
    </Router>
  );
};

export default App;
