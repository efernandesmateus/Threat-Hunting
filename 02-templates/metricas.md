# Métricas do serviço

Catálogo das métricas usadas para acompanhar a operação e demonstrar o valor do serviço. Documentar é o que prova o valor do hunting, inclusive quando nada malicioso é encontrado.

Todas as métricas são calculadas a partir dos **registros de execução** ([template](template-registro-execucao.md)) e dos **relatórios periódicos** ([template](template-relatorio-cliente.md)).

## Público

Cada métrica tem um público definido:

| Público | Significado |
|---------|-------------|
| **Cliente** | Aparece no relatório periódico do cliente |
| **Interno** | Usada apenas na gestão do SOC; não é apresentada ao cliente, porque expõe dados de capacidade, custo ou desempenho individual |
| **Ambos** | Aparece no relatório e na gestão interna |

## 1. Operação

| Métrica | Definição | Fórmula | Frequência | Público |
|---------|-----------|---------|------------|---------|
| Hunts executados | Execuções concluídas no ciclo | Contagem de registros de execução | Ciclo | Ambos |
| Taxa de execução do plano | Quanto do planejado foi feito | Hunts executados ÷ hunts planejados | Ciclo | Ambos |
| Hunts não executados | Hunts planejados que não puderam ser executados, por motivo | Contagem por motivo (fonte, retenção, acesso, consulta quebrada) | Ciclo | Ambos |
| Horas dedicadas | Tempo total de hunting | Soma da duração dos registros | Ciclo | Interno |
| Desvio de esforço | Diferença entre esforço real e estimado | (Duração real − esforço estimado) ÷ esforço estimado, por hunt | Ciclo | Interno |
| Utilização da capacidade | Quanto da capacidade do time foi usada | Horas dedicadas ÷ capacidade ([modelo](../00-servico/objetivos-e-capacidade.md#3-modelo-de-dimensionamento)) | Ciclo | Interno |
| Cumprimento do prazo de notificação | Achados Crítica/Alta notificados no prazo de referência | Notificações no prazo ÷ total de notificações | Ciclo | Ambos |
| Cumprimento do prazo do relatório | Relatórios entregues no prazo | Relatórios no prazo ÷ total de relatórios | Ciclo | Interno |

## 2. Valor

| Métrica | Definição | Fórmula | Frequência | Público |
|---------|-----------|---------|------------|---------|
| Resultado das hipóteses | Distribuição dos resultados | Contagem de confirmadas, refutadas e inconclusivas | Ciclo | Ambos |
| Taxa de inconclusivos | Execuções que não chegaram a uma conclusão | Inconclusivas ÷ executadas | Ciclo | Ambos |
| Achados maliciosos | Atividade maliciosa confirmada | Contagem por severidade (Crítica, Alta) | Ciclo | Ambos |
| Incidentes originados | Achados que viraram incidentes | Contagem de escalonamentos aceitos pela resposta a incidentes | Ciclo | Ambos |
| Outros achados | Problemas que não são ataques, mas expõem o ambiente | Contagem por tipo e severidade | Ciclo | Ambos |
| Recomendações atendidas | Recomendações implementadas pelo cliente | Atendidas ÷ emitidas (acumulado) | Ciclo | Ambos |
| Detecções derivadas | Hunts convertidos em regras de detecção | Contagem acumulada de regras registradas na seção 14 dos hunts | Ciclo | Ambos |

## 3. Redução da superfície de ataque

Métricas inspiradas no módulo 7 do OTHF. Contam o que os hunts revelaram e foi corrigido.

| Métrica | Definição | Fórmula | Frequência | Público |
|---------|-----------|---------|------------|---------|
| Lacunas de visibilidade | Fontes ausentes, logs incompletos, retenção insuficiente | Abertas e corrigidas no ciclo; saldo acumulado | Ciclo | Ambos |
| Hunts viáveis | Hunts da biblioteca que o ambiente comporta | Contagem a partir da matriz de cobertura do onboarding | Ciclo | Ambos |
| Configurações incorretas | Encontradas e corrigidas | Contagem | Ciclo | Ambos |
| Protocolos inseguros | Uso de protocolos em texto claro ou obsoletos | Contagem | Ciclo | Ambos |
| Credenciais expostas | Credenciais em scripts, arquivos ou tráfego | Contagem | Ciclo | Ambos |
| Ativos desconhecidos | Ativos fora do inventário do cliente | Contagem | Ciclo | Ambos |
| Ferramentas de adversário ou malware | Encontrados no ambiente | Contagem | Ciclo | Ambos |

## 4. Biblioteca

| Métrica | Definição | Fórmula | Frequência | Público |
|---------|-----------|---------|------------|---------|
| Hunts por status | Situação da biblioteca | Contagem por status (Rascunho, Proposto, Validado pela comunidade, Descontinuado) | Trimestral | Interno |
| Hunts ativos no ambiente do time | Hunts validados para uso operacional | Contagem por status operacional | Trimestral | Interno |
| Táticas e técnicas caçadas | Técnicas ATT&CK com hunt | Lista de técnicas (não percentual) | Trimestral | Ambos |
| Hunts revisados no prazo | Hunts ativos revisados nos últimos 12 meses | Revisados ÷ ativos | Trimestral | Interno |

## 5. Regras de uso

- **Técnicas caçadas, nunca "% de cobertura".** Ter um hunt para uma técnica não significa estar protegido contra ela. Um percentual passa uma falsa sensação de proteção.
- **Volume não é valor.** Quantidade de consultas, eventos analisados ou horas não mostra valor ao cliente. Use-as apenas internamente, para gestão.
- **Zero achados não é zero valor.** Hipóteses refutadas com dados de qualidade, lacunas corrigidas e hunts viabilizados são resultados e devem aparecer no relatório.
- **Tendência acima de número isolado.** A partir do segundo ciclo, apresente a evolução (ver seção B.7 do [relatório](template-relatorio-cliente.md)).
- **Métricas internas ficam internas.** Desvio de esforço e utilização da capacidade servem para dimensionar o time, não para comparar analistas.
- **Ligação com os objetivos.** Os objetivos do serviço ([GOST](../00-servico/objetivos-e-capacidade.md#2-gost-de-referência-do-serviço)) devem ser acompanhados por métricas deste catálogo.

---

Baseado em: OTHF, módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
