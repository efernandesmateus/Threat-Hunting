# TH-C2-001 — Beaconing: conexões periódicas para destinos raros

| Campo | Valor |
|-------|-------|
| **ID** | TH-C2-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Command and Control (TA0011) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 3–5 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, hosts que se comunicam com destinos externos **raros no ambiente** em intervalos **regulares** (beaconing), padrão típico de malware e de ferramentas de acesso remoto mantendo contato com a infraestrutura do adversário.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** depois de instalado, o implante do adversário precisa receber ordens. Na maior parte dos casos, é o próprio implante que **pergunta**, de tempos em tempos, se há algo novo: ele se conecta ao servidor de comando e controle (C2) em intervalos programados, geralmente por protocolos web comuns e criptografados, para se misturar ao tráfego normal. Para disfarçar a regularidade, muitos implantes acrescentam uma variação aleatória (jitter) ao intervalo.

Mesmo com jitter, o padrão continua **estatisticamente regular**: muitas conexões, do mesmo host para o mesmo destino, em intervalos parecidos, com volume de dados parecido. Isso é muito diferente da navegação humana, que é irregular e concentrada em horários de trabalho.

O problema é que muita coisa **legítima** também é periódica: atualizadores de software, telemetria, sincronização de arquivos, verificação de licenças, agentes de monitoramento. A diferença costuma estar na **distribuição**: serviços legítimos são consultados por muitos hosts do ambiente, enquanto um C2 costuma aparecer em um ou poucos hosts. Por isso este hunt combina **regularidade** com **raridade do destino**.

**Ameaças associadas:** o MITRE ATT&CK registra o uso de protocolos web, canais criptografados e portas não padronizadas para comando e controle por praticamente todos os grupos e famílias de malware que mantêm acesso remoto. Ver referências.

## 3. Hipótese

- **Dado que** implantes de adversários se comunicam com a infraestrutura de comando e controle em intervalos programados,
- **quando** um host comprometido mantém contato com o C2,
- **então** aparecerão pares host → destino com muitas conexões em intervalos regulares (baixa variação), volume de dados semelhante por conexão e destinos consultados por poucos hosts do ambiente, frequentemente novos ou de baixa reputação.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Command and Control | Application Layer Protocol: Web Protocols | T1071.001 |
| Command and Control | Encrypted Channel | T1573 |
| Command and Control | Non-Standard Port | T1571 |
| Command and Control | Non-Application Layer Protocol | T1095 |

A técnica mais comum é a **T1071.001** combinada com a **T1573** (C2 por HTTPS). A **T1571** e a **T1095** cobrem os casos em que o adversário usa portas ou protocolos fora do padrão; o hunt as encontra pela mesma análise de regularidade.

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Logs de proxy ou firewall | Data e hora de cada conexão, host de origem, destino (domínio ou IP), porta, protocolo, bytes enviados e recebidos; no proxy, também URL, método e agente de usuário | 30 dias | Sim |
| Logs DNS | Consultas do host de origem, para associar IPs a domínios | 30 dias | Complementar |
| Telemetria do endpoint (EDR) | Conexões de rede com o processo responsável | 30 dias | Complementar |
| Reputação e idade de domínios e IPs | Data de registro, reputação, categoria, sistema autônomo | — | Complementar |

**Escopo de ativos:** todos os hosts com acesso à internet.

**Requisitos críticos:**

- **Registro por conexão**, e não apenas totais agregados por período. A análise de regularidade precisa do horário de cada conexão.
- **Host de origem real.** Se o tráfego passa por tradução de endereços (NAT) antes do ponto de coleta, o log mostra o endereço traduzido e o hunt não consegue isolar o host.

## 6. Comportamento normal (baseline)

Tráfego periódico legítimo é muito comum. Os principais exemplos são atualizadores de sistema operacional e de aplicativos, telemetria de software, sincronização de arquivos e de e-mail, verificação de licenças, agentes de segurança e de monitoramento e aplicativos de comunicação.

O que esses serviços têm em comum é serem consultados por **muitos** hosts, a partir de **processos conhecidos**, para destinos **com reputação estabelecida**.

**Como levantar o baseline em um cliente novo:**

1. Em pelo menos 14 dias, calcule para cada **destino** a quantidade de hosts distintos que o acessam.
2. Os destinos periódicos acessados por grande parte do ambiente e ligados a serviços conhecidos formam o baseline. Registre-os como exceções do cliente, fora deste repositório.
3. Registre os destinos legítimos de nicho (aplicações específicas de alguns departamentos), que serão periódicos e raros ao mesmo tempo.

## 7. Estratégia de busca

**Técnicas de análise:** análise estatística de intervalos, stacking (raridade do destino) e enriquecimento.

**Lógica:**

1. **Raridade:** para cada destino, contar os hosts distintos que o acessaram nos últimos 30 dias. Manter apenas os destinos raros (poucos hosts) e fora das exceções do cliente.
2. **Regularidade:** para cada par **(host, destino)** restante, por dia:
   - ordenar as conexões por horário;
   - calcular os **intervalos** entre conexões consecutivas;
   - calcular a **média** e o **desvio padrão** dos intervalos;
   - calcular o **coeficiente de variação** (CV = desvio padrão ÷ média). Quanto mais perto de zero, mais regular. Um beacon com jitter de 20% costuma ter CV em torno de 0,1 a 0,2; a navegação humana costuma passar de 1.
3. **Volume:** calcular também o coeficiente de variação dos **bytes** enviados por conexão. Beacons ociosos costumam trocar quase sempre o mesmo volume.
4. **Persistência:** verificar se o padrão se repete em vários dias e se ocorre também fora do horário de trabalho (implantes não dormem).
5. **Enriquecimento:** idade e reputação do destino, porta e protocolo, agente de usuário (no proxy), processo responsável (se houver EDR).
6. Ordenar pelos pares mais regulares, mais persistentes e com destino mais raro.

**Beacons de intervalo longo:** alguns implantes se comunicam a cada várias horas ou uma vez por dia. Para eles, a análise diária tem poucas conexões; repita a análise com janela de **7 dias**, exigindo menos conexões por dia, mas presença na maioria dos dias.

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Raridade do destino | Acessado por ≤ 3 hosts nos últimos 30 dias |
| Conexões por par (host, destino) em 24 h | ≥ 20 |
| Coeficiente de variação dos intervalos | ≤ 0,25 |
| Coeficiente de variação dos bytes enviados | ≤ 0,3 (indicador de reforço) |
| Persistência | Padrão presente em ≥ 3 dias, ou fora do horário de trabalho |
| Beacon de intervalo longo (janela de 7 dias) | ≥ 1 conexão em ≥ 5 dos 7 dias, com intervalos de CV ≤ 0,25 |
| Idade do domínio (enriquecimento) | < 30 dias aumenta a suspeita |

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Logs de proxy ou firewall (uma linha por conexão), últimos 30 dias

FILTRO:     destino externo
            Excluir destinos da lista de exceções do cliente

RARIDADE:   hosts_por_destino = contagem distinta de host por destino (30 dias)
            manter destinos com hosts_por_destino <= 3

POR PAR (host, destino), POR DIA:
            conexoes       = lista de data_hora, ordenada
            n              = quantidade de conexões
            intervalos     = diferenças entre conexões consecutivas (segundos)
            media_int      = média(intervalos)
            desvio_int     = desvio_padrão(intervalos)
            cv_int         = desvio_int ÷ media_int
            cv_bytes       = desvio_padrão(bytes_enviados) ÷ média(bytes_enviados)
            fora_horario   = existe conexão fora do horário de trabalho

PERSISTÊNCIA, por par, no período:
            dias_com_padrao = dias com n >= 20 E cv_int <= 0,25

DESTAQUE:   (n >= 20 E cv_int <= 0,25 E (dias_com_padrao >= 3 OU fora_horario))
            OU beacon_longo
            beacon_longo = em janela de 7 dias: conexões em >= 5 dias
                           E cv dos intervalos entre conexões <= 0,25
            (valores de referência; calibrar pelo baseline do cliente)

REFORÇO:    cv_bytes <= 0,3; domínio com menos de 30 dias; porta não padrão;
            agente de usuário raro ou ausente; reputação ruim

SAÍDA:      host, destino, porta, n, media_int, cv_int, cv_bytes, dias_com_padrao,
            fora_horario, hosts_por_destino, idade e reputação do destino,
            processo (se disponível)
            Ordenar por cv_int (crescente), dias_com_padrao e raridade
```

**Observações:**

- Conexões muito próximas (rajadas de várias requisições em poucos segundos) distorcem os intervalos. Agrupe conexões separadas por menos de alguns segundos como uma só antes do cálculo.
- Se a plataforma não calcular desvio padrão sobre sequências, exporte os pares raros e faça o cálculo fora dela, com qualquer ferramenta de análise de dados.

### 8.2 Regra Sigma

**Status:** Não se aplica

A detecção de beaconing depende de **estatística sobre sequências de eventos ao longo do tempo** (intervalos entre conexões e sua variação) e da **raridade do destino no ambiente**. Nenhum desses elementos é expresso no formato Sigma. A execução usa a pseudoconsulta.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Telemetria esperada:** conexões do host de teste para o destino de teste registradas uma a uma, com horário, bytes e, no proxy, URL e agente de usuário.

**Como gerar os dados (somente em laboratório ou com autorização formal):** em um host de teste, programar um script inofensivo que faça requisições HTTPS a um servidor de teste controlado pelo time, em intervalo fixo com variação aleatória (por exemplo, 60 segundos com 20% de variação), por pelo menos 24 horas. Repetir com intervalo longo (por exemplo, a cada 4 horas) por uma semana. Confirme que os dois casos aparecem destacados e que os serviços periódicos do baseline não aparecem.

**Alternativa sem laboratório:** usar datasets públicos de tráfego de rede gravados durante simulações de comando e controle.

**Validação histórica:** executar o hunt em pelo menos 30 dias de dados reais, com autorização, para montar a lista de exceções de serviços periódicos e calibrar os limiares de regularidade.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | O destino pertence a serviço legítimo identificado (atualização, telemetria, sincronização, aplicação de nicho conhecida); o processo responsável é o software correspondente; a reputação é boa. Incluir nas exceções do cliente. |
| **Suspeito — investigar** | Padrão regular para destino raro sem reputação conhecida e sem processo identificado; destino de nuvem pública genérica (que pode hospedar tanto serviços legítimos quanto C2). |
| **Malicioso — escalar** | Destino recente, de reputação ruim ou presente em inteligência de ameaças; processo responsável desconhecido, não assinado ou injetado em processo legítimo; padrão persistente 24 horas por dia; beacon a partir de host com outros achados (execução ofuscada, persistência, proxy de execução). |

**Perguntas de enriquecimento:**

- Qual processo faz as conexões? Ele é assinado? Ele deveria acessar a internet?
- Quando o padrão começou? O que aconteceu no host nesse momento? (ver [TH-EXE-001](../execution/th-exe-001-interpretadores-comandos-ofuscados.md), [TH-STL-001](../stealth/th-stl-001-utilitarios-legitimos-executando-codigo.md) e [TH-PER-001](../persistence/th-per-001-tarefas-servicos-fora-do-padrao.md))
- O destino é recente? Quem o hospeda? O certificado TLS é autoassinado ou recém-emitido?
- O agente de usuário é comum ou raro? É coerente com o processo?
- Outros hosts começaram a acessar o mesmo destino ou destinos com a mesma hospedagem?
- O volume de saída aumentou em algum momento? (ver [TH-EXF-001](../exfiltration/th-exf-001-exfiltracao-via-dns.md) e [TH-COL-001](../collection/th-col-001-compactacao-grandes-volumes.md))

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Atualizadores e telemetria de software | Destino do fabricante do software, processo correspondente, muitos hosts | Exceção por destino |
| Aplicações de nicho | Destino específico de uma aplicação usada por poucos usuários, processo conhecido | Exceção por destino e grupo de hosts |
| Agentes de segurança e de monitoramento | Processo do agente, destino do fornecedor | Exceção por destino |
| Páginas abertas com atualização automática | Navegador acessando o mesmo site em intervalos fixos, em horário de trabalho | Avaliar o site; não criar exceção ampla |
| Scripts de monitoramento internos | Verificação de disponibilidade de serviços externos, por servidor de monitoramento | Exceção por host e destino |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): conexões do par, destino, processo responsável e linha do tempo do host.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Comunicação de comando e controle confirmada é **Crítica**: indica adversário com controle ativo do host.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - isolar o host;
   - bloquear o destino (domínio e IPs) no proxy, no firewall e no DNS;
   - procurar outros hosts que acessaram o mesmo destino ou destinos da mesma infraestrutura;
   - identificar o implante e o mecanismo de persistência no host;
   - tratar o host como totalmente comprometido e investigar o caminho de entrada.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Hosts com acesso direto à internet, sem passar pelo proxy | Média | Forçar a saída pelo proxy |
| Saída permitida em portas arbitrárias | Média | Restringir a saída às portas necessárias |
| Logs de proxy ou firewall agregados, sem registro por conexão | Média | Registrar conexões individualmente |
| Tradução de endereços antes do ponto de coleta, ocultando o host de origem | Média | Coletar antes da tradução ou correlacionar com os registros de NAT |
| Ausência de inspeção ou de categorização de destinos | Baixa | Avaliar categorização e reputação de destinos no proxy |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Parcialmente |
| **Ruído permite alerta contínuo?** | A combinação de regularidade e raridade pode virar uma análise agendada (diária) com boa precisão, depois que as exceções do cliente estiverem estáveis. Como depende de estatística sobre sequências, ela costuma ser implementada como relatório ou análise programada, e não como regra de tempo real. |
| **Tipos de plataforma alvo** | SIEM |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1071.001 Application Layer Protocol: Web Protocols](https://attack.mitre.org/techniques/T1071/001/)
- MITRE ATT&CK — [T1573 Encrypted Channel](https://attack.mitre.org/techniques/T1573/)
- MITRE ATT&CK — [T1571 Non-Standard Port](https://attack.mitre.org/techniques/T1571/)
- MITRE ATT&CK — [T1095 Non-Application Layer Protocol](https://attack.mitre.org/techniques/T1095/)
- MITRE ATT&CK — [Command and Control (TA0011)](https://attack.mitre.org/tactics/TA0011/)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Sem regra Sigma (ver seção 8.2); não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
