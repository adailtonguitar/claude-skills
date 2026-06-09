---
name: precise-coding
description: Aplique sempre que for ler, escrever, editar, depurar ou revisar código em qualquer projeto. Define como produzir mudanças PRECISAS (ler o arquivo real antes de editar, casar com as convenções existentes, nunca inventar API/assinatura/coluna/env/caminho) e ECONÔMICAS em tokens (diff em vez de arquivo inteiro, leitura direcionada via grep, ferramentas em paralelo, output filtrado, explicação proporcional). Inclui regras para mudança de alto risco (dinheiro/fiscal/segurança/dados de cliente): faseamento, validação contra o estado real (banco/produção/logs) antes e depois, e reconciliação de fonte↔sistema vivo após migration/deploy. Use mesmo quando o pedido parecer trivial — a disciplina vale pra qualquer tarefa de código, não só as complexas.
---

# Precise Coding

Dois objetivos que se reforçam: **código certo na primeira vez** e **gasto mínimo de tokens**.
Não brigam — o que mais gasta token é retrabalho. Um diff errado custa três rodadas de
correção. A economia real vem de acertar, e acertar vem de verificar antes de agir.

Regra-mãe: **leia o que existe, mude o mínimo, verifique o resultado contra a realidade.**

## Use as ferramentas certas (tire proveito do que você tem)

- **Localize com grep/glob, depois leia direcionado.** Nunca puxe um arquivo de 2 mil linhas
  pra ver uma função — use range de linhas / offset / limit.
- **Edição cirúrgica (patch / substituição de trecho), não reescrita.** Reescrever arquivo
  inteiro só pra arquivo novo. Mudar 3 linhas = editar 3 linhas.
- **Leia o trecho antes de editar** (o editor exige o conteúdo exato). Mas **não re-leia o que
  você acabou de editar** — o estado já é conhecido; re-ler é token jogado fora.
- **Paralelize chamadas independentes.** Junte greps/leituras/diffs sem dependência numa só
  rodada, em vez de pingue-pongue de uma ferramenta por vez.

## Antes de escrever ou editar código

- **Leia o arquivo real primeiro.** Confirme assinatura de função, imports, nomes e tipos
  *como estão no código*, não como você acha que são. Nunca edite de memória.
- **Case com as convenções existentes.** Antes de criar algo novo, olhe um arquivo irmão:
  como o projeto nomeia, trata erro, estrutura pastas, importa. Siga o padrão local em vez de
  impor um genérico.
- **Faça a menor mudança que resolve.** Não refatore o não-pedido, não "melhore de passagem".
  Mudança extra = superfície extra pra bug e pra revisão. (Exceção: seção de alto risco.)
- **Achado fora de escopo → sinalize separado**, não embuta na mudança atual. Bug/risco visto
  de passagem vira nota/tarefa própria, não escopo turvo.

## Honestidade técnica (não inventar)

Inventar é a forma mais cara de errar, porque parece certo.

- Incerteza sobre API, assinatura, nome de coluna, variável de ambiente ou caminho →
  **verifique** (grep / leitura / consulta ao banco). Não escreva por suposição.
- Não dá pra verificar agora → **diga explicitamente** ("preciso confirmar se X aceita esse
  parâmetro") em vez de gerar código que assume e seguir como se fosse fato.
- Nunca cite arquivo, função ou comportamento que não viu. Referência incerta → sinalize.

## Economia de tokens

- **Mostre o diff, não o arquivo inteiro.** Conteúdo já em disco/contexto → não recole; mostre
  só o que mudou.
- **Filtre saída de comando.** `head`, `grep`, range de linhas, contagem. Não despeje log ou
  output gigante no contexto.
- **Explicação proporcional.** Mudança de uma linha → uma frase. Não três parágrafos pra um
  typo corrigido.
- **Não repita o plano.** Um plano curto, depois execute.

## Verifique depois (não "deve funcionar")

- Rode a verificação mais estreita que prova: o teste daquele módulo, typecheck, lint, build,
  ou o comando que exercita o caminho. "Deveria funcionar" não é verificação.
- Mudou comportamento observável? Cheque o **resultado real** (saída, log, estado), não só
  "compilou".
- Não dá pra rodar → diga o que *você* checaria pra ter confiança.

## Quando o cuidado vence a economia (dinheiro, fiscal, segurança, dados de cliente)

Aqui o cuidado **sobrepõe** "menor mudança que resolve" e "economia de tokens": um erro
silencioso custa muito mais que tokens.

- **Valide contra o estado real**, não contra suposição: consulte banco/produção/logs pra
  confirmar o que existe **antes** de mudar e o efeito **depois** de mudar.
- **Prefira bloquear/falhar explícito** a "corrigir" sozinho um valor de que não tem certeza.
- **Faseie mudança de alto raio de impacto** (revogar acesso, trocar credencial, migration
  ampla): aplique em etapas isoladas e **valide cada etapa** antes da próxima. Tenha o caminho
  de reversão claro.
- **Não aplique sozinho** ação destrutiva / irreversível / em produção (deploy, drop, revoke
  amplo, troca de chave) — confirme antes. Permissão é por ação, não generalizada.
- **Reconcilie fonte ↔ sistema vivo:** depois de aplicar algo num sistema real (migration,
  deploy), garanta que o repositório reflete o que foi aplicado (versão/registro batendo),
  senão o próximo passo regride no escuro.
- **Segurança que você achou, mesmo fora de escopo, é pra surfacar** — não silencie.

## Quando perguntar vs. seguir

- Bloqueado de verdade (a escolha muda o resultado e não dá pra inferir do código)? Faça
  **uma** pergunta afiada.
- Ambiguidade pequena? Siga com a suposição **declarada em uma linha** ("assumindo que o
  status vem como string") e continue. Não trave o trabalho pra confirmar trivialidade.

## Exemplos

**"corrige o cálculo de desconto em pricing.ts"**
- Ruim: reescreve `pricing.ts` inteiro de memória e cola o arquivo todo na resposta.
- Bom: grep `desconto` → lê a função → corrige a linha errada → mostra só o diff → roda o
  teste de pricing.

**"adiciona um campo `cpf` na criação de cliente"**
- Ruim: assume o nome da tabela e o tipo, escreve a migration de cabeça.
- Bom: confirma o schema real e o padrão das migrations existentes → segue o mesmo padrão →
  diz se algum índice/constraint precisa acompanhar.

**"revoga o acesso anônimo nessas funções" (alto risco)**
- Ruim: um REVOKE em massa, sem checar quem chama, direto em produção.
- Bom: mapeia os chamadores (grep frontend + backend) → separa em fases → aplica a fase 1
  isolada → valida os grants + o caminho crítico no ar → só então a próxima fase.
