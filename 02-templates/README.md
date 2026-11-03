# 02 — Templates

Modelos padronizados usados pelo serviço.

| Template | Uso | Status |
|----------|-----|--------|
| [template-hunt.md](template-hunt.md) | Estrutura obrigatória de todo hunt da biblioteca | v1.0 |
| Template de relatório ao cliente | Resultados, achados, lacunas e recomendações | A fazer |
| Registro de execução | O que foi feito em cada execução de hunt em um cliente (preenchido fora deste repositório) | A fazer |

## Template de hunt

### Diferença entre hunt e execução

- **Hunt** (este template): o procedimento genérico e reutilizável. Fica na biblioteca, sem nenhum dado de cliente.
- **Execução**: a aplicação de um hunt em um cliente, em uma data. Gera evidências e resultados, que ficam nos sistemas de atendimento do SOC, nunca no repositório.

### Ciclo de vida (campo Status)

| Status | Significado | Pode ser executado em cliente? |
|--------|-------------|--------------------------------|
| **Rascunho** | Em elaboração | Não |
| **Em validação** | Completo, sendo testado com dados de teste e histórico | Não (exceto hunts extraordinários, ver [backlog e priorização](../01-processo/backlog-e-priorizacao.md#hunts-extraordinários)) |
| **Ativo** | Validado em pelo menos uma plataforma e revisado | Sim, nas plataformas com consulta validada |
| **Automatizado** | Convertido em regra de detecção; executado manualmente só quando necessário | Sim |
| **Descontinuado** | Obsoleto ou substituído (indicar o substituto) | Não |

Para passar a **Ativo**, o hunt precisa ter:

1. todas as seções preenchidas;
2. ao menos uma consulta com status **Validada**;
3. seção 9 (Validação) preenchida;
4. revisão de outro analista registrada no campo **Revisor**.

### Versionamento do hunt

- `0.x`: antes de ficar Ativo.
- `1.0`: primeira versão Ativa.
- Ajuste de consulta, falso positivo ou texto: incrementa a casa decimal (`1.1`).
- Mudança de hipótese ou de lógica: nova versão inteira (`2.0`).

Toda mudança é registrada na seção 16 do próprio hunt.

### Dicas de preenchimento por seção

- **1. Objetivo:** específico e mensurável. "Procurar exfiltração" é amplo demais; "identificar hosts com volume anômalo de consultas DNS a domínios externos nos últimos 30 dias" é testável.
- **3. Hipótese:** deve poder ser provada ou refutada com as fontes da seção 5. Um hunt que termina com "não encontramos nada" também é um resultado válido, desde que a hipótese tenha sido testada.
- **5. Fontes de dados:** é a seção mais importante para o MSSP. Ela define em quais clientes o hunt é viável e alimenta a matriz de cobertura do onboarding.
- **6. Baseline:** diz como reconhecer o normal em um cliente novo. Sem isso, a triagem vira opinião.
- **7. Estratégia:** escreva a lógica sem depender de sintaxe. Assim, o hunt pode ser portado para novas plataformas.
- **8. Consultas:** nunca invente nomes de campos ou funções. Na dúvida, `TODO: validar`.
- **10. Triagem:** escreva para o analista mais júnior do time conseguir classificar o resultado sozinho.
- **13. Outros achados:** configurações incorretas e pontos cegos são entregas de valor para o cliente, mesmo sem ataque.

Baseado em: OTHF, módulo 6 — Operational e módulo 7 — Measurement.
