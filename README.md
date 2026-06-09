# claude-skills

Duas skills para o [Claude Code](https://code.claude.com) que trabalham juntas:
**acertar de primeira** (`precise-coding`) e **distribuir o custo** entre o modelo
caro (Opus) e modelos mais baratos (`orchestrator-mode`).

> Honestas por design — dizem o que fazem **e o que NÃO fazem**. Nasceram e foram
> validadas em uso real (auditoria de segurança de um ERP fiscal em produção:
> mapeamento de funções, fix de vulnerabilidade cross-tenant, validação no banco ao vivo).

---

## O que cada uma faz (de verdade)

### `precise-coding`
Disciplina aplicada a **qualquer** tarefa de código (ler, escrever, editar, depurar, revisar):

- **Lê o arquivo real antes de editar** — nada de memória; confirma assinatura, imports, tipos como estão.
- **Casa as convenções existentes** (olha um arquivo irmão antes de criar algo novo).
- **Não inventa** API/assinatura/coluna/env/caminho — verifica (grep/leitura/consulta) ou sinaliza a incerteza.
- **Diff, não arquivo inteiro**; leitura direcionada; output filtrado; explicação proporcional ao tamanho da mudança.
- **Verifica depois** (teste/lint/build do caminho real) — não "deve funcionar".
- **Alto risco** (dinheiro/fiscal/segurança/dados de cliente): o cuidado vence a economia — valida contra o **estado real** (banco/produção/logs) antes e depois, **faseia** mudança de alto impacto, e **reconcilia fonte↔sistema vivo** após migration/deploy.

**O que NÃO é:** não é um corta-tokens mágico. É disciplina de **correção** — a economia
vem indiretamente, de **menos retrabalho** (um diff errado custa 3 rodadas de correção).
Custo fixo praticamente nulo.

### `orchestrator-mode`
Quando você roda **Opus** (modelo caro) numa tarefa divisível:

- Opus é o **cérebro**: planeja, decide, revisa e faz o de alto risco.
- **Delega a execução** (localizar, ler, editar, build) a subagentes **Sonnet** (Haiku no trivial/bulk) via a ferramenta Agent, com **prompt auto-contido** e **retorno compacto**.
- Delegação **conservadora**: só trabalho **pesado/divisível** (3+ arquivos, varredura ampla) ou quando você pede. Tarefa pequena → **inline** (delegar custaria mais).
- **Nunca delega:** decisão, risco (fiscal/segurança/dados de cliente), ação destrutiva/produção, review final. E **re-verifica** no Opus o que o modelo barato fez em código sensível.

**Economiza quando:** você está em **Opus** *e* a tarefa é pesada/coletável — move tokens
caros (Opus) para Sonnet barato + um resumo compacto de volta. **Não ajuda** em tarefa
pequena (overhead do subagente) nem se seu modelo principal já for barato. Não é mágica —
é **distribuição de custo**.

---

## Instalar

**Via plugin (recomendado):**
```
/plugin marketplace add adailtonguitar/claude-skills
/plugin install precise-coding@claude-skills
/plugin install orchestrator-mode@claude-skills
```

**Cópia manual (sem plugin):**
```bash
git clone https://github.com/adailtonguitar/claude-skills
cp -r claude-skills/precise-coding/skills/precise-coding       ~/.claude/skills/
cp -r claude-skills/orchestrator-mode/skills/orchestrator-mode ~/.claude/skills/
```
Reinicie a sessão do Claude Code (skills carregam no início da sessão).

---

## Requisitos e notas honestas

- Claude Code com suporte a skills/plugins.
- `orchestrator-mode` só faz sentido com **Opus** como modelo principal; em Sonnet/Haiku principal não há para quem delegar mais barato.
- São **guias de comportamento** (sem código executável): não acessam segredos, não rodam nada sozinhas, não mudam seu projeto — só moldam como o Claude trabalha.
- Os maiores economizadores de token no Claude Code continuam sendo **saída enxuta** e **`/compact` entre tarefas**. Estas skills **complementam**, não substituem.

## Licença
MIT — veja [LICENSE](./LICENSE).
