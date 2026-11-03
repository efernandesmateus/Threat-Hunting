# TH-EXE-001 — Interpretadores com comandos codificados ou ofuscados

| Campo | Valor |
|-------|-------|
| **ID** | TH-EXE-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Execution (TA0002) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 3–4 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, execuções de interpretadores de comando e de script (PowerShell e cmd no Windows; shells no Linux e no macOS) com linhas de comando codificadas, ofuscadas ou com padrão de download e execução, priorizando as combinações raras de processo-pai e interpretador.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** interpretadores de comando já existem em todo sistema, são confiáveis e permitem fazer quase tudo. Por isso estão entre as técnicas de execução mais usadas por adversários, de operadores de ransomware a grupos de espionagem. Para dificultar a leitura por analistas e a detecção por assinaturas, os comandos costumam ser **codificados** (base64, por exemplo), **ofuscados** (concatenação de trechos, caracteres de escape, montagem a partir de variáveis de ambiente) ou estruturados para **baixar e executar** conteúdo direto na memória, sem gravar arquivos em disco.

A ofuscação tem um efeito colateral útil para o hunting: comandos ofuscados **não se parecem** com os comandos que administradores e softwares legítimos usam no dia a dia. Comprimento, aleatoriedade, parâmetros de codificação e processos-pai incomuns se destacam quando comparados com o restante do ambiente.

**Ameaças associadas:** o MITRE ATT&CK registra o uso de interpretadores de comando e de ofuscação de comandos por um grande número de grupos e famílias de malware. A estratégia de detecção do próprio ATT&CK para Command Obfuscation destaca escapes excessivos, codificação em base64, concatenação e linhas de comando com comprimento e entropia fora do padrão. Ver referências.

## 3. Hipótese

- **Dado que** adversários usam interpretadores de comando nativos e escondem o conteúdo dos comandos para evitar a detecção,
- **quando** executam comandos ou scripts ofuscados, codificados ou de download e execução,
- **então** aparecerão linhas de comando longas, de alta aleatoriedade, com parâmetros de codificação, padrões de ofuscação ou de download e execução, frequentemente com processos-pai que normalmente não iniciam interpretadores.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 |
| Execution | Command and Scripting Interpreter: Windows Command Shell | T1059.003 |
| Execution | Command and Scripting Interpreter: Unix Shell | T1059.004 |
| Stealth | Obfuscated Files or Information: Command Obfuscation | T1027.010 |
| Stealth | Deobfuscate/Decode Files or Information | T1140 |

A técnica principal é a **T1059**. A ofuscação (**T1027.010**) e a decodificação (**T1140**) passaram para a tática **Stealth** no ATT&CK v19 e descrevem como o comando é escondido.

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Telemetria de processos do endpoint (EDR) | Criação de processo com linha de comando completa, processo-pai, usuário, host | 30 dias | Sim, **ou** a fonte abaixo |
| Logs do sistema operacional com criação de processos | Windows: evento 4688 com linha de comando habilitada. Linux: auditoria de execuções (execve) | 30 dias | Sim, **ou** a fonte acima |
| Logs de execução de scripts | Windows: registro de blocos de script do PowerShell (evento 4104), que mostra o script já decodificado | 30 dias | Complementar |
| Conexões de rede do endpoint ou do proxy | Destinos acessados pelo processo do interpretador | 30 dias | Complementar |

**Escopo de ativos:** estações de trabalho e servidores Windows, Linux e macOS.

**Requisito crítico:** a **linha de comando completa**. Sem ela, este hunt é inviável. Em ambientes Windows, verifique se a inclusão da linha de comando no evento de criação de processo está habilitada e se o tamanho máximo registrado não trunca comandos longos.

## 6. Comportamento normal (baseline)

Uso legítimo de interpretadores é extremamente comum: scripts de logon, ferramentas de gestão e de inventário, implantação de software, agentes de monitoramento, tarefas agendadas de manutenção, rotinas de administradores e desenvolvedores. Alguns desses usos também empregam comandos codificados.

O que diferencia o uso malicioso costuma ser a **combinação**: um padrão de ofuscação ou codificação **e** uma origem incomum (processo-pai, usuário ou host que normalmente não fazem isso).

**Como levantar o baseline em um cliente novo:**

1. Em pelo menos 7 dias, faça o stacking das combinações **(processo-pai, interpretador)**, com a quantidade de hosts em que cada uma aparece.
2. Faça o mesmo para as linhas de comando com parâmetros de codificação, agrupando pelo início do comando ou pelo script de origem.
3. Identifique as ferramentas de gestão do cliente que geram comandos codificados e registre-as como exceções do cliente (fora deste repositório), preferencialmente pelo processo-pai e pela conta, e não pelo conteúdo do comando.
4. Registre, para referência, o comprimento típico das linhas de comando dos interpretadores no ambiente.

## 7. Estratégia de busca

**Técnicas de análise:** stacking, análise de comprimento e entropia, busca por padrões e enriquecimento.

**Lógica:**

1. Coletar as criações de processo dos interpretadores no período: no Windows, PowerShell (`powershell.exe`, `pwsh.exe`) e cmd (`cmd.exe`); no Linux e no macOS, os shells (`sh`, `bash`, `zsh`, `dash`, `ksh`) e interpretadores de script usados no ambiente.
2. Marcar os indicadores de cada execução:
   - **parâmetro de codificação:** no PowerShell, `-EncodedCommand` e suas abreviações (`-enc`, `-ec`, `-e`), seguidos de um bloco longo em base64;
   - **download e execução:** combinação de download (por exemplo, `DownloadString`, `Net.WebClient`, `Invoke-WebRequest`; ou `curl` e `wget`) com execução direta (`IEX`, `Invoke-Expression`; ou o conteúdo canalizado para um shell);
   - **decodificação seguida de execução:** por exemplo, `FromBase64String` no PowerShell ou `base64 -d` canalizado para um shell;
   - **ofuscação:** escape excessivo por circunflexo (`^`) no cmd, montagem de comandos a partir de substrings de variáveis de ambiente (`%VAR:~n,m%`), concatenação de trechos, alternância de maiúsculas e minúsculas, caracteres de traço tipográfico no lugar do hífen;
   - **comprimento e entropia:** linha de comando muito longa ou com aleatoriedade alta em relação ao baseline;
   - **origem incomum:** combinação (processo-pai, interpretador) rara no ambiente, especialmente quando o pai é um aplicativo de escritório, navegador, leitor de PDF, cliente de e-mail ou serviço web.
3. Decodificar os blocos em base64 encontrados e analisar o conteúdo real do comando.
4. Priorizar as execuções com mais de um indicador e as de origem incomum.
5. Enriquecer com o usuário, o host, a reputação dos destinos acessados e a atividade seguinte do mesmo processo.

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Bloco em base64 após parâmetro de codificação | ≥ 40 caracteres |
| Comprimento da linha de comando | ≥ 1.000 caracteres, ou acima do percentil 99 do ambiente |
| Entropia da linha de comando | ≥ 5,0 bits por caractere |
| Escape por circunflexo no cmd | ≥ 6 ocorrências do padrão `^` seguido de caractere |
| Combinação (processo-pai, interpretador) rara | Presente em ≤ 3 hosts nos últimos 30 dias |

**Prioridade:** download e execução, ou decodificação seguida de execução, já justificam a triagem mesmo isolados. Os demais indicadores pedem combinação.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Criação de processos com linha de comando (EDR ou logs do sistema operacional)
            Últimos 30 dias

FILTRO:     imagem do processo em interpretadores:
              Windows: powershell.exe, pwsh.exe, cmd.exe
              Linux/macOS: sh, bash, zsh, dash, ksh
            Excluir combinações (processo_pai, conta) da lista de exceções do cliente

INDICADORES, por execução:
            codificado     = PowerShell com -e/-ec/-enc/-EncodedCommand (e abreviações)
                             seguido de bloco base64 com >= 40 caracteres
            download_exec  = (download: DownloadString, DownloadData, Net.WebClient,
                              Invoke-WebRequest, Invoke-RestMethod, curl, wget)
                             E (execução: IEX, Invoke-Expression, "| sh", "| bash")
            decodifica_exec = FromBase64String ou "base64 -d" seguido de execução
            ofuscado       = ^ seguido de caractere, >= 6 vezes (cmd)
                             OU %VAR:~n,m% (cmd)
                             OU traço tipográfico (– ou —) usado como parâmetro
            longo          = comprimento(linha_comando) >= 1000 ou > percentil 99 do ambiente
            alta_entropia  = entropia(linha_comando) >= 5,0
            pai_raro       = (processo_pai, interpretador) presente em <= 3 hosts em 30 dias

DESTAQUE:   download_exec OU decodifica_exec
            OU (codificado OU ofuscado OU longo OU alta_entropia) E pai_raro
            OU contagem de indicadores >= 3
            (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      host, usuário, data_hora, processo_pai, interpretador, linha_comando,
            conteúdo decodificado (se houver), indicadores presentes,
            hosts com a mesma combinação (processo_pai, interpretador)
            Ordenar por quantidade de indicadores e raridade
```

**Observações:**

- Decodifique sempre os blocos em base64. No PowerShell, o conteúdo de `-EncodedCommand` é texto em UTF-16LE codificado em base64.
- Se houver o registro de blocos de script (evento 4104), use-o: ele mostra o script depois da decodificação feita pelo próprio PowerShell.
- O cálculo de entropia pode não estar disponível em todas as plataformas; sem ele, use o comprimento e a proporção de caracteres não alfanuméricos.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-exe-001-interpretadores-comandos-ofuscados.yml`](th-exe-001-interpretadores-comandos-ofuscados.yml)

O arquivo contém cinco regras, com sintaxe verificada por um analisador Sigma de código aberto e expressões regulares testadas com exemplos positivos e negativos:

| Regra | Sistema | Nível |
|-------|---------|-------|
| PowerShell com comando codificado | Windows | medium |
| PowerShell com download e execução em memória | Windows | high |
| Ofuscação na linha de comando do cmd | Windows | medium |
| Conteúdo decodificado de base64 executado em shell | Linux | medium |
| Download canalizado direto para shell | Linux | medium |

Limitações da versão Sigma em relação à pseudoconsulta:

- Raridade de processo-pai, comprimento relativo ao baseline e entropia não são expressos nas regras; essa parte é feita na execução do hunt.
- As regras cobrem os padrões mais comuns. Ofuscações mais elaboradas (concatenação de trechos, alternância de maiúsculas) dependem da análise de entropia e da triagem.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Dados de teste (somente em laboratório ou com autorização formal):**

1. Em um host de teste, executar um comando inofensivo (por exemplo, exibir a data) no PowerShell com `-EncodedCommand`, a partir de um processo-pai incomum.
2. Executar o mesmo comando no cmd com escape por circunflexo e com substrings de variáveis de ambiente.
3. Em um host Linux de teste, decodificar com `base64 -d` um comando inofensivo e canalizá-lo para `bash`; repetir com `curl` baixando um script inofensivo de um servidor de teste e canalizando para o shell.
4. Confirmar que cada execução aparece nos resultados e que o conteúdo decodificado corresponde ao comando original.

Alternativa sem laboratório: usar datasets públicos de telemetria gravados durante simulações de técnicas de execução.

**Validação histórica:** executar o hunt em pelo menos 30 dias de dados reais, com autorização, para medir o ruído e montar a lista de exceções do ambiente.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | O comando decodificado é legítimo e o processo-pai e a conta pertencem a uma ferramenta de gestão, implantação ou monitoramento conhecida; o padrão se repete de forma consistente em muitos hosts. Incluir nas exceções do cliente. |
| **Suspeito — investigar** | O conteúdo decodificado não é claro; a combinação de processo-pai e interpretador é rara, mas tem explicação possível; o destino de download é desconhecido. |
| **Malicioso — escalar** | O conteúdo decodificado baixa ou executa código de origem desconhecida, desativa proteções, coleta credenciais ou estabelece conexão de retorno; o processo-pai é um aplicativo de escritório, navegador ou serviço web; o destino tem reputação ruim; há atividade suspeita logo em seguida no mesmo host. |

**Perguntas de enriquecimento:**

- O que o comando faz depois de decodificado?
- Qual é o processo-pai? Ele costuma iniciar interpretadores neste ambiente? E o avô?
- A conta é de usuário comum, administrativa ou de serviço? É compatível com a ação?
- Houve conexão de rede logo após a execução? Para onde? Qual a reputação do destino?
- Algum arquivo foi gravado, alguma tarefa ou serviço foi criado em seguida? (ver [TH-PER-001](../persistence/th-per-001-tarefas-servicos-fora-do-padrao.md))
- O mesmo comando ou padrão aparece em outros hosts?

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Ferramentas de gestão e de implantação | Processo-pai e conta da ferramenta; padrão repetido em muitos hosts; conteúdo decodificado legítimo | Exceção por processo-pai e conta |
| Agentes de monitoramento e de inventário | Execução periódica, mesma linha de comando em muitos hosts | Exceção por processo-pai e conta |
| Scripts de logon e de manutenção | Execução no logon ou em horário fixo; origem em compartilhamento de scripts do cliente | Exceção por caminho do script |
| Instaladores de software no formato "baixar e executar" | Destino de download oficial do software; uso por administradores | Verificar a origem; registrar exceção específica |
| Desenvolvedores e administradores | Uso interativo por contas conhecidas, em hosts de administração | Avaliar caso a caso; não criar exceção ampla |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): linha de comando completa, conteúdo decodificado, árvore de processos, conexões e arquivos gerados.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Execução maliciosa confirmada é **Alta**, e **Crítica** se houver conexão de retorno ativa, movimentação lateral ou envolvimento de ativos críticos.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - isolar o host;
   - bloquear os destinos de download e de conexão identificados;
   - identificar e remover persistências criadas pelo comando;
   - investigar como o comando chegou ao host (e-mail, navegador, acesso remoto).
4. Usar os indicadores encontrados (destinos, nomes de arquivo, trechos do comando) para buscar a mesma atividade em outros hosts do cliente.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Criação de processos sem linha de comando registrada | Média | Habilitar a linha de comando nos eventos de criação de processo |
| Registro de blocos de script do PowerShell desabilitado | Baixa | Habilitar o registro de blocos de script |
| Versões antigas do PowerShell disponíveis, que não registram scripts | Média | Remover ou bloquear versões antigas |
| Política de execução de scripts permissiva em estações | Baixa | Revisar a política de execução |
| Ferramentas legítimas que dependem de comandos codificados | Baixa | Recomendar alternativas que não usem codificação, para reduzir ruído |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Sim, para parte dos indicadores |
| **Ruído permite alerta contínuo?** | Download e execução em memória e decodificação seguida de execução costumam ter ruído baixo depois das exceções do cliente e são bons candidatos a alerta. Comando codificado e ofuscação no cmd dependem das exceções de ferramentas de gestão. Raridade de processo-pai e entropia continuam no hunt manual. |
| **Tipos de plataforma alvo** | SIEM, EDR |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1059 Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059/)
- MITRE ATT&CK — [T1059.001 PowerShell](https://attack.mitre.org/techniques/T1059/001/)
- MITRE ATT&CK — [T1059.003 Windows Command Shell](https://attack.mitre.org/techniques/T1059/003/)
- MITRE ATT&CK — [T1059.004 Unix Shell](https://attack.mitre.org/techniques/T1059/004/)
- MITRE ATT&CK — [T1027.010 Command Obfuscation](https://attack.mitre.org/techniques/T1027/010/)
- MITRE ATT&CK — [T1140 Deobfuscate/Decode Files or Information](https://attack.mitre.org/techniques/T1140/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
