# Temura

Site institucional da Temura, em React 19 e Next.js 16, com animações em GSAP ScrollTrigger. O site é exportado como arquivos estáticos (`output: 'export'`) e publicado no **Netlify**. Não há servidor Node.js em produção; o Node.js é usado só no desenvolvimento e no build.

## Como o projeto é publicado

- **Hospedagem:** Netlify, a partir da branch `main` do repositório `caiovfreitas/Temura`.
- **Build:** `pnpm run build:site` (definido em `netlify.toml`), que gera a pasta `out/`.
- **Fluxo de trabalho:** as alterações são feitas em branches e revisadas antes de entrar na `main`. Como qualquer push na `main` publica o site, a `main` deve conter só versões estáveis.
- **Repositório espelho:** `LucasFreitasBR/Temura` guarda uma cópia (backup). Ele é atualizado por Pull Request a partir do fork, quando há uma versão estável.

## Desenvolvimento

Requer Node.js 22 e pnpm (a versão está em `packageManager`, no `package.json`).

```bash
pnpm install
pnpm dev          # servidor local em http://127.0.0.1:3000
pnpm run build:site   # mesmo build do Netlify; o resultado fica em out/
pnpm typecheck
```

O `pnpm dev` mostra o visual, mas **não** executa a função do formulário (veja abaixo).

## Estrutura

| Caminho | O que é |
| --- | --- |
| `app/` | Páginas, layout e estilos (`page.tsx`, `layout.tsx`, `globals.css`) |
| `public/` | Fontes e imagens |
| `netlify/functions/leads.ts` | Função que recebe o formulário e envia o e-mail |
| `server/validation.ts` | Regras de validação dos campos do formulário (compartilhadas com a função) |
| `netlify.toml` | Comando de build, pasta publicada e pasta de funções |

## Formulário de contato

O formulário envia um `POST` para `/api/leads`, atendido pela função do Netlify em `netlify/functions/leads.ts`. A função valida os dados e envia um e-mail para a Temura pelo [Resend](https://resend.com).

**Variáveis de ambiente** (Netlify → Site configuration → Environment variables):

| Variável | Obrigatória | Descrição |
| --- | --- | --- |
| `RESEND_API_KEY` | Sim | Chave criada em resend.com/api-keys |
| `CONTACT_TO_EMAIL` | Sim | E-mail que recebe os contatos (vários, separados por vírgula) |
| `CONTACT_FROM_EMAIL` | Não | Remetente. Sem ela, usa `onboarding@resend.dev` até o domínio ser verificado no Resend |

Sem as duas primeiras variáveis, o formulário responde que está temporariamente indisponível.

**Proteções na função:** só aceita `POST` com JSON; confere a origem da requisição; limita o corpo a 16 KB; valida os campos no servidor (com zod); usa campo honeypot e tempo mínimo de preenchimento; e envia uma chave de idempotência ao Resend para não duplicar e-mails em reenvios.

**O que a função não faz:** não guarda os contatos em banco de dados e não limita envios por IP. Os contatos existem apenas na caixa de entrada que recebe os e-mails.

## Pendências

- [ ] **Revisar o aviso de privacidade** (modal no final de `app/page.tsx`). O texto ainda diz que os dados ficam em banco de dados, que contatos com mais de 180 dias são removidos e que há um identificador técnico de rede com validade de 24 horas. Isso descrevia a versão antiga com banco de dados e **não corresponde ao funcionamento atual** (envio por e-mail). Deve ser corrigido antes de abrir o site para clientes.
- [ ] Trocar a arte `public/temura-chrome.webp` por imagens reais e adicionar trabalhos apenas quando fornecidos.
- [ ] Cadastrar contato oficial, domínio e informações do responsável no aviso de privacidade.
- [ ] Verificar o domínio no Resend e definir `CONTACT_FROM_EMAIL`.
- [ ] Definir a política de retenção dos e-mails recebidos e, se necessário, integrar um CRM.
- [ ] Quando o site for aprovado para o público: configurar o domínio e trocar `robots: { index: false, follow: false }` em `app/layout.tsx` (hoje o site está com `noindex`, escondido dos buscadores).

## Código legado (Cloudflare Workers + D1)

Antes do Netlify, a API do formulário rodava em Cloudflare Workers com banco D1. Esse código continua no repositório, mas **não é usado pelo deploy atual**:

- `server/worker.ts`, `server/db.ts`
- `db/`, `drizzle/` e `drizzle.config.ts`
- `wrangler.jsonc`
- `scripts/` (`build-worker.mjs`, `preview.mjs`, `dev.mjs`, `test-contact.mjs`)
- scripts `build`, `preview`, `preview:node`, `test` e `db:generate` do `package.json`

Atenção: o script `build` (diferente de `build:site`) faz parte desse caminho antigo e depende de um arquivo (`.openai/hosting.json`) que não está no repositório, então não funciona num clone limpo. Use sempre `build:site`.

## Acessibilidade e conteúdo

O site respeita a preferência de redução de movimento. Não contém clientes, depoimentos, resultados ou métricas fictícios. O gráfico é uma composição visual do serviço, sem dados de desempenho.
