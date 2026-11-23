# Agendamento de Visitas — CENPALEO

Sistema web para solicitar, organizar e acompanhar visitas ao CENPALEO — Centro Paleontológico da Universidade do Contestado.

Esta é a branch `notificacoes-email`, que inclui o envio de mensagens de acompanhamento ao visitante.

## Funcionalidades

- calendário público com disponibilidade atualizada por data e turno;
- solicitação de visita pela manhã, à tarde ou nos dois turnos;
- prevenção de agendamentos e bloqueios conflitantes;
- painel administrativo responsivo com calendário, filtros e turnos bloqueados;
- confirmação interna antes de cancelar agendamentos ou remover bloqueios;
- registro obrigatório do motivo de negação ou cancelamento;
- notificações por e-mail para solicitação recebida, confirmação, negação e cancelamento;
- autenticação administrativa configurada por variáveis de ambiente.

## Tecnologias e dependências

### Requisitos do ambiente

| Ferramenta | Versão |
| --- | --- |
| PHP | 8.3 ou superior |
| Composer | 2.x |
| Node.js | 20.19 ou superior, ou 22.12 ou superior |
| npm | Compatível com a versão instalada do Node.js |
| Banco de dados | SQLite por padrão; PostgreSQL também pode ser configurado |

O PHP precisa ter as extensões usuais do Laravel habilitadas, incluindo `ctype`, `dom`, `fileinfo`, `filter`, `iconv`, `json`, `libxml`, `openssl`, `pcre`, `session`, `tokenizer`, `xml` e `xmlwriter`. Para SQLite, habilite também `pdo_sqlite`.

### Principais dependências

| Camada | Pacotes |
| --- | --- |
| Backend | Laravel 13, Laravel Tinker 3 |
| Frontend | Vite 8, Tailwind CSS 4, plugin Vite do Laravel 3 |
| Desenvolvimento e testes | Laravel Boost 2, Laravel Pint 1, PHPUnit 12, Mockery 1, Faker 1 |

As versões e a lista completas ficam em `composer.json`, `composer.lock`, `package.json` e `package-lock.json`.

## Instalação

Clone o fork e selecione esta branch:

```bash
git clone https://github.com/stela-oliveira/agendamento_cenpaleo.git
cd agendamento_cenpaleo
git checkout notificacoes-email
```

Crie o arquivo do banco SQLite caso ele ainda não exista.

No Windows PowerShell:

```powershell
New-Item database/database.sqlite -ItemType File -Force
```

No Linux ou macOS:

```bash
touch database/database.sqlite
```

Execute a preparação automatizada. O comando instala as dependências PHP e JavaScript, cria o `.env`, gera a chave da aplicação, executa as migrações e compila o frontend:

```bash
composer run setup
```

Se preferir executar manualmente:

```bash
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
npm install
npm run build
```

No Windows PowerShell, substitua `cp .env.example .env` por:

```powershell
Copy-Item .env.example .env
```

## Configuração

Edite o arquivo `.env`. Para desenvolvimento local com SQLite, mantenha:

```dotenv
APP_URL=http://localhost:8000
DB_CONNECTION=sqlite
SESSION_DRIVER=file
CACHE_STORE=file
QUEUE_CONNECTION=database
```

Defina credenciais administrativas próprias, principalmente em produção:

```dotenv
ADMIN_USERNAME=admin
ADMIN_PASSWORD=uma-senha-forte
```

Nunca envie o arquivo `.env` ao Git nem publique senhas reais no README.

## Configuração de e-mail

### Teste local sem enviar mensagens reais

O modo `log` grava o conteúdo das mensagens em `storage/logs/laravel.log`:

```dotenv
MAIL_MAILER=log
MAIL_FROM_ADDRESS="cenpaleo@unc.br"
MAIL_FROM_NAME="CENPALEO"
QUEUE_CONNECTION=database
```

### Envio por SMTP

Solicite ao provedor ou à universidade o host, a porta e o método de segurança corretos. Um exemplo com SMTPS na porta 465 é:

```dotenv
MAIL_MAILER=smtp
MAIL_SCHEME=smtps
MAIL_HOST=smtp.seu-provedor.com
MAIL_PORT=465
MAIL_USERNAME=seu-usuario
MAIL_PASSWORD="sua-senha-ou-senha-de-aplicativo"
MAIL_FROM_ADDRESS=seu-email@dominio.com
MAIL_FROM_NAME="CENPALEO"
QUEUE_CONNECTION=database
```

Para provedores que usam STARTTLS na porta 587, normalmente use `MAIL_SCHEME=smtp` e `MAIL_PORT=587`. Confirme essa configuração com o provedor. Depois de alterar o `.env`, limpe o cache:

```bash
php artisan optimize:clear
```

O endereço em `MAIL_FROM_ADDRESS` precisa ser autorizado pelo provedor. Contas institucionais também podem exigir senha de aplicativo, autenticação em duas etapas ou liberação do SMTP pelo administrador.

## Executando o projeto

O comando recomendado inicia o servidor, o Vite e o processador da fila usados no desenvolvimento:

```bash
composer run dev
```

Acesse:

- site público: `http://localhost:8000`;
- administração: `http://localhost:8000/admin/login`.

Para executar cada processo separadamente, use terminais diferentes:

```bash
php artisan serve
npm run dev
php artisan queue:work
```

O worker da fila deve permanecer ativo para que as notificações sejam enviadas. Em produção, configure-o como um serviço supervisionado e reinicie-o após implantações com `php artisan queue:restart`.

## Testes e qualidade

```bash
composer test
php artisan test
vendor/bin/pint --test
npm run build
```

## Atualização do projeto

Após receber mudanças com novas dependências ou migrações:

```bash
composer install
npm install
php artisan migrate
php artisan optimize:clear
npm run build
```

## Produção

Antes de publicar, use `APP_ENV=production`, `APP_DEBUG=false`, HTTPS e credenciais fortes. Configure o banco, o SMTP e o worker de filas no ambiente do servidor; execute as migrações com `php artisan migrate --force`; e mantenha `.env`, chaves e senhas fora do repositório.
