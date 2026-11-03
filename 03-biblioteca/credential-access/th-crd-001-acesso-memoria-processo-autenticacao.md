# TH-CRD-001 — Acesso à memória do processo de autenticação do sistema

| Campo | Valor |
|-------|-------|
| **ID** | TH-CRD-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Credential Access (TA0006) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 2–3 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, acessos à memória do processo de autenticação do Windows (LSASS) feitos por processos que normalmente não o acessam, a criação de arquivos de dump desse processo e o uso de técnicas e ferramentas conhecidas de extração de credenciais da memória.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** o LSASS (Local Security Authority Subsystem Service) é o processo do Windows que gerencia a autenticação. Para isso, ele mantém em memória material de credenciais dos usuários que se autenticaram no host: hashes, tíquetes e, em algumas configurações, até senhas recuperáveis. Com privilégios de administrador no host, o adversário pode ler essa memória e extrair credenciais de **outros usuários**, inclusive de administradores que se conectaram ao host. É assim que um único host comprometido se transforma em acesso ao domínio inteiro.

Há duas formas principais de fazer isso: ler a memória do LSASS diretamente, com uma ferramenta de extração de credenciais, ou gerar um **arquivo de dump** da memória do processo e analisá-lo depois, fora do ambiente. A segunda costuma usar componentes nativos do sistema ou utilitários de diagnóstico legítimos, para não levantar suspeita.

Poucos processos legítimos precisam ler a memória do LSASS, e eles tendem a ser sempre os mesmos (agentes de segurança, componentes do próprio sistema). Isso torna o **stacking dos processos que acessam o LSASS** uma das técnicas de hunting com melhor relação entre esforço e resultado.

**Ameaças associadas:** o MITRE ATT&CK registra a extração de credenciais da memória do LSASS por um grande número de grupos e ferramentas, entre elas o Mimikatz (S0002), uma das ferramentas mais conhecidas de extração de credenciais. Ver referências.

**Escopo:** este hunt cobre o Windows. Técnicas equivalentes em Linux (leitura de memória de processos e de arquivos de credenciais) estão no backlog como hunt futuro.

## 3. Hipótese

- **Dado que** adversários extraem credenciais da memória do processo de autenticação do Windows para ampliar o acesso no ambiente,
- **quando** eles leem a memória do LSASS ou geram um dump dela,
- **então** aparecerão acessos ao LSASS com permissões de leitura de memória por processos fora do baseline, arquivos de dump do LSASS criados em disco e linhas de comando de técnicas ou ferramentas conhecidas de extração.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Credential Access | OS Credential Dumping: LSASS Memory | T1003.001 |
| Stealth | System Binary Proxy Execution: Rundll32 (quando o dump usa `comsvcs.dll`) | T1218.011 |

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Telemetria do endpoint (EDR) com acesso entre processos | Processo de origem, processo alvo, máscara de acesso concedida, pilha de chamadas quando disponível | 30 dias | Sim |
| Telemetria de criação de arquivos | Arquivos criados, com caminho e processo criador | 30 dias | Complementar |
| Telemetria de processos | Criação de processos com linha de comando completa | 30 dias | Complementar |
| Logs de autenticação | Uso posterior das credenciais do host em outros hosts | 30 dias | Complementar |

**Escopo de ativos:** estações de trabalho e servidores Windows, com prioridade para servidores e hosts onde administradores se autenticam com frequência.

**Requisito crítico:** a telemetria de **acesso entre processos** (process access) com a máscara de acesso. Sem ela, a parte principal do hunt é inviável; o restante (dumps e linhas de comando) cobre apenas as técnicas que deixam esses rastros.

## 6. Comportamento normal (baseline)

Alguns processos legítimos acessam o LSASS rotineiramente: componentes do próprio sistema, agentes de segurança e de monitoramento, ferramentas de gestão e, durante falhas, o mecanismo de relatório de erros do sistema. Eles aparecem em muitos hosts, a partir dos mesmos caminhos, assinados.

**Como levantar o baseline em um cliente novo:**

1. Em pelo menos 7 dias, faça o stacking dos **processos de origem** que acessaram o LSASS, por caminho, assinatura e máscara de acesso, com a quantidade de hosts.
2. Os processos presentes em muitos hosts, assinados e ligados a componentes do sistema ou a agentes conhecidos, formam o baseline. Registre-os como exceções do cliente, fora deste repositório, sempre pelo **caminho completo e pela assinatura**, nunca apenas pelo nome.
3. Verifique se a proteção do LSASS (execução como processo protegido) e o isolamento de credenciais estão habilitados; isso muda o que é possível e o que é esperado nos dados.

## 7. Estratégia de busca

**Técnicas de análise:** stacking, busca por padrões e enriquecimento.

**Lógica:**

1. **Acesso ao LSASS:** coletar os acessos ao processo `lsass.exe` com máscaras que incluem leitura de memória (por exemplo, `0x1010`, `0x1410`, `0x1438`, `0x143a`, `0x1f1fff`, `0x1f3fff`, `0x1fffff`). Fazer o stacking dos processos de origem e destacar os que estão fora do baseline ou presentes em poucos hosts.
2. **Dumps:** coletar a criação de arquivos cujo nome indica o LSASS e que tenham extensão de dump (`.dmp`, `.dump`), e qualquer arquivo de dump criado por processo que não seja o mecanismo de relatório de erros do sistema.
3. **Técnicas conhecidas por linha de comando:**
   - uso da função de minidump da biblioteca nativa `comsvcs.dll` por meio do `rundll32`;
   - utilitários de diagnóstico com referência ao LSASS e a parâmetros de dump completo;
   - módulos característicos de ferramentas de extração de credenciais do catálogo do ATT&CK (por exemplo, `sekurlsa::` e `lsadump::`, do Mimikatz).
4. Enriquecer cada destaque com o processo de origem (assinatura, caminho, processo-pai), a conta e a atividade do host antes e depois.
5. Para cada host com acesso confirmado, verificar o uso posterior de credenciais: autenticações de contas que estavam em memória no host, a partir dele, em outros hosts.

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Processo de origem raro acessando o LSASS | Presente em ≤ 3 hosts, ou fora do baseline de caminho e assinatura |
| Processo de origem não assinado ou em diretório gravável | Qualquer ocorrência |

**Prioridade:** dumps do LSASS, uso de `comsvcs.dll` para minidump, módulos de ferramentas de extração e acesso por processo não assinado ou em diretório gravável já justificam a triagem imediata, mesmo isolados.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
PARTE A — ACESSO AO LSASS

FONTE:      Acesso entre processos (EDR), últimos 30 dias

FILTRO:     processo_alvo termina em \lsass.exe
            mascara_acesso em (0x1010, 0x1410, 0x1438, 0x143a,
                               0x1f1fff, 0x1f3fff, 0x1fffff)
            Excluir processos de origem da lista de exceções do cliente
            (por caminho completo E assinatura)

STACKING:   hosts_com_origem = contagem distinta de host por (caminho_origem, assinatura)

INDICADORES:
            origem_rara      = hosts_com_origem <= 3
            origem_suspeita  = origem não assinada OU em \AppData\, \Users\Public\,
                               \Temp\, \ProgramData\, \Downloads\
            origem_interprete = origem é interpretador de script ou rundll32

PARTE B — DUMPS E TÉCNICAS CONHECIDAS

FONTE:      Criação de arquivos e de processos (EDR), últimos 30 dias

INDICADORES:
            dump_lsass       = arquivo criado com "lsass" no nome e extensão .dmp ou .dump
            dump_comsvcs     = linha_comando com "comsvcs" E ("minidump" OU "#24")
            dump_utilitario  = linha_comando com "lsass" E (".dmp" OU " -ma " OU "minidump")
            ferramenta       = linha_comando com "sekurlsa::" OU "lsadump::"
                               OU "privilege::debug"

DESTAQUE:   dump_lsass OU dump_comsvcs OU dump_utilitario OU ferramenta
            OU origem_suspeita OU origem_interprete
            OU origem_rara
            (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      host, data_hora, processo_origem (caminho, assinatura, pai), conta,
            mascara_acesso, arquivo de dump, linha_comando, indicadores presentes,
            hosts_com_origem
            Ordenar por gravidade (Parte B e origem suspeita primeiro)
```

**Observações:**

- A máscara de acesso pode aparecer em formatos diferentes (hexadecimal com ou sem zeros à esquerda, maiúsculas ou minúsculas). Normalize antes de comparar.
- Exceções devem usar o **caminho completo e a assinatura**. Adversários podem dar ao seu processo o nome de um processo legítimo do baseline.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-crd-001-acesso-memoria-processo-autenticacao.yml`](th-crd-001-acesso-memoria-processo-autenticacao.yml)

O arquivo contém cinco regras, com sintaxe verificada por um analisador Sigma de código aberto:

| Regra | Nível |
|-------|-------|
| Acesso ao processo LSASS com permissões de leitura de memória | medium |
| Dump de memória via `comsvcs.dll` | high |
| Arquivo de dump do LSASS criado | high |
| Linha de comando com referência a dump do LSASS | high |
| Comandos de ferramenta de extração de credenciais | critical |

Limitações da versão Sigma em relação à pseudoconsulta:

- A primeira regra gera ruído até que as exceções do cliente (por caminho e assinatura) sejam aplicadas na conversão; ela serve de base para o stacking, não como alerta direto.
- A raridade do processo de origem e o uso posterior das credenciais não são expressos nas regras.
- Técnicas mais avançadas (acesso por manipulação de handles, drivers ou leitura sem abertura direta do processo) podem não gerar esses eventos. Este hunt cobre as técnicas mais comuns.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Telemetria esperada:** o evento de acesso entre processos com o LSASS como alvo e a máscara de acesso, a criação do arquivo de dump com o processo criador, e a criação do processo responsável com a linha de comando completa.

**Como gerar os dados (somente em laboratório isolado e com autorização formal):** use ferramentas de emulação de adversário com testes mapeados à subtécnica T1003.001, em hosts de laboratório dedicados, sem credenciais reais. Confirme que cada teste aparece nos resultados com os indicadores esperados e que os processos legítimos do baseline não aparecem. **Nunca** execute extração de credenciais em hosts de produção para testar o hunt.

**Alternativa sem laboratório:** usar datasets públicos de telemetria gravados durante simulações de extração de credenciais do LSASS.

**Validação histórica:** executar o hunt em pelo menos 30 dias de dados reais, com autorização, para montar a lista de exceções de processos que acessam o LSASS no ambiente.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | O processo de origem é componente do sistema ou agente conhecido, assinado, no caminho esperado e presente em muitos hosts; o dump foi gerado pelo mecanismo de relatório de erros do sistema após uma falha registrada do LSASS. |
| **Suspeito — investigar** | Processo de origem assinado, mas raro e sem explicação imediata; dump gerado por suporte técnico sem chamado registrado. |
| **Malicioso — escalar** | Dump do LSASS fora de um processo de falha do sistema; uso de `comsvcs.dll` para minidump; módulos de ferramenta de extração; processo de origem não assinado, em diretório gravável, interpretador de script ou com nome de processo legítimo em caminho diferente. |

**Perguntas de enriquecimento:**

- Qual processo acessou o LSASS? Caminho, assinatura, processo-pai e conta?
- Quais contas estavam com sessão ativa no host no momento (especialmente contas administrativas)?
- O arquivo de dump ainda existe? Foi copiado ou enviado para fora do host?
- As credenciais do host foram usadas em seguida em outros hosts? (sinal de movimentação lateral)
- O que aconteceu antes no host? (ver [TH-EXE-001](../execution/th-exe-001-interpretadores-comandos-ofuscados.md), [TH-STL-001](../stealth/th-stl-001-utilitarios-legitimos-executando-codigo.md) e [TH-DIM-001](../defense-impairment/th-dim-001-desativacao-ferramentas-logs.md))

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Agentes de segurança e de monitoramento | Processo assinado pelo fornecedor do agente, no caminho de instalação, presente em muitos hosts | Exceção por caminho e assinatura |
| Componentes do próprio sistema | Processos do diretório do sistema, assinados pelo sistema operacional | Já cobertos pelo baseline |
| Relatório de erros do sistema | Dump gerado após falha registrada do LSASS, pelo mecanismo de relatório de erros | Confirmar a falha nos logs do sistema |
| Diagnóstico solicitado pelo suporte | Chamado registrado, executado por equipe autorizada | Confirmar com o ponto focal; registrar a exceção pontual |
| Exercícios de red team e pentest | Período e hosts previamente informados | Confirmar com o ponto focal |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): eventos de acesso, arquivo de dump (hash e local), processo responsável e lista de contas com sessão no host.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Acesso a credenciais confirmado é **Alta** no mínimo, e **Crítico** quando envolve servidores, controladores de domínio ou contas administrativas, ou quando há indício de uso das credenciais em outros hosts.
3. **Partir do princípio de que as credenciais do host foram comprometidas.** Recomendar ao cliente, conforme o modo de resposta contratado:
   - isolar o host;
   - redefinir as senhas de **todas** as contas que tinham sessão no host, priorizando as administrativas e de serviço;
   - se uma conta com privilégios no domínio foi exposta, avaliar a redefinição das credenciais de alto privilégio do domínio, com apoio da resposta a incidentes;
   - investigar onde as credenciais expostas foram usadas depois do acesso;
   - remover o arquivo de dump depois de preservá-lo como evidência.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| LSASS sem execução como processo protegido | Média | Habilitar a proteção do LSASS |
| Isolamento de credenciais do sistema desabilitado em hosts compatíveis | Média | Avaliar a habilitação |
| Armazenamento de senhas recuperáveis em memória habilitado (configuração legada) | Alta | Desabilitar a configuração |
| Administradores do domínio fazendo logon interativo em estações comuns | Média | Adotar modelo de camadas de administração |
| Telemetria sem eventos de acesso entre processos | Média | Ampliar a telemetria do endpoint |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Sim |
| **Ruído permite alerta contínuo?** | Dumps do LSASS, uso de `comsvcs.dll` para minidump e módulos de ferramentas de extração têm ruído muito baixo e devem virar alertas de alta prioridade. O acesso ao LSASS por processo fora do baseline pode virar alerta depois que as exceções do cliente estiverem estáveis. |
| **Tipos de plataforma alvo** | SIEM, EDR |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1003.001 OS Credential Dumping: LSASS Memory](https://attack.mitre.org/techniques/T1003/001/)
- MITRE ATT&CK — [T1218.011 Rundll32](https://attack.mitre.org/techniques/T1218/011/)
- MITRE ATT&CK — [S0002 Mimikatz](https://attack.mitre.org/software/S0002/)
- MITRE ATT&CK — [Credential Access (TA0006)](https://attack.mitre.org/tactics/TA0006/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
