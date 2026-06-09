# claude-skills

Duas skills para o [Claude Code](https://code.claude.com), num **único pacote**:
**acertar de primeira** (`precise-coding`) e **distribuir o custo** entre o modelo
caro (Opus) e modelos mais baratos (`orchestrator-mode`).

> Honestas por design — dizem o que fazem **e o que NÃO fazem**. Nasceram e foram
> validadas em uso real (auditoria de segurança de um ERP fiscal em produção:
> mapeamento de funções, fix de vulnerabilidade cross-tenant, validação no banco ao vivo).

---

## Instalar (2 comandos — instala as duas juntas)

```
/plugin marketplace add adailtonguitar/claude-skills
/plugin install claude-skills@claude-skills
```
O `marketplace add` só registra; o `install` traz **as duas skills** de uma vez.
Reinicie a sessão do Claude Code (skills carregam no início).

**Cópia manual (sem plugin):**
```bash
git clone https://github.com/adailtonguitar/claude-skills
cp -r claude-skills/plugin/skills/precise-coding    ~/.claude/skills/
cp -r claude-skills/plugin/skills/orchestrator-mode ~/.claude/skills/
```

---

## O que cada uma faz (de verdade)

### `precise-coding`
Disciplina aplicada a **qualquer** tarefa de código (ler, escrever, editar, depurar, revisar):

- **Lê o arquivo real antes de editar** — nada de memória; confirma assinatura, imports, tipos como estão.
- **Casa as convenções existentes** (olha um arquivo irmão antes de criar algo novo).
- **Não inventa** API/assinatura/coluna/env/caminho — verifica (grep/leitura/consulta) ou sinaliza a incerteza.
- **Diff, não arquivo inteiro**; leitura direcionada; output filtrado; explicação proporcional.
- **Verifica depois** (teste/lint/build do caminho real) — não "deve funcionar".
- **Alto risco** (dinheiro/fiscal/segurança/dados de cliente): o cuidado vence a economia — valida contra o **estado real** (banco/produção/logs) antes e depois, **faseia** mudança de alto impacto, e **reconcilia fonte↔sistema vivo** após migration/deploy.

**O que NÃO é:** não é um corta-tokens mágico. É disciplina de **correção** — a economia
vem indiretamente, de **menos retrabalho** (um diff errado custa 3 rodadas de correção).

### `orchestrator-mode`
Quando você roda **Opus** (modelo caro) numa tarefa divisível:

- Opus é o **cérebro**: planeja, decide, revisa e faz o de alto risco.
- **Delega a execução** (localizar, ler, editar, build) a subagentes **Sonnet** (Haiku no trivial/bulk), com **prompt auto-contido** e **retorno compacto**.
- Delegação **conservadora**: só trabalho **pesado/divisível** (3+ arquivos, varredura ampla) ou quando você pede. Tarefa pequena → **inline**.
- **Nunca delega:** decisão, risco (fiscal/segurança/dados), ação destrutiva/produção, review final. E **re-verifica** no Opus o que o modelo barato fez em código sensível.

**Economiza quando:** você está em **Opus** *e* a tarefa é pesada/coletável — move tokens
caros (Opus) para Sonnet barato + um resumo compacto de volta. **Não ajuda** em tarefa
pequena (overhead) nem se seu modelo principal já for barato. Não é mágica — é
**distribuição de custo**.

---

## Notas honestas

- São **guias de comportamento** (sem código executável): não acessam segredos, não rodam nada sozinhas, só moldam como o Claude trabalha.
- `orchestrator-mode` só faz sentido com **Opus** como modelo principal.
- Os maiores economizadores de token continuam sendo **saída enxuta** e **`/compact` entre tarefas**. Estas skills **complementam**, não substituem.

## Licença
MIT — veja [LICENSE](./LICENSE).
