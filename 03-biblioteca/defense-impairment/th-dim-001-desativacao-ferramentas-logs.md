# TH-DIM-001 — Desativação ou alteração de ferramentas de segurança e de logs

| Campo | Valor |
|-------|-------|
| **ID** | TH-DIM-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Defense Impairment (TA0112) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 2–4 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, sinais de que defesas foram desativadas ou alteradas nos hosts do cliente: limpeza de logs, desativação de auditoria, parada ou remoção de agentes de segurança, desligamento do firewall do host, supressão do histórico de comandos e, principalmente, **hosts que deixaram de enviar telemetria** sem motivo conhecido.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** antes de executar as etapas mais ruidosas de um ataque (roubo de credenciais, movimentação lateral, criptografia de dados), adversários costumam **cegar as defesas**: param o agente de segurança, desligam o firewall, limpam os logs que registraram a intrusão ou desativam a auditoria para que as próximas ações não sejam registradas. No ATT&CK v19, esse comportamento ganhou uma tática própria, **Defense Impairment**, separada de Stealth: o adversário não está se escondendo das ferramentas, está **quebrando** as ferramentas.

Essa diferença muda o hunting. Muitas vezes, a desativação bem-sucedida de uma defesa apaga justamente o registro que a denunciaria. Por isso, o sinal mais confiável desta tática costuma ser a **ausência**: um host que enviava milhares de eventos por dia e de repente para, ou um agente que deixa de responder. Este hunt procura tanto os eventos de desativação quanto esse silêncio.

**Ameaças associadas:** o MITRE ATT&CK registra a desativação e a alteração de ferramentas de defesa por um grande número de grupos e famílias de malware, especialmente operadores de ransomware, que fazem isso de forma sistemática antes da criptografia. Ver referências.

## 3. Hipótese

- **Dado que** adversários desativam ou alteram defesas para que suas próximas ações não sejam detectadas nem registradas,
- **quando** eles param agentes de segurança, limpam logs, desativam a auditoria ou o firewall,
- **então** aparecerão eventos de limpeza de log e de alteração de auditoria, serviços de segurança parados ou removidos, comandos de desativação e hosts com queda abrupta ou interrupção da telemetria.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Defense Impairment | Disable or Modify Tools | T1685 |
| Defense Impairment | Disable or Modify Tools: Disable or Modify Windows Event Log | T1685.001 |
| Defense Impairment | Disable or Modify Tools: Disable or Modify Linux Audit System Log | T1685.004 |
| Defense Impairment | Disable or Modify Tools: Clear Windows Event Logs | T1685.005 |
| Defense Impairment | Disable or Modify Tools: Clear Linux or Mac System Logs | T1685.006 |
| Defense Impairment | Disable or Modify System Firewall: Windows Host Firewall | T1686.003 |
| Defense Impairment | Prevent Command History Logging | T1690 |

A técnica **T1685** foi criada no ATT&CK v19, a partir da antiga Impair Defenses (T1562) e de subtécnicas que foram revogadas e reemitidas. Mapeamentos antigos para T1562 devem ser remapeados para as técnicas acima.

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Logs do sistema operacional | Windows: log de segurança limpo (1102), outro log limpo (104), política de auditoria alterada (4719), alteração do firewall (4950), serviço com estado ou tipo de inicialização alterado (7036, 7040). Linux: eventos do sistema de auditoria e do gerenciador de serviços | 30 dias | Sim, **ou** a fonte abaixo |
| Telemetria do endpoint (EDR) | Criação de processos com linha de comando; parada de serviços; alterações de configuração de ferramentas de segurança; eventos de proteção contra adulteração | 30 dias | Sim, **ou** a fonte acima |
| Saúde dos agentes e volume de eventos por host | Último contato de cada agente de segurança e de coleta; quantidade de eventos recebidos por host ao longo do tempo | 30 dias | Complementar, **altamente recomendada** |
| Inventário de ativos | Lista de hosts que deveriam estar enviando dados, com status (ativo, desligado, descomissionado) | — | Complementar |

**Escopo de ativos:** estações de trabalho e servidores Windows e Linux.

**Requisito crítico:** a fonte de **saúde e volume** precisa vir de **fora do host** (da plataforma que recebe os dados ou do console de gestão dos agentes). Um registro gerado pelo próprio host pode ser exatamente o que o adversário desativou.

## 6. Comportamento normal (baseline)

Desativações legítimas acontecem: manutenção planejada, troca ou atualização de agentes, preparação de imagens, solução de problemas, desligamento de hosts e descomissionamento. Hosts também param de enviar dados por motivos inofensivos: estão desligados, fora da rede, em férias com o usuário ou com problema de conectividade.

**Como levantar o baseline em um cliente novo:**

1. Para cada host, calcule o **volume diário de eventos** em pelo menos 14 dias e registre a mediana e a variação típica.
2. Identifique os hosts com padrão naturalmente intermitente (notebooks, hosts de laboratório) e trate-os com limiares próprios.
3. Liste com o ponto focal os **serviços e processos dos agentes de segurança e de coleta** usados no ambiente. Essa lista é necessária para o hunt e fica fora deste repositório, porque identifica produtos.
4. Registre as janelas de manutenção e os processos legítimos de troca de agentes.

## 7. Estratégia de busca

**Técnicas de análise:** busca por padrões, análise temporal (silêncio de telemetria) e enriquecimento.

**Lógica:**

1. **Eventos de desativação:** coletar no período:
   - limpeza de logs (1102, 104; no Linux, truncamento ou remoção de arquivos de log);
   - alteração da política de auditoria com remoção de auditoria (4719);
   - desligamento ou alteração do firewall do host;
   - parada, desabilitação ou remoção de serviços da **lista de agentes de segurança e de coleta** do cliente;
   - comandos de desativação: `wevtutil cl`, `auditpol /clear` ou `/remove`, `netsh advfirewall ... state off`, `auditctl -e 0`, parada de `auditd`, `rsyslog` ou `systemd-journald`;
   - supressão do histórico de comandos do shell;
   - eventos de proteção contra adulteração gerados pelos próprios agentes.
2. **Silêncio de telemetria:** para cada host do inventário marcado como ativo:
   - comparar o volume de eventos de cada dia com o baseline do host;
   - destacar quedas abruptas (por exemplo, abaixo de 10% da mediana) e interrupções totais;
   - destacar hosts em que **um** agente parou de reportar enquanto os demais continuam (sinal forte de desativação seletiva).
3. Correlacionar: um evento de desativação seguido de silêncio no mesmo host é a combinação mais grave.
4. Para cada destaque, verificar a atividade **imediatamente antes** da desativação: quem executou, de onde, e o que mais aconteceu no host.

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Queda de volume de eventos | Abaixo de 10% da mediana diária do host |
| Interrupção de telemetria | Nenhum evento por ≥ 4 horas em host ativo que normalmente envia dados continuamente |
| Agente sem contato | Sem contato por ≥ 4 horas enquanto outros agentes do mesmo host continuam ativos |
| Hosts com eventos de desativação no mesmo dia | ≥ 3 hosts (possível ação em massa) |

**Prioridade:** limpeza do log de segurança, desativação de auditoria, parada de agente de segurança fora de janela de manutenção e silêncio seletivo de um agente já justificam a triagem mesmo isolados. Eventos de desativação em vários hosts no mesmo dia indicam ação em massa e são urgentes.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
PARTE A — EVENTOS DE DESATIVAÇÃO

FONTE:      Logs do sistema operacional e telemetria do endpoint, últimos 30 dias

INDICADORES:
            log_limpo        = Windows 1102 ou 104
                               OU wevtutil com "cl" / "clear-log"
                               OU (Linux) truncamento ou remoção de arquivos de log
            auditoria_off    = Windows 4719 com remoção de sucesso/falha
                               OU auditpol com /clear, /remove ou :disable
                               OU auditctl -e 0 / -D
            firewall_off     = alteração do firewall do host para desligado
                               OU netsh com "advfirewall" e "state off"
            agente_parado    = parada, desabilitação ou remoção de serviço da lista de
                               agentes de segurança e de coleta do cliente
            log_servico_off  = (Linux) parada, desabilitação ou máscara de auditd, rsyslog,
                               syslog-ng ou systemd-journald
            historico_off    = HISTFILE=/dev/null, unset HISTFILE, HISTSIZE=0, history -c
            adulteracao      = evento de proteção contra adulteração de agente

FILTRO:     Excluir janelas de manutenção e contas da lista de exceções do cliente

PARTE B — SILÊNCIO DE TELEMETRIA

FONTE:      Volume de eventos por host e último contato dos agentes,
            obtidos da plataforma que recebe os dados (não do próprio host)

PARA CADA:  host ativo no inventário, por dia
            volume_dia    = quantidade de eventos recebidos
            queda         = volume_dia < 10% da mediana do host (14 dias)
            interrupcao   = maior intervalo sem eventos >= 4 h (host contínuo)
            agente_mudo   = um agente sem contato >= 4 h enquanto outro agente do
                            mesmo host continua reportando

CORRELAÇÃO: evento da Parte A seguido de queda, interrupção ou agente_mudo no mesmo host
            hosts_no_dia = quantidade de hosts com eventos da Parte A no mesmo dia

DESTAQUE:   log_limpo (log de segurança) OU auditoria_off OU agente_parado OU adulteracao
            OU agente_mudo OU correlação Parte A + Parte B
            OU hosts_no_dia >= 3
            OU qualquer outro indicador fora de janela de manutenção
            (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      host, data_hora, indicador, conta, processo, linha_comando,
            volume antes e depois, tempo sem contato, hosts afetados no mesmo dia
            Ordenar por gravidade (correlação e ação em massa primeiro)
```

**Observações:**

- A Parte B depende de dados que a plataforma de coleta tenha sobre si mesma (volume recebido, último contato). Se essa informação não existir, registre como lacuna de visibilidade de alta prioridade.
- Diferencie host desligado de host silenciado: um host desligado para **todos** os sinais (inclusive de rede, se houver); um host com um agente silenciado continua ativo em outras fontes.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-dim-001-desativacao-ferramentas-logs.yml`](th-dim-001-desativacao-ferramentas-logs.yml)

O arquivo contém seis regras, com sintaxe verificada por um analisador Sigma de código aberto e expressão regular testada com exemplos positivos e negativos:

| Regra | Sistema | Nível |
|-------|---------|-------|
| Log de segurança limpo (1102) | Windows | high |
| Log do sistema ou de aplicação limpo (104) | Windows | high |
| Auditoria desativada (4719) | Windows | high |
| Limpeza de logs, desativação de auditoria ou do firewall por linha de comando | Windows | high |
| Auditoria ou serviço de log desativado | Linux | high |
| Supressão do histórico de comandos do shell | Linux | medium |

Limitações da versão Sigma em relação à pseudoconsulta:

- A **parada de agentes de segurança** depende da lista de serviços de cada ambiente, que identifica produtos e por isso não pode estar no repositório (ADR 0004). Essa parte fica na pseudoconsulta, com a lista mantida pelo time.
- O **silêncio de telemetria** é uma ausência de eventos e não pode ser expresso como regra Sigma; ele é o centro da Parte B da pseudoconsulta.
- A supressão do histórico só aparece na telemetria de processos quando o comando é passado a um novo shell; quando digitado em uma sessão interativa, ele não gera processo.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Telemetria esperada:** os eventos de limpeza de log e de alteração de auditoria (1102, 104, 4719), a criação dos processos de desativação com a linha de comando completa, os eventos de parada de serviço e, na Parte B, a queda de volume ou a perda de contato do agente na plataforma de coleta.

**Como gerar os dados (somente em laboratório isolado e com autorização formal):** use ferramentas de emulação de adversário com testes mapeados às técnicas T1685 e T1686, em hosts de laboratório dedicados. Para a Parte B, interrompa a coleta de um host de laboratório e confirme que a interrupção é detectada dentro da janela de referência. **Nunca** desative defesas em hosts de produção para testar o hunt.

**Alternativa sem laboratório:** usar datasets públicos de telemetria gravados durante simulações de desativação de defesas.

**Validação histórica:** executar o hunt em pelo menos 30 dias de dados reais, com autorização, para medir o ruído, ajustar os limiares de silêncio e registrar as janelas de manutenção.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | A desativação corresponde a manutenção planejada, troca de agente documentada ou descomissionamento; o silêncio corresponde a host desligado, fora da rede ou com problema de conectividade confirmado. |
| **Suspeito — investigar** | Desativação sem registro de manutenção, feita por conta administrativa conhecida; silêncio de host sem explicação imediata; agente sem contato com os demais ativos. |
| **Malicioso — escalar** | Limpeza de log ou desativação de auditoria por conta ou processo incomum; parada de agente de segurança seguida de atividade suspeita; desativação em vários hosts no mesmo dia; desativação precedida de acesso remoto incomum, execução ofuscada ou proxy de execução; adulteração detectada pelo próprio agente. |

**Perguntas de enriquecimento:**

- Quem desativou, de onde e com qual processo? A conta costuma fazer isso?
- Havia manutenção planejada? O ponto focal confirma?
- O que aconteceu no host **antes** da desativação? (ver [TH-INI-001](../initial-access/th-ini-001-acesso-remoto-origem-incomum.md), [TH-EXE-001](../execution/th-exe-001-interpretadores-comandos-ofuscados.md) e [TH-STL-001](../stealth/th-stl-001-utilitarios-legitimos-executando-codigo.md))
- O host silenciado continua visível em outras fontes (rede, autenticação, outro agente)?
- Outros hosts tiveram a mesma desativação no mesmo período?
- A desativação foi revertida? Por quem?

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Manutenção planejada e troca de agentes | Coincide com janela de manutenção ou mudança registrada | Registrar as janelas como exceção temporária |
| Preparação de imagens e implantação | Limpeza de logs em hosts recém-criados, por ferramenta de implantação | Exceção por processo e período de criação do host |
| Hosts desligados ou fora da rede | Silêncio em **todas** as fontes; padrão intermitente (notebooks) | Limiares próprios para hosts intermitentes |
| Problemas de coleta | Vários hosts silenciam ao mesmo tempo por falha no ponto de coleta | Verificar a saúde da infraestrutura de coleta antes de tratar como ataque |
| Solução de problemas por administradores | Parada de serviço seguida de reinício em minutos, por administrador conhecido | Confirmar com o ponto focal |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): eventos de desativação, linha do tempo da telemetria antes e depois, contas e processos envolvidos. Se os logs locais foram limpos, preserve o que já tinha sido coletado pela plataforma.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Desativação maliciosa de defesas é **Alta** no mínimo, e **Crítica** se ocorrer em vários hosts, em ativos críticos ou se estiver acompanhada de outra atividade suspeita, porque frequentemente antecede ransomware.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - isolar os hosts afetados;
   - restaurar e reativar as defesas desativadas, verificando a integridade da configuração;
   - bloquear a conta usada na desativação até a investigação;
   - verificar se a mesma ação ocorreu em outros hosts;
   - tratar o caso como possível preparação de ataque maior e acionar a resposta a incidentes.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Agentes de segurança sem proteção contra adulteração habilitada | Média | Habilitar a proteção contra adulteração |
| Hosts ativos que nunca enviaram telemetria | Média | Incluir os hosts na coleta |
| Ausência de monitoramento de saúde dos agentes e de volume por host | Média | Implementar alertas de silêncio de telemetria |
| Contas comuns com permissão para parar serviços de segurança | Média | Restringir permissões |
| Logs locais com tamanho máximo pequeno, que se sobrescrevem rapidamente | Baixa | Aumentar o tamanho e encaminhar os logs para fora do host |
| Auditoria de alteração de política (4719) desativada | Média | Habilitar a auditoria correspondente |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Sim |
| **Ruído permite alerta contínuo?** | Limpeza do log de segurança, desativação de auditoria e eventos de adulteração costumam ter ruído muito baixo e devem virar alertas. O **silêncio de telemetria** também deve virar monitoramento contínuo, com limiares por tipo de host: é a detecção mais importante desta tática e uma das mais esquecidas. |
| **Tipos de plataforma alvo** | SIEM, EDR |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [Defense Impairment (TA0112)](https://attack.mitre.org/tactics/TA0112/)
- MITRE ATT&CK — [T1685 Disable or Modify Tools](https://attack.mitre.org/techniques/T1685/)
- MITRE ATT&CK — [T1686 Disable or Modify System Firewall](https://attack.mitre.org/techniques/T1686/)
- MITRE ATT&CK — [T1690 Prevent Command History Logging](https://attack.mitre.org/techniques/T1690/)
- MITRE ATT&CK — [Notas de lançamento da v19](https://attack.mitre.org/resources/updates/updates-april-2026/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
