# Escopo do serviço

Este documento define os limites do serviço. Ele serve de base para a seção de escopo dos contratos e deve ser apresentado ao cliente antes do onboarding.

## 1. O que está incluído

Em todos os níveis (detalhes por nível em [ofertas](ofertas.md)):

- Onboarding do cliente, com inventário das fontes de dados e definição dos hunts viáveis.
- Planejamento e priorização dos hunts.
- Execução dos hunts, conforme o [processo operacional](../01-processo/).
- Triagem e investigação inicial dos resultados, até a classificação como benigno, suspeito ou malicioso.
- Escalonamento de achados maliciosos, no modo de resposta contratado (ver [seção 4](#4-modos-de-resposta)).
- Registro de outros achados: configurações incorretas, protocolos inseguros, ativos desconhecidos, pontos cegos de visibilidade.
- Relatórios periódicos de resultados.

## 2. Premissas

O serviço só pode ser prestado se as premissas abaixo forem atendidas. Premissas não atendidas devem ser registradas e reduzem o conjunto de hunts viáveis.

| Premissa | Detalhe |
|----------|---------|
| **Acesso às plataformas** | Acesso de leitura às plataformas de segurança onde os dados estão (SIEM, EDR, XDR), com permissão para executar buscas. |
| **Fontes de dados** | O cliente mantém a coleta das fontes de dados acordadas no onboarding. Mudanças na coleta devem ser comunicadas ao SOC. |
| **Retenção** | Os dados ficam disponíveis por um período mínimo compatível com os hunts. Referência: **pelo menos 30 dias** pesquisáveis; recomendado 90 dias ou mais. |
| **Ponto focal técnico** | O cliente indica uma pessoa para responder dúvidas sobre o ambiente (ex.: "este comportamento é esperado?"). |
| **Responsável por decisões em incidentes** | O cliente indica quem decide sobre contenção e remediação, com contato disponível para escalonamentos críticos. |
| **Janela de execução** | Hunts que exijam ações ativas no ambiente (ex.: coleta em endpoints) são combinados previamente. Hunts sobre dados já coletados não exigem janela. |

## 3. Exclusões

Não fazem parte do serviço, salvo contratação à parte:

| Exclusão | Observação |
|----------|------------|
| **Resposta a incidentes completa** | O serviço escala o achado. Investigação aprofundada, erradicação e recuperação são do processo de resposta a incidentes. |
| **Forense digital** | Aquisição e análise forense de discos, memória ou dispositivos. |
| **Remediação** | Correção de vulnerabilidades, mudanças de configuração e limpeza de sistemas são responsabilidade do cliente. |
| **Pentest e red team** | Testes ofensivos não fazem parte do serviço. |
| **Monitoramento 24x7** | Hunting é periódico. Monitoramento contínuo de alertas é outro serviço. |
| **Compromise assessment** | Avaliação ampla para determinar se o ambiente foi comprometido. Tem escopo, método e entregáveis próprios. |
| **Gestão das plataformas** | Implantação, manutenção e licenciamento das ferramentas do cliente. |
| **Coleta de novas fontes de dados** | O serviço recomenda novas fontes; a implementação é do cliente. |

## 4. Modos de resposta

O que acontece quando um hunt encontra atividade maliciosa depende do modo contratado. O modo deve constar no contrato e no onboarding.

| Modo | O SOC faz | O cliente faz |
|------|-----------|---------------|
| **Somente notificação** | Confirma o achado, notifica o cliente, entrega as evidências e as recomendações de contenção. | Decide e executa a contenção e a remediação. |
| **Contenção pelo SOC** | Tudo do modo anterior, mais a execução de ações de contenção pré-autorizadas (ex.: isolar host, bloquear hash ou IP, desabilitar conta). Exige contrato de MDR ou resposta. | Autoriza previamente as ações, decide sobre ações fora da lista e executa a remediação. |

As ações de contenção pré-autorizadas devem estar listadas por escrito, com os ativos excluídos (ex.: servidores críticos que nunca podem ser isolados sem aprovação).

## 5. Limitações

Estas limitações devem ser comunicadas ao cliente de forma explícita:

1. **Um hunt sem achados não prova a ausência de comprometimento.** Ele indica que a hipótese testada não foi confirmada nos dados disponíveis, no período analisado.
2. **O resultado depende dos dados.** Fontes ausentes, incompletas ou com retenção curta limitam o que pode ser encontrado. Essas lacunas são reportadas.
3. **Os hunts cobrem técnicas específicas.** Nenhum conjunto de hunts cobre todas as técnicas possíveis de um adversário.
4. **O serviço é periódico.** Atividade maliciosa que comece e termine entre duas execuções pode não ser encontrada.

## 6. Prazos de comunicação de referência

Valores de **referência**, ajustáveis por contrato:

| Situação | Prazo de referência |
|----------|---------------------|
| Achado classificado como **malicioso** | Notificação ao cliente em até **2 horas** após a confirmação. |
| Achado **suspeito** que depende de informação do cliente | Consulta ao ponto focal em até **1 dia útil**. |
| Outros achados (configuração, visibilidade) | Registrados no relatório periódico. |
| Relatório periódico | Até **5 dias úteis** após o fim do ciclo. |

O prazo conta a partir da **confirmação** do achado pelo analista, não do início do hunt.
