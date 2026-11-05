# Cena do Crime — formulário e vídeo

## O que faz
- Página única (`index.html`): responde perguntas, salva no Supabase e libera o vídeo somente após a confirmação do envio.
- `admin.html`: login administrativo, perguntas configuráveis, URL do vídeo, e-mail destinatário e histórico das respostas.
- Sem Formspree. O envio de e-mails é feito pela Edge Function com a API Resend, caso configurada.

## Instalação no Supabase
Projeto já criado: `rjpjxvuyiipgiplknzll` (São Paulo).

1. No painel do projeto, abra **SQL Editor** e execute o conteúdo de `supabase/migrations/001_setup.sql` (uma única vez).
2. Abra **Edge Functions** e crie uma função chamada `api`, copiando `supabase/functions/api/index.ts`. Configure a função como **pública / Verify JWT desativado**: ela valida o token do administrador internamente para rotas privadas. Não publique outras tabelas com permissões abertas.
3. Em **Authentication → Users**, crie um usuário administrativo com seu e-mail e uma senha forte (ou cadastre-o usando os métodos permitidos no projeto). Copie o UUID desse usuário.
4. No SQL Editor, execute: `insert into public.admin_users(user_id) values ('UUID_DO_USUARIO');` Substitua pelo UUID real. Não inclua usuários desconhecidos.
5. Em **Edge Functions → Secrets**, configure `RESEND_API_KEY` e `EMAIL_FROM` (por exemplo, `Perícia <avisos@seudominio.com>`). Verifique o domínio remetente no Resend. O e-mail de destino é alterado em `admin.html`. Sem esses secrets, as respostas são salvas, mas os e-mails não são enviados.
6. Hospede os arquivos `index.html`, `admin.html`, `style.css`, `app.js`, `admin.js` e `config.js` em hospedagem HTTPS estática (Netlify, Vercel, GitHub Pages etc.). Para testes locais, sirva por HTTP (`python -m http.server 8000`) em vez de abrir arquivos por `file://`.
7. Abra `admin.html`, entre com o usuário autorizado e configure o e-mail e o vídeo.

## Segurança e observações
- O painel só funciona para usuários cadastrados em `admin_users`. Não há senha administrativa embutida no HTML.
- As tabelas estão protegidas por RLS, sem acesso direto de visitantes; a Edge Function faz a validação e salva os dados.
- A chave pública em `config.js` é adequada para frontend; **nunca** coloque `service_role`, chave secreta ou token Resend em arquivos públicos.
- A URL do vídeo só é retornada após salvar o relatório. Isso não impede que um link do YouTube público seja compartilhado por outra pessoa.
- O envio de e-mail pode falhar sem impedir a liberação do vídeo, pois o relatório já foi salvo. O status fica no painel.
- O endpoint público aceita envios sem login; para uso público de grande escala, adicione CAPTCHA, limite de requisições e política de privacidade antes de divulgar.
- Ao salvar a lista de perguntas, os identificadores são recriados; o histórico mantém as respostas, mas pode mostrar identificadores de perguntas antigas. Para produção, recomenda-se versionamento de perguntas.

## Identificação da equipe (nova versão)
O formulário exige Nome / Equipe. O servidor grava o valor em `answers.__team_name` e o painel mostra esse campo na aba Respostas e na exportação CSV. A configuração dos e-mails foi removida da interface administrativa; os destinatários já cadastrados no banco permanecem inalterados. Para validar o campo também no servidor, publique a versão atualizada da Edge Function em `supabase/functions/api/index.ts`.
