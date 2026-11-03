# TH-IMP-001 — Preparação para ransomware: exclusão de backups e cópias de recuperação

| Campo | Valor |
|-------|-------|
| **ID** | TH-IMP-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Impact (TA0040) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 1–2 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

> **Prioridade de automação.** Os indicadores deste hunt devem virar **alertas de tempo real** o quanto antes (ver seção 14). Entre a preparação e a criptografia costumam passar poucas horas, e um hunt periódico pode encontrá-la tarde demais. O hunt continua útil para verificar a cobertura dos alertas e encontrar tentativas que eles não pegaram.

## 1. Objetivo

Identificar, nos últimos 30 dias, a preparação típica de um ataque de ransomware nos hosts Windows do cliente: exclusão de cópias de sombra e de backups do sistema, desativação da recuperação na inicialização e parada em massa de serviços (bancos de dados, backup e segurança), especialmente quando ocorrem em vários hosts quase ao mesmo tempo.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** o objetivo de um ataque de ransomware é que a vítima **não consiga se recuperar sozinha**. Por isso, pouco antes de criptografar os dados, os operadores destroem as formas de recuperação disponíveis no próprio host: apagam as cópias de sombra (versões anteriores dos arquivos mantidas pelo sistema), excluem o catálogo e os backups do sistema e desativam o ambiente de recuperação da inicialização. Também param serviços que manteriam arquivos abertos e travariam a criptografia, como bancos de dados e soluções de backup, e serviços de segurança que poderiam interromper o processo.

Essas ações são feitas com **utilitários nativos** e deixam rastros claros na criação de processos. Elas raramente fazem parte da administração normal, e quase nunca em vários hosts ao mesmo tempo. Isso torna este o hunt com **menor ruído** da biblioteca e, ao mesmo tempo, o mais **urgente**: encontrar a preparação pode ser a última oportunidade de evitar a criptografia.

**Escopo:** este hunt cobre a **preparação** em hosts Windows. A criptografia em andamento (renomeação em massa de arquivos, notas de resgate) é assunto de detecção em tempo real, não de hunting periódico. A preparação em Linux e em hipervisores está no backlog como hunt futuro.

**Ameaças associadas:** o MITRE ATT&CK registra a inibição da recuperação do sistema e a parada de serviços por praticamente todas as famílias de ransomware conhecidas. Ver referências.

## 3. Hipótese

- **Dado que** operadores de ransomware destroem as formas de recuperação e param serviços antes de criptografar os dados,
- **quando** eles preparam o ambiente para a criptografia,
- **então** aparecerão exclusões de cópias de sombra e de backups, desativação da recuperação na inicialização e paradas de serviços em massa, frequentemente em vários hosts em pouco tempo e a partir de contas ou processos incomuns.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Impact | Inhibit System Recovery | T1490 |
| Impact | Service Stop | T1489 |
| Impact | Data Encrypted for Impact (contexto: etapa seguinte) | T1486 |

A técnica **T1486** não é procurada por este hunt; ela aparece no mapeamento porque é a etapa que a preparação antecede.

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Telemetria de processos do endpoint (EDR) | Criação de processos com linha de comando, conta, processo-pai, host | 30 dias | Sim, **ou** a fonte abaixo |
| Logs do sistema operacional | Criação de processos com linha de comando (4688); serviços com estado ou tipo de inicialização alterados (7036, 7040) | 30 dias | Sim, **ou** a fonte acima |
| Logs das soluções de backup | Exclusão de backups, alterações de retenção, falhas de tarefas, acessos ao console de backup | 30 dias | Complementar |

**Escopo de ativos:** estações de trabalho e servidores Windows, com prioridade para servidores de arquivos, de bancos de dados e de backup.

## 6. Comportamento normal (baseline)

Algumas dessas ações têm uso legítimo pontual: administradores podem excluir cópias de sombra para liberar espaço, limpar backups antigos ou parar serviços para manutenção e atualização. Esse uso é **raro**, feito por contas administrativas conhecidas, em hosts específicos e geralmente em janelas de mudança.

**Como levantar o baseline em um cliente novo:**

1. Em todo o histórico disponível, procure as execuções de `vssadmin`, `wbadmin` e `bcdedit` com os parâmetros deste hunt. Em muitos ambientes, o resultado é **vazio**, e isso já é o baseline.
2. Identifique os scripts de manutenção que param vários serviços (por exemplo, para atualização de aplicações) e registre-os como exceções do cliente, fora deste repositório, por host, conta e horário.
3. Registre as janelas de mudança e as contas que fazem administração de armazenamento e de backup.

## 7. Estratégia de busca

**Técnicas de análise:** busca por padrões, agrupamento temporal e enriquecimento.

**Lógica:**

1. Coletar as execuções que inibem a recuperação:
   - **cópias de sombra:** `vssadmin delete shadows`, `vssadmin resize shadowstorage` (reduzir o espaço apaga cópias), exclusão por WMI (`shadowcopy delete`) ou por PowerShell (`Win32_Shadowcopy` com exclusão);
   - **backups do sistema:** `wbadmin delete catalog`, `wbadmin delete systemstatebackup`, `wbadmin delete backup`;
   - **recuperação na inicialização:** `bcdedit` com `recoveryenabled no` ou `bootstatuspolicy ignoreallfailures`.
2. Coletar as paradas e desabilitações de serviços (`net stop`, `sc stop`, `sc config ... start= disabled`, `Stop-Service`, `Set-Service`) e agrupá-las por host em janelas curtas.
3. Destacar:
   - qualquer inibição de recuperação fora de exceção;
   - parada de muitos serviços no mesmo host em pouco tempo, especialmente de bancos de dados, backup e segurança;
   - as mesmas ações em **vários hosts** em pouco tempo (ação em massa, frequentemente distribuída por execução remota ou por política de domínio).
4. Verificar o que aconteceu **antes**: acesso remoto incomum, movimentação lateral, desativação de defesas e execução remota de comandos nos hosts afetados.
5. Verificar se há sinais de criptografia **já em andamento** nos mesmos hosts e acionar a resposta imediatamente se houver.

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Inibição de recuperação | Qualquer ocorrência fora de exceção |
| Parada de serviços em massa | ≥ 10 paradas ou desabilitações no mesmo host em 10 minutos |
| Ação em massa | Mesma ação em ≥ 3 hosts em 1 hora |

**Prioridade:** **todas** as ocorrências deste hunt justificam triagem imediata. Ação em massa e combinação com outros achados recentes justificam escalonamento direto.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Criação de processos com linha de comando (EDR ou logs do sistema operacional)
            Últimos 30 dias

FILTRO:     Excluir scripts de manutenção da lista de exceções do cliente
            (por host, conta e horário)

INDICADORES, por execução:
            sombra      = vssadmin com (delete E shadows) OU (resize E shadowstorage)
                          OU linha_comando com (shadowcopy E delete)
                          OU linha_comando com Win32_Shadowcopy E
                             (.Delete() OU Remove-WmiObject OU Remove-CimInstance)
            backup      = wbadmin com delete E (catalog OU systemstatebackup OU backup)
            recuperacao = bcdedit com (recoveryenabled E no)
                          OU (bootstatuspolicy E ignoreallfailures)
            parada      = net/net1 stop, sc stop, sc config start= disabled,
                          Stop-Service, Set-Service

AGREGAÇÃO:
            paradas_host  = contagem de parada por host em 10 minutos
            hosts_acao    = contagem distinta de host com (sombra OU backup OU
                            recuperacao) em 1 hora

DESTAQUE:   sombra OU backup OU recuperacao
            OU paradas_host >= 10
            OU hosts_acao >= 3
            (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      host, conta, data_hora, processo_pai, linha_comando, indicador,
            paradas_host, hosts_acao, atividade anterior no host
            Ordenar por hosts_acao e data_hora (mais recentes primeiro)
```

**Observações:**

- Em um hunt retroativo, verifique a **data** dos achados com atenção: uma preparação recente pode significar que a criptografia ainda não aconteceu, e cada hora conta.
- Execuções a partir de processos de execução remota (serviço criado remotamente, WMI, política de domínio) indicam distribuição em massa; procure imediatamente os demais hosts alcançados.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-imp-001-preparacao-ransomware.yml`](th-imp-001-preparacao-ransomware.yml)

O arquivo contém cinco regras, com sintaxe e condições verificadas por um analisador Sigma de código aberto:

| Regra | Nível |
|-------|-------|
| Exclusão ou redução de cópias de sombra (vssadmin, WMI, PowerShell) | high |
| Exclusão de catálogo ou de backups do sistema | high |
| Desativação da recuperação do sistema na inicialização | high |
| Parada ou desabilitação de serviço por linha de comando (base da correlação) | informational |
| **Correlação:** parada de 10 ou mais serviços no mesmo host em 10 minutos | high |

Limitações da versão Sigma em relação à pseudoconsulta:

- A **ação em massa** (a mesma ação em vários hosts) não está nas regras; pode ser implementada como correlação adicional na plataforma ou feita na execução do hunt.
- A classificação dos serviços parados (banco de dados, backup, segurança) depende dos nomes dos serviços de cada ambiente, que identificam produtos, e fica fora do repositório (ADR 0004).

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Telemetria esperada:** a criação dos processos `vssadmin`, `wbadmin`, `bcdedit`, `net`, `sc` ou do PowerShell com a linha de comando completa, a conta e o processo-pai; para as paradas de serviço, também os eventos de mudança de estado dos serviços.

**Como gerar os dados (somente em laboratório isolado e com autorização formal):** use ferramentas de emulação de adversário com testes mapeados às técnicas T1490 e T1489, em hosts de laboratório dedicados e descartáveis. **Estas ações são destrutivas**: apagam pontos de recuperação e podem impedir a recuperação do host. **Nunca** as execute em hosts de produção, nem em hosts de laboratório que tenham dados ou backups que precisem ser mantidos.

**Alternativa sem laboratório:** usar datasets públicos de telemetria gravados durante simulações de ransomware.

**Validação histórica:** executar o hunt em todo o histórico disponível, com autorização. Em muitos ambientes, o resultado esperado é **vazio** ou quase vazio; qualquer ocorrência deve ser explicada.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | Ação pontual por administrador conhecido, em janela de mudança registrada, confirmada pelo ponto focal; script de manutenção identificado. |
| **Suspeito — investigar** | Ação isolada por conta administrativa conhecida, sem mudança registrada e sem outros indicadores. Mesmo assim, confirmar **imediatamente** com o ponto focal. |
| **Malicioso — escalar** | Qualquer inibição de recuperação sem explicação confirmada; parada de serviços em massa fora de manutenção; ação em vários hosts; ação a partir de conta, processo ou host incomuns; ação precedida de outros achados (acesso remoto incomum, movimentação lateral, desativação de defesas). |

**Perguntas de enriquecimento:**

- Quem executou, de onde e com qual processo-pai? Foi execução remota?
- Quantos hosts tiveram a mesma ação? Quando começou?
- Há sinais de criptografia já em andamento nesses hosts?
- O que aconteceu antes? (ver [TH-DIM-001](../defense-impairment/th-dim-001-desativacao-ferramentas-logs.md), [TH-LAT-001](../lateral-movement/th-lat-001-movimentacao-lateral-contas-administrativas.md) e [TH-COL-001](../collection/th-col-001-compactacao-grandes-volumes.md))
- Os backups fora dos hosts (em outra infraestrutura) estão íntegros e acessíveis?

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Liberação de espaço de armazenamento | Administrador conhecido, host específico, janela de mudança | Confirmar com o ponto focal; exceção pontual |
| Limpeza de backups antigos | Conta de administração de backup, retenção documentada | Exceção por conta e host |
| Scripts de atualização de aplicações | Paradas de vários serviços da mesma aplicação, em horário de manutenção, por conta de serviço conhecida | Exceção por host, conta e horário |
| Preparação de imagens | Ações em hosts recém-criados, por ferramenta de implantação | Exceção pelo período de criação do host |

## 12. Ação ao encontrar atividade maliciosa

> **Este é o achado mais urgente da biblioteca.** Não espere o fim do hunt nem do ciclo.

1. Escalar **imediatamente** conforme o [processo de escalonamento](../../01-processo/escalonamento.md), com contato direto ao responsável do cliente. Preparação para ransomware confirmada é **Crítica**.
2. Recomendar ao cliente, conforme o modo de resposta contratado, **em paralelo à investigação**:
   - isolar os hosts afetados e, se a ação for em massa, considerar segmentar a rede inteira;
   - desabilitar as contas usadas, especialmente as privilegiadas;
   - proteger os backups fora dos hosts: desconectar ou bloquear o acesso a eles e verificar sua integridade;
   - acionar a resposta a incidentes e o plano de crise do cliente.
3. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório), sem atrasar a contenção.
4. Buscar a mesma atividade em **todos** os hosts do cliente e identificar o mecanismo de distribuição (execução remota, política de domínio, ferramenta de gestão comprometida).

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Backups acessíveis com as mesmas credenciais do domínio | Alta | Isolar a infraestrutura de backup e usar credenciais separadas |
| Ausência de cópias de backup imutáveis ou fora da rede | Alta | Implementar cópias imutáveis ou desconectadas |
| Cópias de sombra desabilitadas em servidores de arquivos | Baixa | Avaliar a habilitação como camada adicional de recuperação |
| Contas comuns com permissão para parar serviços críticos | Média | Restringir permissões |
| Ausência de teste periódico de restauração de backups | Média | Testar a restauração periodicamente |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | **Sim, com prioridade máxima** |
| **Ruído permite alerta contínuo?** | Sim. Inibição de recuperação e parada de serviços em massa têm ruído muito baixo depois das poucas exceções do cliente e devem virar **alertas de tempo real**, com escalonamento imediato e, quando o modo de resposta permitir, contenção automática do host. Este é o caso mais urgente de "hunt once" da biblioteca. |
| **Tipos de plataforma alvo** | SIEM, EDR |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1490 Inhibit System Recovery](https://attack.mitre.org/techniques/T1490/)
- MITRE ATT&CK — [T1489 Service Stop](https://attack.mitre.org/techniques/T1489/)
- MITRE ATT&CK — [T1486 Data Encrypted for Impact](https://attack.mitre.org/techniques/T1486/)
- MITRE ATT&CK — [Impact (TA0040)](https://attack.mitre.org/tactics/TA0040/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
