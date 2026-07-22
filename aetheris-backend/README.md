# Aetheris — Fichas com Login e Banco de Dados

Sistema completo: servidor Node.js (Express) + banco de dados SQLite + login real (usuário/senha com senha criptografada) + a ficha interativa de Aetheris servindo como frontend.

## Estrutura

```
aetheris-backend/
├── server.js          → servidor (API + serve o frontend)
├── package.json        → dependências
└── public/
    └── index.html      → a ficha interativa (com tela de login)
```

## Rodando localmente (pra testar)

Você precisa ter o [Node.js](https://nodejs.org) instalado (versão 18 ou mais recente).

```bash
cd aetheris-backend
npm install
npm start
```

Abra `http://localhost:3000` no navegador. Crie uma conta, faça login, preencha uma ficha e salve — ela fica gravada no arquivo `aetheris.db` (banco SQLite) dentro dessa mesma pasta.

## Como funciona o login

- Senhas nunca são guardadas em texto puro — são criptografadas com `bcrypt` antes de ir pro banco.
- Ao logar, o servidor manda um cookie de sessão (`httpOnly`, não acessível via JavaScript do navegador, o que ajuda a proteger contra roubo de sessão).
- Cada ficha salva fica vinculada ao `user_id` de quem criou — um jogador nunca vê ou edita a ficha de outro, mesmo sabendo o link do site.

## Hospedando de verdade (pra jogadores acessarem de qualquer lugar)

Você precisa de um serviço que rode Node.js continuamente. Algumas opções simples e com plano gratuito/barato:

### Opção recomendada: Render.com
1. Suba essa pasta (`aetheris-backend`) num repositório do GitHub.
2. Em [render.com](https://render.com), crie um **Web Service** novo, conecte o repositório.
3. Build command: `npm install` — Start command: `npm start`.
4. **Importante:** o SQLite salva num arquivo local (`aetheris.db`). Nos planos gratuitos, o disco é apagado a cada novo deploy. Pra persistir de verdade, adicione um **Persistent Disk** nas configurações do serviço (Render oferece isso nos planos pagos) e aponte o caminho do banco pra esse disco.
5. Defina a variável de ambiente `JWT_SECRET` com uma string aleatória longa (não deixe o valor padrão do código em produção).

### Alternativa: Railway.app ou Fly.io
Funcionam de forma parecida — suportam "volumes" persistentes pra guardar o arquivo SQLite entre deploys.

### Se quiser trocar SQLite por um banco gerenciado (mais robusto)
Serviços como **Supabase** ou **Neon** oferecem Postgres gratuito e persistente sem se preocupar com disco. Isso exigiria trocar as poucas linhas de `better-sqlite3` no `server.js` por um cliente Postgres (ex: `pg`) — a lógica de rotas e autenticação continua igual, só a camada de banco muda. Posso montar essa versão se você decidir ir por esse caminho no futuro.

## Variáveis de ambiente importantes

| Variável | Para que serve | Padrão |
|---|---|---|
| `PORT` | Porta do servidor | 3000 |
| `JWT_SECRET` | Chave usada pra assinar o token de sessão — troque em produção! | valor de exemplo no código |
| `NODE_ENV` | Se `production`, ativa cookie seguro (exige HTTPS) | não definido |

## Limitações desta versão

- Não tem recuperação de senha por e-mail (seria um próximo passo, exigindo um serviço de envio de e-mail).
- Não tem "esqueci minha senha" nem verificação de e-mail — é o mínimo funcional de login real.
- Se hospedar sem disco persistente, o banco de dados zera a cada novo deploy — leia a seção de hospedagem acima.
