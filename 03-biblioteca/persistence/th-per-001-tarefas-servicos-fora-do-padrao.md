# TH-PER-001 — Tarefas agendadas e serviços fora do padrão

| Campo | Valor |
|-------|-------|
| **ID** | TH-PER-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Persistence (TA0003) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 3–4 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, tarefas agendadas e serviços criados ou alterados de forma incomum no ambiente: raros entre os hosts, com executáveis em diretórios graváveis por usuários, com interpretadores de script como ação, criados remotamente ou por contas e processos que normalmente não fazem isso. Cobre Windows (tarefas agendadas e serviços) e Linux (cron, timers e serviços do systemd).

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** depois de entrar e executar código, o adversário precisa **permanecer**: sobreviver a reinicializações, a trocas de senha e ao fim da sessão. Tarefas agendadas e serviços são dois dos mecanismos mais usados para isso, porque existem em todo sistema, são executados automaticamente e, no caso dos serviços, costumam rodar com privilégios elevados. Por isso o ATT&CK também os associa às táticas de Execution e Privilege Escalation.

Ambientes legítimos têm centenas de tarefas e serviços, mas eles tendem a ser **os mesmos** em muitos hosts, instalados pelos mesmos softwares, a partir dos mesmos caminhos. Uma persistência maliciosa costuma ser **rara**: aparece em um ou poucos hosts, com nomes que imitam os legítimos, mas com caminhos, ações ou criadores incomuns. É essa raridade que o hunt procura, por meio de stacking.

**Ameaças associadas:** o MITRE ATT&CK registra o uso de tarefas agendadas e de criação de serviços por um grande número de grupos e famílias de malware, inclusive como mecanismo de movimentação lateral (criação remota em outros hosts). Ver referências.

## 3. Hipótese

- **Dado que** adversários criam tarefas agendadas e serviços para manter acesso aos hosts comprometidos,
- **quando** eles criam ou alteram esses mecanismos,
- **então** aparecerão tarefas e serviços raros no ambiente, com executáveis em diretórios graváveis por usuários, com interpretadores de script como ação, criados remotamente ou por contas e processos incomuns.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Persistence | Scheduled Task/Job: Scheduled Task | T1053.005 |
| Persistence | Scheduled Task/Job: Cron | T1053.003 |
| Persistence | Scheduled Task/Job: Systemd Timers | T1053.006 |
| Persistence | Create or Modify System Process: Windows Service | T1543.003 |
| Persistence | Create or Modify System Process: Systemd Service | T1543.002 |
| Execution | System Services: Service Execution | T1569.002 |

As técnicas **T1053** e **T1543** também pertencem às táticas Execution e Privilege Escalation no ATT&CK. Este hunt as classifica por Persistence, que é a intenção principal investigada. A **T1569.002** entra quando o serviço é criado apenas para executar um comando, inclusive remotamente.

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Logs do sistema operacional | Windows: tarefa criada (4698), alterada (4702) e excluída (4699), no log de segurança; serviço instalado (7045, log do sistema; 4697, log de segurança). Linux: auditoria de alterações em arquivos de cron e de unidades do systemd | 30 dias | Sim, **ou** a fonte abaixo |
| Telemetria do endpoint (EDR) | Criação de processos (`schtasks`, `sc`, `crontab`, `systemctl`) com linha de comando; criação e alteração de arquivos em diretórios de tarefas, cron e systemd; alterações no registro de serviços | 30 dias | Sim, **ou** a fonte acima |
| Inventário de tarefas e serviços | Lista atual de tarefas e serviços de cada host, com caminho do executável e assinatura | — | Complementar |
| Autenticação | Logons de rede no host no momento da criação (para identificar criação remota) | 30 dias | Complementar |

**Escopo de ativos:** estações de trabalho e servidores Windows e Linux.

**Requisito crítico:** no Windows, os eventos 4698, 4702 e 4697 exigem que a auditoria correspondente esteja habilitada. Sem ela, só o evento 7045 (serviços) é registrado por padrão, e a criação de tarefas fica invisível, a menos que haja telemetria de endpoint.

## 6. Comportamento normal (baseline)

Tarefas e serviços legítimos são criados por instalação e atualização de software, ferramentas de gestão, agentes de segurança e de monitoramento, e por administradores. Muitos atualizadores de software instalados pelo próprio usuário criam tarefas que executam binários em `AppData`, o que é o principal falso positivo do indicador de "diretório gravável por usuário".

**Como levantar o baseline em um cliente novo:**

1. Faça o stacking das tarefas e dos serviços existentes, por **nome** e por **caminho do executável**, com a quantidade de hosts em que cada um aparece.
2. Os itens presentes em muitos hosts, ligados a softwares conhecidos e com executáveis assinados, formam o baseline.
3. Identifique os processos e as contas que normalmente criam tarefas e serviços (instaladores, ferramentas de gestão) e registre-os como exceções do cliente, fora deste repositório.
4. No Linux, registre os diretórios e as unidades gerenciados por ferramentas de gestão de configuração.

## 7. Estratégia de busca

**Técnicas de análise:** stacking, busca por padrões e enriquecimento.

**Lógica:**

1. Coletar as criações e alterações de tarefas agendadas e serviços no período, pelos eventos do sistema operacional ou pela telemetria de endpoint.
2. Para cada item, extrair: host, nome, ação ou caminho do executável, argumentos, conta e processo criador, e se a criação foi local ou remota.
3. Marcar os indicadores:
   - **raro:** a combinação (nome, caminho) aparece em poucos hosts do ambiente;
   - **diretório gravável:** executável em diretórios como `\Users\*\AppData\`, `\Users\Public\`, `\Windows\Temp\`, `\ProgramData\`, `\Downloads\`; no Linux, `/tmp/`, `/var/tmp/`, `/dev/shm/` ou diretórios pessoais;
   - **interpretador como ação:** a ação executa PowerShell, cmd, `mshta`, `wscript`, `cscript`, `rundll32`, `regsvr32`, ou, no Linux, um shell com comando inline, `curl` ou `wget`;
   - **conteúdo codificado:** argumentos com base64 ou ofuscação (ver [TH-EXE-001](../execution/th-exe-001-interpretadores-comandos-ofuscados.md));
   - **criação remota:** criada a partir de outro host (por exemplo, `schtasks /create /s`, criação de serviço via rede, ou logon de rede imediatamente antes);
   - **criador incomum:** criada por processo ou conta que normalmente não cria tarefas ou serviços (aplicativo de escritório, navegador, conta de usuário comum, serviço web);
   - **nome imitando o legítimo:** nome parecido com o de tarefas ou serviços do sistema, mas com caminho diferente do habitual;
   - **executável não assinado** ou assinado por entidade desconhecida (quando houver essa informação).
4. Priorizar os itens raros com pelo menos mais um indicador.
5. Enriquecer com a atividade do executável da tarefa ou do serviço (o que ele faz quando roda, para onde se conecta).

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Raridade da combinação (nome, caminho) | Presente em ≤ 2 hosts do ambiente |
| Janela entre logon de rede e criação (criação remota) | ≤ 5 minutos |
| Itens criados por um mesmo criador incomum | Qualquer ocorrência |

**Prioridade:** criação remota, interpretador com conteúdo codificado e serviço com executável em diretório gravável já justificam a triagem mesmo sem raridade.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Windows: eventos 4698, 4702 (tarefas), 7045, 4697 (serviços)
            Linux: alterações em /etc/cron*, /var/spool/cron/, /etc/systemd/system/,
                   /lib/systemd/system/, /usr/lib/systemd/system/, ~/.config/systemd/user/
            Alternativa: telemetria de endpoint com os mesmos eventos
            Últimos 30 dias

FILTRO:     Excluir criadores (processo, conta) da lista de exceções do cliente
            Linux: excluir alterações feitas por gerenciadores de pacotes

CAMPOS:     host, tipo (tarefa / serviço / cron / timer / unidade systemd),
            nome, executavel (caminho), argumentos, conta_criadora,
            processo_criador, origem_criacao (local / remota)

STACKING:   hosts_com_item = contagem distinta de host por (nome, executavel)
            no ambiente inteiro (itens existentes + criados no período)

INDICADORES:
            raro          = hosts_com_item <= 2
            dir_gravavel  = executavel em \AppData\, \Users\Public\, \Temp\, \ProgramData\,
                            \Downloads\, /tmp/, /var/tmp/, /dev/shm/, /home/
            interpretador = executavel ou argumentos com powershell, pwsh, cmd /c, mshta,
                            wscript, cscript, rundll32, regsvr32, sh -c, bash -c, curl, wget
            codificado    = argumentos com bloco base64 ou ofuscação (ver TH-EXE-001)
            remota        = origem_criacao = remota
                            OU logon de rede no host <= 5 min antes da criação
            criador_raro  = processo_criador ou conta_criadora fora do baseline de criadores

DESTAQUE:   remota OU (interpretador E codificado) OU (tipo = serviço E dir_gravavel)
            OU (raro E qualquer outro indicador)
            (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      host, tipo, nome, executavel, argumentos, conta_criadora, processo_criador,
            origem_criacao, hosts_com_item, indicadores presentes
            Ordenar por quantidade de indicadores e raridade
```

**Observações:**

- No evento 4698, a ação da tarefa está no XML do campo de conteúdo da tarefa; extraia o comando e os argumentos antes do stacking.
- O stacking deve considerar também as tarefas e os serviços **já existentes** nos hosts (inventário), não só os criados no período: uma persistência antiga continua sendo persistência.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-per-001-tarefas-servicos-fora-do-padrao.yml`](th-per-001-tarefas-servicos-fora-do-padrao.yml)

O arquivo contém quatro regras, com sintaxe verificada por um analisador Sigma de código aberto:

| Regra | Sistema | Nível |
|-------|---------|-------|
| Tarefa agendada com ação suspeita (evento 4698) | Windows | medium |
| Serviço instalado com caminho ou comando suspeito (evento 7045) | Windows | high |
| Criação de tarefa por linha de comando com ação suspeita ou em host remoto | Windows | medium |
| Criação ou alteração de unidades systemd e de arquivos cron | Linux | low |

Limitações da versão Sigma em relação à pseudoconsulta:

- A **raridade** (stacking entre hosts) e o **criador incomum** dependem do baseline e não são expressos nas regras; são o centro do hunt e ficam na execução.
- A regra de Linux é propositalmente ampla (nível low): serve para alimentar o stacking, não como alerta.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Dados de teste (somente em laboratório ou com autorização formal):**

1. Em um host Windows de teste, criar uma tarefa agendada que execute um script inofensivo a partir de `\Users\Public\`, e outra cuja ação seja PowerShell com comando codificado inofensivo.
2. Criar um serviço que aponte para um executável inofensivo em diretório temporário; criar uma tarefa em outro host de teste com `schtasks /create /s`.
3. Em um host Linux de teste, adicionar uma linha de cron e criar uma unidade systemd de usuário que executem um comando inofensivo a partir de `/tmp/`.
4. Confirmar que todos os itens aparecem destacados e que as tarefas e os serviços legítimos do baseline não aparecem.
5. Remover os itens de teste ao final.

Alternativa sem laboratório: usar datasets públicos de telemetria gravados durante simulações de técnicas de persistência.

**Validação histórica:** executar o hunt em pelo menos 30 dias de dados reais, com o inventário atual dos hosts, com autorização, para medir o ruído e montar a lista de exceções do ambiente.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | O item pertence a software conhecido e legítimo, com executável assinado; foi criado pelo instalador ou pela ferramenta de gestão do cliente; o comportamento do executável é compatível. Incluir nas exceções do cliente quando recorrente. |
| **Suspeito — investigar** | Item raro sem explicação imediata; executável em diretório gravável, mas de software possivelmente legítimo; criador incomum com justificativa possível. |
| **Malicioso — escalar** | A ação executa conteúdo codificado ou baixa código; o executável é desconhecido, não assinado ou malicioso; a criação foi remota sem justificativa administrativa; o nome imita um item do sistema com caminho diferente; o executável se conecta a destino de reputação ruim. |

**Perguntas de enriquecimento:**

- O que o executável ou o comando faz quando roda? Para onde se conecta?
- Quem criou o item, de onde e quando? Houve logon de rede logo antes?
- O executável é assinado? Por quem? Ele aparece em outros hosts?
- O nome imita algum item legítimo? O caminho é o esperado para esse nome?
- O que aconteceu no host pouco antes da criação? (ver [TH-EXE-001](../execution/th-exe-001-interpretadores-comandos-ofuscados.md) e [TH-INI-001](../initial-access/th-ini-001-acesso-remoto-origem-incomum.md))
- O cliente reconhece o item? (consultar o ponto focal)

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Atualizadores de software instalados por usuário | Tarefa executando binário assinado em `AppData`, com nome do fabricante do software; presente em vários hosts | Exceção por caminho e assinatura |
| Ferramentas de gestão e de implantação | Criador é o agente da ferramenta; padrão repetido em muitos hosts | Exceção por processo criador e conta |
| Instaladores com serviços temporários | Serviço criado e removido em minutos durante uma instalação conhecida | Verificar a instalação; exceção específica |
| Administração de servidores Linux | Alterações em cron e systemd feitas por administradores conhecidos ou por ferramenta de gestão de configuração | Exceção por conta ou por ferramenta |
| Agentes de segurança e de monitoramento | Serviços dos agentes instalados no ambiente | Exceção por caminho e assinatura |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): definição completa da tarefa ou do serviço, executável (hash e, se possível, cópia), criador e linha do tempo.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Persistência maliciosa confirmada é **Alta**, e **Crítica** se envolver ativos críticos, criação remota em vários hosts ou executável ativo com conexão de retorno.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - isolar o host;
   - desabilitar a tarefa ou o serviço **depois** de preservar as evidências;
   - bloquear o executável e os destinos de conexão;
   - procurar o mesmo nome, caminho ou hash em todos os hosts do cliente;
   - se a criação foi remota, investigar o host e a conta de origem (movimentação lateral).

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Auditoria de criação de tarefas (4698) ou de instalação de serviços (4697) desabilitada | Média | Habilitar a auditoria correspondente |
| Serviços com caminho sem aspas e com espaços | Média | Corrigir o caminho (risco de escalação de privilégios) |
| Serviços ou tarefas executando como conta privilegiada sem necessidade | Média | Aplicar menor privilégio |
| Tarefas e serviços órfãos, apontando para executáveis inexistentes | Baixa | Remover os itens órfãos |
| Diretórios de cron ou systemd com permissão de escrita para usuários comuns | Média | Corrigir as permissões |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Parcialmente |
| **Ruído permite alerta contínuo?** | Serviço instalado com executável em diretório gravável ou com interpretador e criação remota de tarefas costumam ter ruído baixo depois das exceções e são bons candidatos a alerta. A raridade entre hosts depende do stacking periódico e continua no hunt manual. |
| **Tipos de plataforma alvo** | SIEM, EDR |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1053 Scheduled Task/Job](https://attack.mitre.org/techniques/T1053/)
- MITRE ATT&CK — [T1053.005 Scheduled Task](https://attack.mitre.org/techniques/T1053/005/)
- MITRE ATT&CK — [T1053.003 Cron](https://attack.mitre.org/techniques/T1053/003/)
- MITRE ATT&CK — [T1053.006 Systemd Timers](https://attack.mitre.org/techniques/T1053/006/)
- MITRE ATT&CK — [T1543.003 Windows Service](https://attack.mitre.org/techniques/T1543/003/)
- MITRE ATT&CK — [T1543.002 Systemd Service](https://attack.mitre.org/techniques/T1543/002/)
- MITRE ATT&CK — [T1569.002 Service Execution](https://attack.mitre.org/techniques/T1569/002/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
