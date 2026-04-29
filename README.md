# 🌱 PW-Maniva — API de Gestão para CSAs (Comunidades que Sustentam a Agricultura)

> API RESTful desenvolvida com Node.js, TypeScript, Express e MongoDB para gerenciar o ecossistema de uma CSA: propriedades rurais, cestas de produtos orgânicos, usuários e grupos locais.

---

## 📌 Sobre o Projeto

O **PW-Maniva** resolve um problema real do modelo de agricultura sustentada pela comunidade (CSA): a falta de uma plataforma centralizada para agricultores cadastrarem suas propriedades e co-agricultores gerenciarem cestas de produtos orgânicos.

O sistema implementa:
- Controle de acesso baseado em papéis (`agricultor` / `co-agricultor`)
- Validação geoespacial para evitar sobreposição de propriedades rurais
- Upload de imagens para cestas de produtos
- Busca textual completa com relevância em propriedades

---

## ✨ Funcionalidades Principais

- **Autenticação JWT** com expiração de 1 hora e proteção de rotas por papel (role-based)
- **Gestão de Propriedades** — CRUD completo com validação de área mínima (3 ha), coordenadas geoespaciais (GeoJSON) e detecção de sobreposição geográfica entre propriedades
- **Busca textual** em propriedades por nome, descrição, cultura principal e tags, com ranqueamento por relevância
- **Gestão de Cestas** — CRUD com upload de imagem via `multipart/form-data`, associadas a uma CSA específica
- **Gestão de CSAs** — Cadastro de grupos locais por cidade/estado (com restrição de duplicidade)
- **Gestão de Usuários** — Cadastro, listagem, atualização e exclusão com senhas hashadas via `bcrypt`
- **Validação de dados** com `yup` nas rotas de usuário

---

## 🛠 Tecnologias Utilizadas

| Camada | Tecnologia |
|---|---|
| Runtime | Node.js |
| Linguagem | TypeScript 5 |
| Framework | Express 5 |
| Banco de Dados | MongoDB (via MongoDB Atlas) |
| ODM | Mongoose 8 |
| Autenticação | JSON Web Token (jsonwebtoken) |
| Hash de Senhas | bcrypt |
| Upload de Arquivos | Multer |
| Geolocalização | geolib |
| Validação | Yup |
| Build | TypeScript Compiler (`tsc`) |
| Dev | ts-node, nodemon |

---

## 📁 Estrutura do Projeto

```
PW-Maniva/
├── server.ts                  # Ponto de entrada: configura Express, MongoDB e rotas
├── tsconfig.json              # Configuração do compilador TypeScript
├── package.json
│
├── middleware/
│   ├── auth.ts                # Middlewares: autenticarToken e autorizarRole
│   └── propertyValidation.ts  # Middleware de detecção de sobreposição geoespacial
│
├── models/
│   ├── usuario.ts             # Schema: nome, email, senha (hash), role, CSA vinculada
│   ├── propriedade.ts         # Schema: nome, área, localização (GeoJSON), tags, cultura
│   ├── cesta.ts               # Schema: nome, produtos, preço, CSA, imagem
│   └── csa.ts                 # Schema: cidade, estado (único por combinação)
│
├── routes/
│   ├── auth.ts                # POST /api/auth/login
│   ├── usuarios.ts            # CRUD /api/usuarios
│   ├── propriedades.ts        # CRUD + busca textual /api/propriedades
│   ├── cestas.ts              # CRUD + upload de imagem /cestas
│   └── csa.ts                 # CRUD /api/csa
│
├── uploads/                   # Imagens de cestas (geradas em runtime)
└── dist/                      # Código compilado (gerado pelo tsc)
```

---

## 🚀 Como Rodar Localmente

### Pré-requisitos

- Node.js 18+
- Conta no [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) (ou instância local do MongoDB)

### Passo a Passo

**1. Clone o repositório**
```bash
git clone https://github.com/yohannemdrs/PW-Maniva.git
cd PW-MANIVA
```

**2. Instale as dependências**
```bash
npm install
```

**3. Configure as variáveis de ambiente**

Crie um arquivo `.env` na raiz do projeto:
```env
MONGO_URI=mongodb+srv://<usuario>:<senha>@cluster.mongodb.net/<banco>
JWT_SECRET=seu-segredo-super-secreto
PORT=5000
```

**4. Compile o TypeScript**
```bash
npm run build
```

**5. Inicie o servidor**
```bash
npm start
```

A API estará disponível em `http://localhost:5000`.

> **Dica para desenvolvimento:** Use `npx nodemon --exec ts-node server.ts` para hot-reload.

---

## 📡 Exemplos de Uso da API

### 🔐 Autenticação

**Registrar usuário**
```http
POST /api/usuarios
Content-Type: application/json

{
  "nome": "João Silva",
  "email": "joao@email.com",
  "senha": "senha123",
  "role": "agricultor"
}
```

**Login**
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "joao@email.com",
  "senha": "senha123"
}
```
```json
{
  "message": "Login bem-sucedido",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6..."
}
```

> Nas rotas protegidas, envie o header: `Authorization: Bearer <token>`

---

### 🏡 Propriedades

**Criar propriedade** *(requer role: `agricultor`)*
```http
POST /api/propriedades
Authorization: Bearer <token>
Content-Type: application/json

{
  "nome": "Sítio Boa Vista",
  "descricao": "Produção orgânica de hortaliças",
  "areaHectares": 10,
  "culturaPrincipal": "Hortaliças",
  "localizacao": {
    "type": "Point",
    "coordinates": [-35.881, -7.230]
  },
  "tags": ["orgânico", "hortaliças", "agroecologia"]
}
```

**Busca textual**
```http
GET /api/propriedades/search/orgânico
```
Retorna propriedades ordenadas por relevância com base no texto buscado.

---

### 🧺 Cestas

**Criar cesta com imagem** *(requer role: `co-agricultor`)*
```http
POST /cestas
Authorization: Bearer <token>
Content-Type: multipart/form-data

nome=Cesta Verde
descricao=Legumes e folhas frescas
produtos=["alface", "cenoura", "tomate"]
preco=45.00
csa=<id_da_csa>
imagem=<arquivo_de_imagem>
```

**Buscar cesta por nome**
```http
GET /cestas/buscar/nome?nome=Verde
```

---

### 🏘 CSA

**Criar CSA**
```http
POST /api/csa
Content-Type: application/json

{
  "cidade": "Campina Grande",
  "estado": "PB"
}
```

---

## 🔮 Melhorias Futuras

- [ ] Rota para cadastro do primeiro administrador (`/primeiroAdmin`) está implementada no `dist` mas não exposta — finalizar e ativar
- [ ] Implementar testes automatizados (Jest + Supertest)
- [ ] Paginação nas rotas de listagem (`GET /api/propriedades`, `GET /cestas`)
- [ ] Busca geoespacial por proximidade — encontrar propriedades/CSAs em um raio de X km
- [ ] Endpoint para associar usuários a CSAs via API
- [ ] Armazenamento de imagens em serviço de nuvem (ex: AWS S3) em vez do disco local
- [ ] Refresh token para renovação de sessão sem novo login
- [ ] Documentação interativa com Swagger/OpenAPI

---

## 👤 Autor

Desenvolvido pelo time **LazyTeam** como projeto acadêmico da disciplina de Bancos de Dados II.

🔗 [Repositório no GitHub](https://github.com/Bancos-de-Dados-II/projeto-2-novo-lazyteam)
