# 02 — Templates

Modelos padronizados usados pelo serviço.

| Template | Uso | Status |
|----------|-----|--------|
| [template-hunt.md](template-hunt.md) | Estrutura obrigatória de todo hunt da biblioteca | v1.1 |
| Template de relatório ao cliente | Resultados, achados, lacunas e recomendações | A fazer |
| Registro de execução | O que foi feito em cada execução de hunt em um cliente (preenchido fora deste repositório) | A fazer |

## Template de hunt

### Diferença entre hunt e execução

- **Hunt** (este template): o procedimento genérico e reutilizável. Fica na biblioteca, sem nenhum dado de cliente.
- **Execução**: a aplicação de um hunt em um cliente, em uma data. Gera evidências e resultados, que ficam nos sistemas de atendimento do SOC, nunca no repositório.

### Dois níveis de status

Este repositório é uma **biblioteca de referência**: os hunts são publicados completos e revisados, mas a **validação em plataforma real é feita por quem os adota**, no próprio ambiente (ver [ADR 0006](../decisoes/0006-validacao-pelo-adotante.md)). Por isso existem dois níveis de status.

#### Status na biblioteca (campo Status)

É o status registrado no hunt, neste repositório.

| Status | Significado |
|--------|-------------|
| **Rascunho** | Em elaboração; incompleto |
| **Proposto** | Completo, com hipótese, lógica, consultas com sintaxe verificada e triagem. **Não validado** em plataforma real pelo projeto |
| **Validado pela comunidade** | Proposto + pelo menos uma validação em plataforma real contribuída e registrada na seção 9 |
| **Descontinuado** | Obsoleto ou substituído (indicar o substituto) |

Para passar a **Proposto**, o hunt precisa ter:

1. todas as seções preenchidas;
2. pseudoconsulta escrita e, quando houver regra Sigma, sintaxe verificada;
3. nenhum dado real de cliente;
4. revisão por outra pessoa, **recomendada** e registrada no campo **Revisor** quando houver.

#### Status operacional (no ambiente de quem adota)

É o status que cada time controla **internamente**, fora deste repositório, antes de usar um hunt em clientes.

| Status | Significado | Pode ser executado em cliente? |
|--------|-------------|--------------------------------|
| **Em validação** | Em teste no ambiente do time, com dados de teste e validação histórica | Não (exceto hunts extraordinários, ver [backlog e priorização](../01-processo/backlog-e-priorizacao.md#hunts-extraordinários)) |
| **Ativo** | Validado na plataforma do time e revisado | Sim |
| **Automatizado** | Convertido em regra de detecção; executado manualmente só quando necessário | Sim |

> **Importante:** um hunt **Proposto** é um ponto de partida. Antes de executá-lo em clientes, valide-o no seu ambiente, conforme o [processo de desenvolvimento](../01-processo/desenvolvimento-de-hunts.md).

### Versionamento do hunt

- `0.x`: Rascunho.
- `1.0`: primeira versão publicada como Proposto.
- Ajuste de consulta, falso positivo ou texto: incrementa a casa decimal (`1.1`).
- Mudança de hipótese ou de lógica: nova versão inteira (`2.0`).

Toda mudança é registrada na seção 16 do próprio hunt.

### Dicas de preenchimento por seção

- **1. Objetivo:** específico e mensurável. "Procurar exfiltração" é amplo demais; "identificar hosts com volume anômalo de consultas DNS a domínios externos nos últimos 30 dias" é testável.
- **3. Hipótese:** deve poder ser provada ou refutada com as fontes da seção 5. Um hunt que termina com "não encontramos nada" também é um resultado válido, desde que a hipótese tenha sido testada.
- **5. Fontes de dados:** é a seção mais importante para o MSSP. Ela define em quais clientes o hunt é viável e alimenta a matriz de cobertura do onboarding.
- **6. Baseline:** diz como reconhecer o normal em um cliente novo. Sem isso, a triagem vira opinião.
- **7. Estratégia:** escreva a lógica sem depender de sintaxe. Assim, o hunt pode ser portado para novas plataformas.
- **8. Consultas:** nunca invente nomes de campos ou funções. Marque como **Não validada** toda consulta que não foi testada em plataforma real. A pseudoconsulta é obrigatória; a regra Sigma, quando couber, vai em arquivo `.yml` separado (ADR 0005).
- **10. Triagem:** escreva para o analista mais júnior do time conseguir classificar o resultado sozinho.
- **13. Outros achados:** configurações incorretas e pontos cegos são entregas de valor para o cliente, mesmo sem ataque.

Baseado em: OTHF, módulo 6 — Operational e módulo 7 — Measurement.
