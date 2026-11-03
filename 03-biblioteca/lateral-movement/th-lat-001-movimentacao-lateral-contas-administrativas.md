# TH-LAT-001 — Movimentação lateral com contas administrativas entre hosts

| Campo | Valor |
|-------|-------|
| **ID** | TH-LAT-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Lateral Movement (TA0008) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 3–5 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, autenticações de contas administrativas e privilegiadas que indiquem movimentação lateral: pares **conta → host de destino** nunca vistos no histórico, uma mesma conta entrando em muitos hosts em pouco tempo, logons administrativos entre estações de trabalho, uso de credenciais explícitas ou de material de autenticação alternativo e execução remota de comandos.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** com credenciais administrativas em mãos (muitas vezes extraídas da memória de um host, ver [TH-CRD-001](../credential-access/th-crd-001-acesso-memoria-processo-autenticacao.md)), o adversário passa a se mover pelo ambiente usando os **mesmos mecanismos que os administradores usam**: compartilhamentos administrativos, área de trabalho remota, gerenciamento remoto, execução remota de serviços. Cada passo parece uma tarefa administrativa normal.

O que denuncia a movimentação é o **caminho**: administradores legítimos seguem padrões estáveis (as mesmas contas acessam os mesmos servidores a partir dos mesmos hosts de administração). Um adversário usa a conta de forma diferente: a partir de uma estação comum, para hosts que a conta nunca acessou, em sequência rápida e às vezes com material de autenticação alternativo, como o hash da senha em vez da senha. Por isso o centro deste hunt é o histórico de **quem acessa o quê, de onde**.

**Ameaças associadas:** o MITRE ATT&CK registra o uso de serviços remotos com contas válidas e de material de autenticação alternativo (como pass the hash) por um grande número de grupos e operadores de ransomware, frequentemente com ferramentas de execução remota como o PsExec (S0029). Ver referências.

## 3. Hipótese

- **Dado que** adversários usam credenciais administrativas obtidas no ambiente para alcançar outros hosts por serviços remotos,
- **quando** eles se movem lateralmente,
- **então** aparecerão pares conta → host de destino nunca vistos, contas autenticando em muitos hosts em pouco tempo, logons administrativos entre estações de trabalho e sinais de material de autenticação alternativo ou de execução remota de comandos.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Lateral Movement | Remote Services: Remote Desktop Protocol | T1021.001 |
| Lateral Movement | Remote Services: SMB/Windows Admin Shares | T1021.002 |
| Lateral Movement | Remote Services: Windows Remote Management | T1021.006 |
| Lateral Movement | Use Alternate Authentication Material: Pass the Hash | T1550.002 |
| Lateral Movement | Valid Accounts: Domain Accounts | T1078.002 |
| Execution | Windows Management Instrumentation | T1047 |
| Execution | System Services: Service Execution | T1569.002 |

A técnica principal é a **T1021**. A **T1550.002** também pertence à tática Stealth no ATT&CK. A **T1047** e a **T1569.002** entram quando a movimentação é feita por execução remota de comandos.

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Logs de autenticação dos hosts | Windows: logon bem-sucedido (4624) com tipo de logon, conta, host de origem, pacote de autenticação e processo de logon; logon com credenciais explícitas (4648); privilégios especiais no logon (4672) | 30 dias, mais 30 dias anteriores como histórico | Sim |
| Logs de autenticação dos controladores de domínio | Emissão de tíquetes e validação de credenciais, com origem | 30 dias | Complementar |
| Inventário de contas privilegiadas e de hosts de administração | Quem é administrador, quais hosts são de administração, quais são servidores e estações | — | Complementar, **muito recomendada** |
| Telemetria do endpoint (EDR) | Processos iniciados por logons remotos; execução remota de comandos; instalação de serviços | 30 dias | Complementar |

**Escopo de ativos:** estações de trabalho, servidores e controladores de domínio.

**Requisito crítico:** o **histórico** e o **inventário**. O indicador "nunca visto" exige pelo menos 30 dias de autenticações **anteriores** ao período analisado. E sem saber quais contas são privilegiadas e quais hosts são estações ou servidores, não é possível destacar logons administrativos entre estações.

## 6. Comportamento normal (baseline)

A movimentação legítima vem de administradores, ferramentas de gestão e de implantação, soluções de backup e de inventário, monitoramento e suporte remoto. Ela tende a ser **estável**: as mesmas contas acessam os mesmos conjuntos de hosts, a partir dos mesmos hosts de administração. Contas de serviço de gestão podem acessar centenas de hosts por dia de forma legítima.

**Como levantar o baseline em um cliente novo:**

1. Com pelo menos 30 dias de autenticações, monte o histórico de pares **(conta, host de origem, host de destino, tipo de logon)** para as contas privilegiadas.
2. Classifique os hosts em **estações**, **servidores**, **controladores de domínio** e **hosts de administração**.
3. Identifique as contas de serviço de gestão, backup e inventário que acessam muitos hosts e registre-as como exceções do cliente, fora deste repositório, com o conjunto de origens esperado.
4. Registre o espalhamento típico (quantidade de hosts de destino por dia) de cada conta administrativa.

## 7. Estratégia de busca

**Técnicas de análise:** comparação com o histórico ("primeira vez"), análise de grafo (quem acessa o quê, de onde), agrupamento temporal e enriquecimento.

**Lógica:**

1. Coletar os logons bem-sucedidos de rede (tipo 3) e remotos interativos (tipo 10) de contas de usuário no período, excluindo contas de máquina e logons anônimos.
2. Para cada logon, marcar os indicadores:
   - **par novo:** o par (conta, host de destino) não aparece no histórico de 30 dias anteriores;
   - **origem nova:** o par (conta, host de origem) não aparece no histórico;
   - **espalhamento:** a conta autenticou em muitos hosts distintos em pouco tempo;
   - **estação para estação:** conta privilegiada autenticando de uma estação de trabalho em outra estação de trabalho;
   - **origem fora da administração:** conta privilegiada usada a partir de host que não é de administração;
   - **material alternativo:** logon do tipo 9 pelo processo `seclogo` com pacote Negotiate, ou logon de rede NTLM de conta privilegiada onde normalmente se usa Kerberos;
   - **credenciais explícitas:** uso de credenciais explícitas (4648) para conta privilegiada a partir de host incomum;
   - **execução remota:** instalação de serviço de execução remota, uso de WMI, WinRM ou PowerShell Remoting para iniciar processos em outro host.
3. Montar a **sequência**: ordenar os logons destacados no tempo e ligar host de origem a host de destino. Uma cadeia (A → B → C) a partir de uma origem incomum é o padrão mais característico de movimentação lateral.
4. Priorizar as cadeias e os logons com mais de um indicador.
5. Enriquecer com o que aconteceu em cada host de destino depois do logon.

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Histórico para "par novo" e "origem nova" | 30 dias anteriores ao período analisado |
| Espalhamento | ≥ 10 hosts de destino distintos em 1 hora |
| Logons de estação para estação de conta privilegiada | Qualquer ocorrência fora de exceção |
| Cadeia de movimentação | ≥ 2 saltos (A → B → C) com pares novos |

**Prioridade:** material de autenticação alternativo, logons de conta privilegiada entre estações e cadeias com pares novos já justificam a triagem, mesmo isolados.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Logs de autenticação dos hosts (4624, 4648, 4672)
            Período analisado: últimos 30 dias
            Histórico: 30 dias anteriores ao período

FILTRO:     resultado = sucesso
            tipo_logon em (3, 10)     para a análise principal
            tipo_logon = 9            para material alternativo
            Excluir contas de máquina (terminadas em $) e ANONYMOUS LOGON
            Excluir contas de serviço de gestão, nas origens esperadas
            (lista de exceções do cliente)

ENRIQUECER: host_origem, host_destino -> tipo_host (estação, servidor, controlador,
                                         administração)
            conta -> privilegiada (sim/não)

INDICADORES, por logon:
            par_novo       = (conta, host_destino) ausente do histórico
            origem_nova    = (conta, host_origem) ausente do histórico
            espalhamento   = destinos distintos da conta em 1 h >= 10
            estacao_estacao = privilegiada E origem = estação E destino = estação
            origem_nao_adm = privilegiada E tipo_host(origem) <> administração
            material_alt   = (tipo_logon = 9 E processo_logon = seclogo
                              E pacote = Negotiate)
                             OU (privilegiada E pacote = NTLM onde o baseline é Kerberos)
            cred_explicita = 4648 para conta privilegiada a partir de origem nova
            exec_remota    = serviço de execução remota instalado no destino
                             OU WMI/WinRM/PowerShell Remoting iniciando processo no destino

SEQUÊNCIA:  Ordenar logons destacados por data_hora e ligar origem -> destino
            cadeia = sequência com >= 2 saltos e pares novos

DESTAQUE:   material_alt OU estacao_estacao OU cadeia
            OU (par_novo E (origem_nao_adm OU espalhamento OU exec_remota))
            OU contagem de indicadores >= 3
            (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      conta, privilegiada, data_hora, host_origem, host_destino, tipo_logon,
            pacote, indicadores presentes, cadeia (se houver)
            Ordenar por cadeia e quantidade de indicadores
```

**Observações:**

- O host de origem nem sempre é registrado com nome; quando vier como endereço IP, resolva-o para o host com base no inventário ou nos registros de DHCP do período.
- Um grafo de conta, origem e destino ajuda muito na triagem: plataformas e ferramentas de análise que permitem visualizar relações tornam as cadeias evidentes.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-lat-001-movimentacao-lateral-contas-administrativas.yml`](th-lat-001-movimentacao-lateral-contas-administrativas.yml)

O arquivo contém cinco regras, com sintaxe verificada por um analisador Sigma de código aberto:

| Regra | Nível |
|-------|-------|
| Logon de rede ou remoto interativo de conta de usuário (base da correlação) | informational |
| **Correlação:** mesma conta autenticando em 10 ou mais hosts em 1 hora | medium |
| Logon com credenciais alternativas indicativo de pass the hash | high |
| Serviço de execução remota instalado no host (incluindo o do PsExec, S0029) | medium |
| Execução remota de comandos por WMI, WinRM ou PowerShell Remoting | medium |

Limitações da versão Sigma em relação à pseudoconsulta:

- O indicador central, **par conta → host nunca visto**, depende de histórico e não é expresso em Sigma.
- A classificação de hosts (estação, servidor, administração) e de contas privilegiadas depende do inventário do cliente e é feita na execução do hunt.
- A montagem de cadeias de movimentação exige análise de sequência, fora do alcance das regras.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Telemetria esperada:** os logons de rede e remotos interativos nos hosts de destino, com origem, conta, tipo de logon e pacote de autenticação; os eventos de credenciais explícitas na origem; e, para execução remota, a instalação de serviço ou o processo iniciado no destino.

**Como gerar os dados (somente em laboratório ou com autorização formal):** em um domínio de laboratório com histórico acumulado, usar uma conta administrativa de teste a partir de uma estação comum para acessar, em sequência, compartilhamentos administrativos e o gerenciamento remoto de dois ou três hosts que ela nunca acessou. Para material de autenticação alternativo, use ferramentas de emulação de adversário com testes mapeados à T1550.002, em laboratório isolado e sem credenciais reais. Confirme que os pares novos, a cadeia e os demais indicadores aparecem destacados.

**Alternativa sem laboratório:** usar datasets públicos de telemetria gravados durante simulações de movimentação lateral.

**Validação histórica:** executar o hunt em pelo menos 30 dias de dados reais, com mais 30 dias de histórico, com autorização, para calibrar o espalhamento e montar as exceções de contas de serviço.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | O acesso corresponde a mudança planejada, novo servidor, nova atribuição do administrador ou implantação em massa documentada; a origem é host de administração; o padrão é confirmado pelo ponto focal. |
| **Suspeito — investigar** | Par novo de conta administrativa a partir de host de administração, sem mudança registrada; espalhamento de conta de serviço fora do horário habitual. |
| **Malicioso — escalar** | Material de autenticação alternativo; conta privilegiada entre estações de trabalho; cadeia de movimentação a partir de origem incomum; logons precedidos de acesso a credenciais no host de origem; execução remota de comandos sem justificativa; o administrador não reconhece o acesso. |

**Perguntas de enriquecimento:**

- O dono da conta reconhece os acessos? (confirmar pelo ponto focal)
- O host de origem é de administração? O que aconteceu nele antes dos logons? (ver [TH-CRD-001](../credential-access/th-crd-001-acesso-memoria-processo-autenticacao.md) e [TH-DIS-001](../discovery/th-dis-001-enumeracao-diretorio-em-rajada.md))
- O que a conta fez em cada host de destino depois do logon?
- Há outras contas usadas a partir da mesma origem no mesmo período?
- Houve instalação de serviços ou criação de tarefas nos destinos? (ver [TH-PER-001](../persistence/th-per-001-tarefas-servicos-fora-do-padrao.md))

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Contas de serviço de gestão, backup, inventário e monitoramento | Acessam muitos hosts de forma periódica e estável, a partir de servidores conhecidos | Exceção por conta e conjunto de origens |
| Administradores em tarefas novas | Pares novos a partir de host de administração, durante mudança registrada | Confirmar com o ponto focal |
| Implantações em massa | Muitos hosts em pouco tempo, por ferramenta de implantação, em janela de mudança | Exceção temporária pela janela |
| Novos servidores e recém-chegados ao time | Primeiros acessos de contas legítimas | Atualizar o histórico; confirmar |
| Suporte remoto a estações | Técnicos acessando estações; em geral com contas de suporte, não de domínio | Revisar o modelo de contas de suporte |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): logons, cadeia de movimentação, contas e hosts envolvidos e atividade em cada destino.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Movimentação lateral confirmada é **Crítica**: indica adversário ativo se expandindo no ambiente.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - desabilitar ou redefinir as contas usadas, priorizando as privilegiadas, e encerrar as sessões ativas;
   - isolar os hosts de origem e de destino envolvidos;
   - mapear todos os hosts alcançados pela conta no período e investigar cada um;
   - investigar como a credencial foi obtida (ver [TH-CRD-001](../credential-access/th-crd-001-acesso-memoria-processo-autenticacao.md));
   - avaliar, com a resposta a incidentes, a necessidade de redefinir credenciais de alto privilégio do domínio.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Administradores do domínio fazendo logon em estações de trabalho | Média | Adotar modelo de camadas de administração |
| Ausência de hosts dedicados de administração | Média | Implementar estações de administração dedicadas |
| Mesma senha de administrador local em vários hosts | Alta | Implementar senhas locais únicas e gerenciadas |
| NTLM amplamente usado onde Kerberos seria possível | Baixa | Planejar a redução do NTLM |
| Compartilhamentos administrativos e gerenciamento remoto liberados entre estações | Média | Restringir o tráfego de administração entre estações |
| Auditoria de logon sem informação do host de origem | Média | Ajustar a auditoria e a coleta |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Parcialmente |
| **Ruído permite alerta contínuo?** | Material de autenticação alternativo e logons de conta privilegiada entre estações têm ruído baixo e são bons candidatos a alerta. O espalhamento pode virar alerta com as exceções de contas de serviço. O par nunca visto e as cadeias dependem de histórico e continuam no hunt manual, ou em análises de comportamento de usuários e entidades. |
| **Tipos de plataforma alvo** | SIEM |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1021 Remote Services](https://attack.mitre.org/techniques/T1021/)
- MITRE ATT&CK — [T1021.002 SMB/Windows Admin Shares](https://attack.mitre.org/techniques/T1021/002/)
- MITRE ATT&CK — [T1021.006 Windows Remote Management](https://attack.mitre.org/techniques/T1021/006/)
- MITRE ATT&CK — [T1550.002 Pass the Hash](https://attack.mitre.org/techniques/T1550/002/)
- MITRE ATT&CK — [T1078.002 Domain Accounts](https://attack.mitre.org/techniques/T1078/002/)
- MITRE ATT&CK — [T1047 Windows Management Instrumentation](https://attack.mitre.org/techniques/T1047/)
- MITRE ATT&CK — [S0029 PsExec](https://attack.mitre.org/software/S0029/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
