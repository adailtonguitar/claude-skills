---
name: orchestrator-mode
description: Use quando estiver rodando como modelo caro (Opus) numa tarefa de código ou multi-step que dê pra dividir. O Opus vira ORQUESTRADOR — planeja, decide, revisa e faz o de alto risco — e DELEGA a execução a subagentes mais baratos via a ferramenta Agent (Sonnet por padrão; Haiku para trivial/bulk), pra distribuir e economizar custo de tokens. Define o que delegar (trabalho pesado, paralelo ou mecânico) vs. fazer inline (trivial), como escrever prompt de subagente auto-contido e exigir retorno compacto, e o que NUNCA delegar (decisão, fiscal/dinheiro/segurança/dados de cliente, ação destrutiva/produção). Delega de forma CONSERVADORA — só trabalho pesado/divisível (3+ arquivos ou varredura ampla) ou quando o usuário pede; tarefa pequena faz inline. O usuário autorizou a delegação nos projetos dele.
---

# Orchestrator Mode

Objetivo: **modelo caro = cérebro, modelos baratos = mãos.** O Opus decide e orquestra;
Sonnet/Haiku executam exatamente o que o Opus mandar. Assim o token caro fica na decisão,
não na digitação. (O usuário pediu isso como padrão — pode delegar sem perguntar a cada vez.)

> Pareia com `precise-coding`: o subagente também lê antes de editar, casa convenções, mostra
> diff e não inventa. O prompt que você dá a ele tem que carregar essa precisão.

## Divisão de papéis
| Camada | Modelo | Faz |
|---|---|---|
| Orquestra / decide / risco | **Opus** (você) | plano, arquitetura, fiscal/dinheiro/segurança/dados, review final, deploy/destrutivo |
| Execução normal | **Sonnet** (`model: "sonnet"`) | localizar código, ler trecho, editar 1–3 arquivos, rodar build/teste, refactor mecânico |
| Bulk / trivial | **Haiku** (`model: "haiku"`) | renomear em massa, varrer, extrair lista, formatação repetitiva |

Use a ferramenta **Agent** com `model` explícito. Onde encaixar, prefira os agentes
**cavecrew** (`investigator` = localizar, `builder` = edit 1–2 arquivos, `reviewer` = revisar
diff) — o retorno deles já vem comprimido.

## Quando delegar vs. fazer inline
**Delegue** (vale o overhead do spawn):
- Varredura/investigação ampla (vários arquivos, "onde está X", mapear diretório).
- Edição mecânica em múltiplos arquivos / refactor repetitivo.
- Trabalho **paralelizável** — dispare vários subagentes na mesma rodada.
- Leitura pesada cujo resultado cabe num resumo curto.

**Faça inline** (delegar custaria mais que resolver):
- Edição de 1–3 linhas, leitura única e curta, comando rápido.
- Qualquer coisa menor que o custo de subir um subagente frio.

Regra de ouro: **agrupe** trabalho num subagente só; não pulverize em muitos spawns minúsculos.

**Default conservador — na dúvida, NÃO delegue.** Só delegue quando bater um destes:
- o usuário disse "delega" / "use subagente"; OU
- a tarefa toca **3+ arquivos**, ou é varredura/investigação ampla cujo resultado vira resumo curto.

Tarefa de 1–2 arquivos, edição pontual ou leitura única → **faça inline**. Não delegue por viés:
o overhead de subir subagente frio + o resumo de volta só compensa em trabalho realmente pesado.
Errar pro conservador (uns tokens Opus a mais) custa menos que spawn surpresa + latência.

## Como mandar (o subagente nasce "frio", sem o contexto da conversa)
- Prompt **auto-contido**: caminhos exatos, a mudança precisa, critério de aceite, e o que
  rodar pra validar. Não conte com o que só você (Opus) sabe.
- Exija **retorno compacto**: "devolva só `arquivo:linha` + o diff aplicado + resultado do
  teste, sem colar arquivo inteiro." Tool-result enxuto = menos token de volta no seu contexto.
- Dê o **escopo fechado**: o que NÃO mexer. Subagente barato tende a extrapolar.

## NUNCA delegue a decisão (mantém no Opus)
- Julgamento de **dinheiro / fiscal / segurança / dados de cliente** — pode até delegar a
  *coleta* (ler/consultar), mas a **decisão e a validação** são suas.
- **Ação destrutiva / irreversível / em produção** (deploy, drop, revoke amplo, troca de
  chave): só o Opus, e só com confirmação do usuário.
- **Review final / aprovação** de qualquer mudança de risco.
- Arquitetura e trade-offs.

## Verifique o que o barato fez (não confie cego)
- Revise o retorno do subagente **antes** de agir em cima. Modelo barato erra em código sutil.
- Em código de risco, **re-cheque você mesmo** (não aceite "deve estar certo" do subagente).
- Errou? Corrigir 1 vez ainda é mais barato que ter feito tudo no Opus — mas erro repetido =
  traga a tarefa de volta pro Opus em vez de insistir no barato.

## Economia honesta
- Ganho real vem de mover **execução pesada** pro barato + **retorno compacto**. Não de
  delegar tudo. Tarefa trivial delegada = mais cara (overhead).
- O Opus ainda paga orquestração + o resumo de volta + contexto fixo do turno — isso não some.
- Se a tarefa toda é pequena, **ignore esta skill e faça inline**.
