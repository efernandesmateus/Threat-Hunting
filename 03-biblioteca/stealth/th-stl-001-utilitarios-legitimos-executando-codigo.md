# TH-STL-001 — Utilitários legítimos do sistema usados para executar código

| Campo | Valor |
|-------|-------|
| **ID** | TH-STL-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Stealth (TA0005) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 2–3 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, execuções de utilitários nativos e assinados do Windows (como `rundll32`, `regsvr32`, `mshta`, `msiexec` e `certutil`) com argumentos, origens ou comportamentos incomuns, que indiquem o uso desses binários para executar ou buscar código malicioso.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** o Windows traz dezenas de utilitários assinados pelo próprio sistema que, além da função original, conseguem carregar DLLs, executar scripts ou baixar arquivos. Como são binários confiáveis, presentes em todo host e muitas vezes permitidos por controles de aplicação, adversários os usam para **executar código por procuração**: em vez de rodar o próprio malware, pedem a um utilitário legítimo que o faça. Esse conjunto de binários é conhecido na comunidade como LOLBins (living off the land binaries).

O uso legítimo desses utilitários é estável e previsível: os mesmos argumentos, os mesmos processos-pai, os mesmos arquivos. O uso malicioso costuma quebrar esse padrão: conteúdo remoto, scripts inline, DLLs em diretórios temporários ou processos-pai que nunca iniciam esses binários. A própria estratégia de detecção do ATT&CK para a técnica recomenda comparar as execuções com o histórico de argumentos e arquivos conhecidos.

**Ameaças associadas:** o MITRE ATT&CK registra o uso de proxy de execução por binários do sistema por diversos grupos, incluindo o Volt Typhoon, conhecido por operar quase exclusivamente com ferramentas nativas. Ver referências.

**Escopo:** este hunt cobre o Windows. O ATT&CK também descreve o abuso de binários confiáveis em Linux e macOS; esses casos estão no backlog como hunt futuro.

## 3. Hipótese

- **Dado que** adversários usam binários nativos e assinados do sistema para executar código sem levantar suspeita,
- **quando** eles usam esses utilitários para carregar, executar ou baixar conteúdo malicioso,
- **então** aparecerão execuções desses utilitários com argumentos incomuns (conteúdo remoto, scripts inline, arquivos em diretórios graváveis), processos-pai atípicos ou conexões de rede que eles normalmente não fazem.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Stealth | System Binary Proxy Execution: Rundll32 | T1218.011 |
| Stealth | System Binary Proxy Execution: Regsvr32 | T1218.010 |
| Stealth | System Binary Proxy Execution: Mshta | T1218.005 |
| Stealth | System Binary Proxy Execution: Msiexec | T1218.007 |
| Stealth | System Script Proxy Execution | T1216 |
| Stealth | Deobfuscate/Decode Files or Information | T1140 |
| Command and Control | Ingress Tool Transfer | T1105 |

A técnica principal é a **T1218**, que passou para a tática **Stealth** no ATT&CK v19. O `certutil` não é um proxy de execução, mas é um dos utilitários nativos mais usados para **baixar** (T1105) e **decodificar** (T1140) arquivos, e por isso está neste hunt.

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Telemetria de processos do endpoint (EDR) | Criação de processo com linha de comando completa, nome original do arquivo, processo-pai, usuário; carregamento de módulos (DLLs) | 30 dias | Sim |
| Conexões de rede do endpoint | Conexões iniciadas pelos utilitários (processo, destino, porta) | 30 dias | Complementar |
| Logs do sistema operacional com criação de processos | Evento 4688 com linha de comando, como alternativa parcial (sem módulos nem conexões) | 30 dias | Complementar |

**Escopo de ativos:** estações de trabalho e servidores Windows.

**Requisito crítico:** o **nome original do arquivo** (gravado no cabeçalho do executável). Adversários renomeiam esses utilitários para escapar de buscas pelo nome; o nome original denuncia a cópia renomeada.

## 6. Comportamento normal (baseline)

Esses utilitários são usados o tempo todo pelo próprio sistema e por softwares legítimos: o `rundll32` executa funções do painel de controle e de componentes do sistema, o `msiexec` instala e atualiza programas, o `regsvr32` registra componentes durante instalações. O uso legítimo é **repetitivo**: os mesmos pares de processo-pai e argumentos aparecem em muitos hosts.

**Como levantar o baseline em um cliente novo:**

1. Em pelo menos 7 dias, faça o stacking de cada utilitário por **(processo-pai, padrão de argumentos)**, normalizando partes variáveis (GUIDs, números de versão, caminhos de usuário).
2. Os pares presentes em muitos hosts e ligados a softwares conhecidos formam o baseline.
3. Identifique as ferramentas de implantação que usam `msiexec` com pacotes servidos por HTTP interno e registre-as como exceções do cliente, fora deste repositório.
4. Registre se os utilitários fazem conexões de rede no uso normal. A maioria deles quase nunca faz.

## 7. Estratégia de busca

**Técnicas de análise:** stacking, busca por padrões e enriquecimento.

**Lógica:**

1. Coletar as execuções dos utilitários no período, identificando-os pelo **nome original do arquivo**, e não apenas pelo nome do executável.
2. Marcar os indicadores:
   - **conteúdo remoto:** URL na linha de comando (`http://`, `https://`), ou UNC para hosts externos;
   - **script inline:** `javascript:` ou `vbscript:` na linha de comando (`rundll32`, `mshta`);
   - **componente de scriptlet:** `regsvr32` com `/i:` seguido de URL, ou carregando `scrobj.dll`;
   - **arquivo em diretório gravável:** DLL, scriptlet ou pacote carregado de `\AppData\`, `\Users\Public\`, `\Temp\`, `\ProgramData\` ou `\Downloads\`;
   - **download ou decodificação com certutil:** parâmetros `urlcache`, `verifyctl` ou `decode`;
   - **utilitário renomeado:** nome original do arquivo diferente do nome do executável, ou executável fora do diretório do sistema;
   - **processo-pai atípico:** par (processo-pai, utilitário) fora do baseline, especialmente quando o pai é um aplicativo de escritório, navegador, cliente de e-mail, interpretador de script ou serviço web;
   - **rede inesperada:** o utilitário inicia conexão para destino externo;
   - **rundll32 sem argumentos:** execução sem DLL ou função indicada, comum quando o processo serve de casca para código injetado.
3. Priorizar as execuções com pelo menos dois indicadores e as que se encaixam nas prioridades abaixo.
4. Enriquecer com o arquivo carregado (hash, assinatura, reputação), o destino de rede e a atividade seguinte no host.

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Par (processo-pai, padrão de argumentos) raro | Presente em ≤ 3 hosts nos últimos 30 dias |
| Conexão de rede por utilitário | Qualquer conexão para destino externo, quando o baseline não mostra conexões |

**Prioridade:** conteúdo remoto, script inline, componente de scriptlet, `certutil` baixando arquivos e utilitário renomeado já justificam a triagem mesmo isolados.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Criação de processos (EDR), com nome original do arquivo
            Últimos 30 dias

FILTRO:     nome_original em (RUNDLL32.EXE, REGSVR32.EXE, MSHTA.EXE, msiexec.exe,
                              CertUtil.exe)
            Excluir pares (processo_pai, padrão_argumentos) da lista de exceções do cliente

NORMALIZAR: padrão_argumentos = linha_comando sem GUIDs, versões e nomes de usuário

INDICADORES, por execução:
            remoto        = linha_comando contém http:// ou https://
            inline        = linha_comando contém javascript: ou vbscript:
            scriptlet     = REGSVR32 E (/i: com URL OU scrobj.dll)
            dir_gravavel  = arquivo carregado em \AppData\, \Users\Public\, \Temp\,
                            \ProgramData\, \Downloads\
            certutil_baixa = CertUtil E (urlcache OU verifyctl OU decode)
            renomeado     = nome do executável <> nome_original
                            OU executável fora de \Windows\System32\ e \Windows\SysWOW64\
            pai_raro      = (processo_pai, padrão_argumentos) em <= 3 hosts em 30 dias
            rede          = conexão para destino externo pelo processo (se disponível)
            sem_args      = RUNDLL32 sem argumentos

DESTAQUE:   remoto OU inline OU scriptlet OU certutil_baixa OU renomeado
            OU contagem de indicadores >= 2
            (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      host, usuário, data_hora, processo_pai, utilitário, nome_original,
            linha_comando, arquivo carregado (hash, assinatura), destino de rede,
            indicadores presentes, hosts com o mesmo par
            Ordenar por quantidade de indicadores e raridade
```

**Observações:**

- A lista de utilitários é um ponto de partida. O ATT&CK descreve outros binários com o mesmo uso (por exemplo, os da T1218 e da T1216); acrescente os que fizerem sentido para o ambiente.
- Sem o nome original do arquivo, a detecção de utilitários renomeados fica impossível; registre como lacuna.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-stl-001-utilitarios-legitimos-executando-codigo.yml`](th-stl-001-utilitarios-legitimos-executando-codigo.yml)

O arquivo contém cinco regras, com sintaxe verificada por um analisador Sigma de código aberto:

| Regra | Nível |
|-------|-------|
| Rundll32 com conteúdo remoto, script inline ou DLL em diretório gravável | medium |
| Regsvr32 carregando scriptlet remoto ou componente de script | high |
| Mshta executando conteúdo remoto ou script inline | high |
| Msiexec instalando pacote remoto | medium |
| Certutil usado para baixar ou decodificar arquivos | high |

Todas as regras identificam o utilitário pelo nome do executável **ou** pelo nome original do arquivo, o que cobre cópias renomeadas.

Limitações da versão Sigma em relação à pseudoconsulta:

- Raridade do par (processo-pai, argumentos), conexões de rede e assinatura do arquivo carregado não são expressas nas regras; essa parte é feita na execução do hunt.
- A detecção de utilitário renomeado fora do diretório do sistema depende da comparação entre caminho e nome original, feita na pseudoconsulta.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Telemetria esperada:** para cada utilitário, a validação deve mostrar, nos dados, a criação do processo com a linha de comando completa e o nome original do arquivo, o processo-pai, e, quando houver, o carregamento do módulo e a conexão de rede.

**Como gerar os dados (somente em laboratório ou com autorização formal):** use ferramentas de emulação de adversário que tenham testes mapeados às subtécnicas da T1218, executando conteúdo inofensivo. Confira que cada teste aparece nos resultados com os indicadores esperados, inclusive com uma cópia renomeada de um dos utilitários.

**Alternativa sem laboratório:** usar datasets públicos de telemetria gravados durante simulações de proxy de execução.

**Validação histórica:** executar o hunt em pelo menos 30 dias de dados reais, com autorização, para medir o ruído e montar a lista de exceções do ambiente.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | O par (processo-pai, argumentos) pertence a software ou ferramenta de implantação conhecida; o arquivo carregado é assinado por entidade confiável; o padrão se repete em vários hosts. Incluir nas exceções do cliente quando recorrente. |
| **Suspeito — investigar** | Par raro com explicação possível; DLL em diretório gravável, mas assinada; pacote remoto de servidor interno desconhecido. |
| **Malicioso — escalar** | Conteúdo remoto de destino externo desconhecido ou de reputação ruim; script inline; componente de scriptlet; `certutil` baixando arquivos de fora; utilitário renomeado; processo-pai que nunca inicia o utilitário (aplicativo de escritório, navegador); arquivo carregado não assinado e desconhecido. |

**Perguntas de enriquecimento:**

- Qual arquivo ou conteúdo o utilitário carregou ou baixou? Ele é assinado? Qual a reputação?
- Qual é o processo-pai? Ele costuma iniciar esse utilitário neste ambiente?
- O utilitário fez conexão de rede? Para onde?
- O que aconteceu antes no host? Houve comando ofuscado (ver [TH-EXE-001](../execution/th-exe-001-interpretadores-comandos-ofuscados.md)) ou abertura de anexo?
- O que aconteceu depois? Houve criação de tarefas ou serviços (ver [TH-PER-001](../persistence/th-per-001-tarefas-servicos-fora-do-padrao.md))?
- O mesmo padrão aparece em outros hosts?

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Componentes do sistema e do painel de controle via `rundll32` | DLLs do diretório do sistema; pares comuns em muitos hosts | Já cobertos pelo baseline |
| Instalação e atualização de software via `msiexec` | Pacotes de servidores internos de implantação; ferramenta de implantação como pai | Exceção por servidor e processo-pai |
| Software instalado por usuário em `AppData` | DLL assinada pelo fabricante do software; presente em vários hosts | Exceção por caminho e assinatura |
| Aplicações legadas em HTA | `mshta` executando arquivo local de aplicação conhecida | Exceção específica; recomendar a substituição |
| Uso administrativo do `certutil` para certificados | Parâmetros de manipulação de certificados, sem download | Já excluídos pelos indicadores |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): linha de comando, árvore de processos, arquivo carregado ou baixado (hash e, se possível, cópia) e conexões.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Execução maliciosa confirmada por proxy é **Alta**, e **Crítica** se houver conexão de retorno ativa ou envolvimento de ativos críticos.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - isolar o host;
   - bloquear os destinos remotos e o hash do arquivo carregado;
   - procurar o mesmo padrão e o mesmo arquivo nos demais hosts;
   - avaliar restringir, por controle de aplicação, os utilitários que o ambiente não usa.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Utilitários raramente necessários (`mshta`, por exemplo) liberados para todos os usuários | Baixa | Restringir por controle de aplicação |
| Telemetria sem nome original do arquivo ou sem carregamento de módulos | Média | Ampliar a telemetria do endpoint |
| Aplicações legadas que dependem de HTA ou de scriptlets | Baixa | Planejar a substituição |
| Estações sem controle de aplicação | Média | Avaliar política de controle de aplicação |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Sim, para a maior parte dos indicadores |
| **Ruído permite alerta contínuo?** | Regsvr32 com scriptlet, mshta com conteúdo remoto ou inline, certutil baixando arquivos e utilitário renomeado costumam ter ruído baixo e são bons candidatos a alerta. Rundll32 e msiexec dependem das exceções do cliente. A raridade de processo-pai continua no hunt manual. |
| **Tipos de plataforma alvo** | SIEM, EDR |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1218 System Binary Proxy Execution](https://attack.mitre.org/techniques/T1218/)
- MITRE ATT&CK — [T1218.011 Rundll32](https://attack.mitre.org/techniques/T1218/011/)
- MITRE ATT&CK — [T1218.010 Regsvr32](https://attack.mitre.org/techniques/T1218/010/)
- MITRE ATT&CK — [T1218.005 Mshta](https://attack.mitre.org/techniques/T1218/005/)
- MITRE ATT&CK — [T1218.007 Msiexec](https://attack.mitre.org/techniques/T1218/007/)
- MITRE ATT&CK — [T1216 System Script Proxy Execution](https://attack.mitre.org/techniques/T1216/)
- MITRE ATT&CK — [T1140 Deobfuscate/Decode Files or Information](https://attack.mitre.org/techniques/T1140/)
- MITRE ATT&CK — [T1105 Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
