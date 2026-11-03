# Definição e missão do serviço

## 1. Definição de threat hunting

> **Threat hunting** é a busca proativa, orientada por hipóteses e repetível por sinais de atividade maliciosa que não foram identificados pelos controles de segurança e pelas regras de detecção existentes no ambiente do cliente.

Cada parte da definição tem um propósito:

| Termo | Significado prático |
|-------|---------------------|
| **Proativa** | O hunt não começa com um alerta. Começa com uma pergunta feita pelo analista. |
| **Orientada por hipóteses** | Todo hunt parte de uma hipótese testável, que pode ser confirmada ou refutada com dados. |
| **Repetível** | O hunt segue um processo documentado e pode ser executado de novo, por outro analista ou em outro cliente, com o mesmo método. |
| **Não identificados** | O foco é o que escapou das detecções. Investigar alertas existentes é trabalho de monitoramento. |

### O que threat hunting **não** é

| Atividade | Por que é diferente |
|-----------|---------------------|
| **Monitoramento e triagem de alertas** | Reativo: parte de um alerta gerado por uma regra. O hunting procura o que nenhuma regra alertou. |
| **Resposta a incidentes** | Começa quando um incidente já foi confirmado. Um hunt pode **originar** um incidente, que então segue para a resposta. |
| **Pentest ou red team** | Simulam o atacante para testar defesas. O hunting procura o atacante real nos dados. |
| **Varredura de IOCs isolada** | Buscar indicadores conhecidos é útil, mas é só uma técnica dentro do hunting, não o serviço inteiro. |
| **Análise de vulnerabilidades** | Identifica fraquezas, não atividade maliciosa. Pode ser uma **fonte** de hipóteses. |

## 2. Missão

> Reduzir o tempo em que adversários permanecem sem ser detectados no ambiente dos clientes, por meio de hunts estruturados, orientados por inteligência e documentados, que transformam cada descoberta em melhoria permanente das defesas.

A missão deve ser revisada pelo menos uma vez por ano, ou quando o escopo do serviço mudar.

## 3. Princípios do serviço

1. **Toda execução parte de uma hipótese.** Sem hipótese não há hunt, apenas exploração de dados sem critério.
2. **Tudo é documentado.** Um hunt não documentado não pode ser repetido, auditado nem demonstrado ao cliente.
3. **Dados antes de consultas.** Antes de executar um hunt, verifica-se se o cliente tem as fontes de dados necessárias e se elas são confiáveis.
4. **"Nada encontrado" também é resultado.** Uma hipótese refutada com dados de qualidade reduz a incerteza e é reportada como tal.
5. **Todo hunt gera melhoria.** Achados viram incidentes tratados; padrões viram detecções; lacunas de visibilidade viram recomendações.
6. **Nenhuma garantia absoluta.** Hunting reduz o risco, mas não prova a ausência de comprometimento (ver [escopo](escopo.md#5-limitações)).
7. **Agnóstico de ferramenta.** O método vale para qualquer plataforma; as consultas são apenas a implementação.

## 4. Proposta de valor para o cliente

Por que contratar threat hunting além do monitoramento já existente:

| Benefício | Como o serviço entrega |
|-----------|------------------------|
| **Menor tempo de permanência do adversário** | Procura ativamente atividade que as regras não detectaram, em vez de esperar um alerta. |
| **Detecções melhores** | Hunts que encontram padrões relevantes são convertidos em regras de detecção contínua. |
| **Visibilidade sobre as próprias lacunas** | Cada hunt revela fontes de dados ausentes, logs incompletos e pontos cegos. |
| **Validação dos controles** | Confirma se os controles de segurança funcionam contra técnicas reais de adversários. |
| **Redução da superfície de ataque** | Hunts revelam configurações incorretas, protocolos inseguros, credenciais expostas e ativos desconhecidos. |
| **Evidência de diligência** | Relatórios periódicos documentam a postura proativa, o que apoia auditorias e requisitos regulatórios. |
| **Foco nas ameaças do setor** | Os hunts são priorizados pelas técnicas dos adversários que atacam o segmento do cliente. |

---

Baseado em: OTHF, módulo 2 — Foundational (definição, missão e apoio executivo). Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
