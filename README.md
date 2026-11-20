# Rei das Embalagens

Site da Rei das Embalagens em Betim-MG, com catálogo, carrinho, pedidos pelo WhatsApp, cadastro/login de clientes e histórico privado de pedidos. A interface usa um único `index.html` e um único `style.css`, com o JavaScript da página dentro do HTML.

## Tecnologias

- Front-end: HTML5, CSS3 e JavaScript
- Back-end: Python 3.10+ e Flask
- Banco: SQLite (`backend/database/loja.db`)
- Segurança: bcrypt, JWT, CORS e limite de tentativas

## Instalação e execução

Requisitos: Python 3.10 ou superior.

No PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
python -m backend.database.seed
python -m backend.app
```

Antes do seed, configure `ADMIN_SENHA` no `.env` com pelo menos 12 caracteres. O seed é idempotente: cria categorias, produtos de exemplo e o administrador somente quando necessário; não apaga pedidos nem produtos existentes e não redefine senhas cadastradas.

Acesse o site em http://localhost:3000. O usuário inicial é o valor de `ADMIN_USUARIO` no `.env`. Entre como administrador, abra **Segurança** e troque a senha inicial por uma senha exclusiva antes de publicar.

## Configuração

Edite o arquivo `.env`:

- `PORT`: porta do servidor (padrão `3000`)
- `APP_ENV`: use `development` localmente e `production` na hospedagem
- `JWT_SECRET`: segredo aleatório com pelo menos 32 caracteres; obrigatório em produção
- `CORS_ALLOWED_ORIGINS`: origens permitidas separadas por vírgula; wildcard não é aceito em produção
- `ADMIN_USUARIO` e `ADMIN_SENHA`: credenciais criadas pelo seed quando ainda não existe esse usuário
- `WHATSAPP_NUMERO`: número internacional usado nos links de pedido, somente dígitos

Gere um segredo para `JWT_SECRET` com `python -c "import secrets; print(secrets.token_urlsafe(48))"`. Em produção, configure `APP_ENV=production`, um segredo próprio e as origens reais em `CORS_ALLOWED_ORIGINS`.

Os preços e estoques existentes são exemplos e precisam ser conferidos com a loja. O painel permite editá-los sem rodar novamente o seed.

## Banco e dados

O banco SQLite existente em `backend/database/loja.db` é preservado. O servidor aplica o schema e a migração de `cliente_id` sem limpar dados. O seed também não remove registros existentes.

## Área do cliente e gestão

Clientes podem criar conta, entrar pela página inicial e consultar somente os pedidos associados ao próprio cadastro. Pedidos feitos durante uma sessão de cliente são associados ao ID autenticado, nunca a um ID recebido do navegador. Pedidos sem login continuam disponíveis pelo fluxo normal do site.

As rotas administrativas exigem JWT de administrador. O painel permite consultar indicadores, editar produtos e estoque, alterar status de pedidos e trocar a senha administrativa. Produtos podem ser ocultados do catálogo e reativados sem excluí-los. Ao cancelar um pedido, os itens são devolvidos ao estoque uma única vez.

## API

Rotas públicas:

- `GET /api/empresa`
- `GET /api/categorias`
- `GET /api/produtos?categoria=&busca=&disponiveis=true&ordem=preco_asc|preco_desc|nome&pagina=&limite=`
- `GET /api/produtos/:id`
- `POST /api/pedidos`
- `POST /api/auth/login`
- `POST /api/auth/clientes/cadastro`
- `POST /api/auth/clientes/login`

Rotas protegidas com `Authorization: Bearer <token>`:

- Cliente: `GET /api/clientes/me`, `GET /api/clientes/me/pedidos`
- Administrador: `GET /api/admin/produtos`, `POST /api/auth/admin/senha`
- Administrador: `POST /api/produtos`, `PUT /api/produtos/:id`, `DELETE /api/produtos/:id`, `PATCH /api/produtos/:id/estoque`
- Administrador: `GET /api/pedidos`, `GET /api/pedidos/:id`, `PATCH /api/pedidos/:id/status`, `GET /api/dashboard`

## Publicação

Use um serviço de hospedagem que execute Python e mantenha armazenamento persistente para o SQLite. Instale dependências com `pip install -r requirements.txt` e inicie com `python -m backend.app`. Configure `APP_ENV=production`, um `JWT_SECRET` aleatório e exclusivo, `CORS_ALLOWED_ORIGINS`, `WHATSAPP_NUMERO` e `PORT`. Depois, entre no painel e altere a senha do administrador.

## Resumo

O site apresenta o catálogo e envia pedidos para uma loja de embalagens. Clientes podem criar conta, enviar pedidos e consultar somente o próprio histórico; a administração tem autenticação separada. A interface usa HTML, CSS e JavaScript; o backend usa Python, Flask e SQLite.
