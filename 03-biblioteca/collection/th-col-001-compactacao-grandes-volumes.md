# TH-COL-001 — Compactação de grandes volumes de arquivos antes da saída

| Campo | Valor |
|-------|-------|
| **ID** | TH-COL-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Collection (TA0009) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 2–3 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, a preparação de dados para exfiltração: compactação de grandes volumes de arquivos, compactação com senha ou criptografia, divisão em volumes e criação de arquivos compactados em diretórios usados como área de preparação, em Windows e Linux.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** antes de levar os dados embora, o adversário costuma **reunir e empacotar**: copia os arquivos de interesse para um diretório de preparação, compacta tudo, geralmente **com senha** (para que o conteúdo não seja inspecionado na saída) e às vezes **dividido em volumes** (para que cada parte passe despercebida ou caiba nos limites do canal de saída). Em ataques de ransomware com dupla extorsão, essa etapa acontece pouco antes da criptografia.

Este hunt detecta o **comportamento**, e não a ferramenta: o adversário pode trocar de compactador ou renomeá-lo, mas continua precisando adicionar arquivos a um pacote, protegê-lo com senha, dividi-lo e gravá-lo em algum lugar. Por isso as regras e a pseudoconsulta buscam os **parâmetros** e os **resultados** da compactação, sem depender do nome do executável.

É o elo entre o acesso aos dados e a exfiltração (ver [TH-EXF-001](../exfiltration/th-exf-001-exfiltracao-via-dns.md)).

**Ameaças associadas:** o MITRE ATT&CK registra a compactação de dados coletados com utilitários e a preparação local de dados por um grande número de grupos e operadores de ransomware. Ver referências.

## 3. Hipótese

- **Dado que** adversários reúnem e compactam os dados antes de exfiltrá-los,
- **quando** eles preparam os dados para a saída,
- **então** aparecerão execuções de compactação com parâmetros de senha, de criptografia ou de divisão em volumes, compactação de muitos arquivos ou de grande volume a partir de hosts e contas incomuns, e arquivos compactados grandes em diretórios de preparação.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Collection | Archive Collected Data: Archive via Utility | T1560.001 |
| Collection | Data Staged: Local Data Staging | T1074.001 |
| Collection | Data Staged: Remote Data Staging | T1074.002 |
| Exfiltration | Data Transfer Size Limits (divisão em volumes) | T1030 |

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Telemetria do endpoint (EDR) | Criação de processos com linha de comando; criação de arquivos com caminho, tamanho e processo criador | 30 dias | Sim |
| Acesso a compartilhamentos de arquivos | Leitura em massa de arquivos em servidores de arquivos (quantidade de arquivos e volume por conta) | 30 dias | Complementar |
| Conexões de rede e proxy | Volume de saída do host após a compactação | 30 dias | Complementar |

**Escopo de ativos:** estações de trabalho e servidores Windows e Linux, com prioridade para servidores de arquivos e hosts com acesso a dados sensíveis.

**Requisito crítico:** o **tamanho** dos arquivos criados. Sem ele, o hunt detecta os parâmetros suspeitos, mas não consegue avaliar o volume, que é um dos indicadores mais fortes.

## 6. Comportamento normal (baseline)

Compactação é muito comum: rotinas de backup, coleta de logs para suporte, envio de arquivos por e-mail, desenvolvedores empacotando projetos, instaladores. Algumas rotinas de backup também usam senha e divisão em volumes.

O que diferencia a preparação de exfiltração costuma ser a **combinação**: volume ou quantidade de arquivos muito acima do habitual **e** origem incomum (host, conta ou diretório), muitas vezes com senha, em horário atípico e seguida de saída de dados.

**Como levantar o baseline em um cliente novo:**

1. Em pelo menos 14 dias, faça o stacking das execuções de compactação por **(host, conta, processo-pai)**, com volume e quantidade de arquivos.
2. Identifique as rotinas de backup e de coleta de logs, com seus horários, contas e destinos, e registre-as como exceções do cliente, fora deste repositório.
3. Registre o tamanho típico dos arquivos compactados criados por host e por conta.

## 7. Estratégia de busca

**Técnicas de análise:** stacking, busca por padrões, análise temporal e enriquecimento.

**Lógica:**

1. Coletar as execuções de compactação no período, identificadas pelo **comportamento**: linha de comando com comando de adicionar a um pacote e extensão de arquivo compactado (`.7z`, `.rar`, `.zip`, `.tar`, `.gz`), uso do recurso nativo de compactação do PowerShell (`Compress-Archive`) ou de `tar` e `zip` no Linux.
2. Coletar a criação de arquivos compactados, com tamanho e processo criador.
3. Marcar os indicadores:
   - **senha ou criptografia:** parâmetros de senha (por exemplo, `-p<senha>`, `-hp<senha>`, `-P <senha>`, `-e`), criptografia dos nomes de arquivos (`-mhe=on`) ou canalização para ferramentas de criptografia (`openssl`, `gpg`);
   - **divisão em volumes:** parâmetro de tamanho de volume (por exemplo, `-v500m`) ou vários arquivos sequenciais (`.part1`, `.001`, `.z01`) criados juntos;
   - **volume alto:** arquivo compactado muito maior que o baseline do host, ou soma dos volumes;
   - **muitos arquivos:** compactação recursiva de diretórios inteiros, compartilhamentos de rede ou perfis de usuários;
   - **diretório de preparação:** arquivo criado em `\Users\Public\`, `\ProgramData\`, `\PerfLogs\`, `\$Recycle.Bin\`, `\Windows\Temp\` ou, no Linux, `/tmp/`, `/var/tmp/`, `/dev/shm/`;
   - **origem incomum:** (host, conta, processo-pai) fora do baseline, especialmente servidores de arquivos e contas que normalmente não compactam;
   - **horário atípico:** fora do horário de trabalho e das janelas de backup;
   - **saída em seguida:** aumento do volume de saída do host nas horas seguintes, ou o arquivo compactado aberto por processo de transferência.
4. Priorizar as execuções com mais de um indicador e as seguidas de saída de dados.
5. Enriquecer com os arquivos de origem (o que foi compactado) e com a atividade anterior no host.

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Volume alto | Arquivo compactado ≥ 1 GB, ou acima de 10 vezes o maior arquivo do baseline do host |
| Volumes sequenciais | ≥ 3 partes criadas em 1 hora |
| Compactação recursiva de compartilhamento ou perfis | Qualquer ocorrência fora de exceção |
| Saída de dados após a compactação | Volume de saída do host nas 24 h seguintes ≥ 50% do tamanho do arquivo compactado |

**Prioridade:** compactação com senha **e** volume alto, divisão em volumes fora de rotina de backup e compactação seguida de saída de dados já justificam a triagem.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Criação de processos e de arquivos (EDR), últimos 30 dias
            Complementar: conexões de rede / proxy, acesso a compartilhamentos

FILTRO:     Excluir rotinas de backup e coleta de logs da lista de exceções do cliente
            (por host, conta, processo e horário)

IDENTIFICAÇÃO (por comportamento, sem depender do nome do executável):
            compactacao = linha_comando com comando de adicionar (" a " ou " u ")
                          e extensão .7z, .rar, .zip, .tar, .gz
                          OU Compress-Archive
                          OU (Linux) tar com criação, zip, gzip

INDICADORES, por execução e arquivo criado:
            senha        = -p<senha>, -hp<senha>, -mhe=on, zip -P / -e,
                           canalização para openssl ou gpg
            volumes      = -v<tamanho> OU >= 3 partes sequenciais criadas em 1 h
            volume_alto  = tamanho >= 1 GB OU > 10 × maior arquivo do baseline do host
            muitos_arqs  = compactação recursiva de compartilhamento ou de perfis
            preparacao   = arquivo em \Users\Public\, \ProgramData\, \PerfLogs\,
                           \$Recycle.Bin\, \Windows\Temp\, /tmp/, /var/tmp/, /dev/shm/
            origem_rara  = (host, conta, processo_pai) fora do baseline
            fora_horario = fora do horário de trabalho e das janelas de backup
            saida        = volume de saída do host nas 24 h seguintes
                           >= 50% do tamanho do arquivo

DESTAQUE:   (senha E volume_alto) OU volumes OU (volume_alto E saida)
            OU contagem de indicadores >= 3
            (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      host, conta, data_hora, processo_pai, linha_comando, arquivo criado,
            tamanho, diretório, indicadores presentes, volume de saída posterior
            Ordenar por volume e quantidade de indicadores
```

**Observações:**

- Os parâmetros de senha variam entre compactadores; os exemplos acima cobrem os formatos mais comuns. Ajuste a lista aos utilitários presentes no ambiente.
- A senha costuma aparecer em texto claro na linha de comando. Ela pode ser útil para a resposta a incidentes (abrir o pacote e saber o que foi levado); registre-a nas evidências.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-col-001-compactacao-grandes-volumes.yml`](th-col-001-compactacao-grandes-volumes.yml)

O arquivo contém cinco regras, com sintaxe verificada por um analisador Sigma de código aberto e expressões regulares testadas com exemplos positivos e negativos. Todas detectam o **comportamento**, sem depender do nome do compactador:

| Regra | Sistema | Nível |
|-------|---------|-------|
| Compactação com senha ou criptografia de cabeçalhos | Windows | medium |
| Compactação com divisão em volumes | Windows | medium |
| Arquivo compactado criado em diretório de preparação | Windows | low |
| Compactação com o recurso nativo do PowerShell em diretório de preparação | Windows | medium |
| Compactação com senha ou criptografia | Linux | medium |

Limitações da versão Sigma em relação à pseudoconsulta:

- **Tamanho** dos arquivos, quantidade de arquivos compactados e **saída de dados** posterior não são expressos nas regras; são parte central do hunt.
- Detectar pelo comportamento reduz a dependência do nome da ferramenta, mas compactadores com sintaxe muito diferente podem exigir ajuste dos padrões.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Telemetria esperada:** a criação do processo de compactação com a linha de comando completa e a criação do arquivo compactado com caminho e tamanho; para a correlação com a saída, o volume de rede do host nas horas seguintes.

**Como gerar os dados (somente em laboratório ou com autorização formal):** em um host de teste, criar um diretório com arquivos fictícios (sem dados reais), compactá-lo com senha e com divisão em volumes, gravando o resultado em um dos diretórios de preparação. Repetir no Linux com `zip -P` e com `tar` canalizado para criptografia. Confirme que as execuções e os arquivos aparecem destacados e que as rotinas de backup do baseline não aparecem.

**Alternativa sem laboratório:** usar datasets públicos de telemetria gravados durante simulações de coleta e preparação de dados.

**Validação histórica:** executar o hunt em pelo menos 30 dias de dados reais, com autorização, para calibrar os limiares de volume e montar a lista de exceções de backup.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | Rotina de backup ou de coleta de logs identificada; usuário compactando os próprios arquivos de trabalho em volume compatível; desenvolvedor empacotando projeto. |
| **Suspeito — investigar** | Volume alto sem senha a partir de conta comum; compactação com senha de volume pequeno; compactação fora do horário por conta administrativa conhecida. |
| **Malicioso — escalar** | Compactação com senha de grande volume ou de compartilhamentos e perfis inteiros por conta ou host incomum; divisão em volumes fora de rotina; arquivo em diretório de preparação seguido de saída de dados; compactação precedida de acesso a credenciais, movimentação lateral ou enumeração. |

**Perguntas de enriquecimento:**

- O que foi compactado? De onde vieram os arquivos? Há dados sensíveis?
- A conta e o host costumam compactar volumes assim? Há rotina de backup que explique?
- Houve saída de dados do host depois da compactação? Para onde? (ver [TH-EXF-001](../exfiltration/th-exf-001-exfiltracao-via-dns.md))
- O que aconteceu antes no host? Houve movimentação lateral ou enumeração? (ver [TH-LAT-001](../lateral-movement/th-lat-001-movimentacao-lateral-contas-administrativas.md) e [TH-DIS-001](../discovery/th-dis-001-enumeracao-diretorio-em-rajada.md))
- O arquivo compactado ainda existe? Foi apagado depois da saída?

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Rotinas de backup | Horário fixo, conta de serviço de backup, destino conhecido | Exceção por conta, horário e destino |
| Coleta de logs para suporte | Pacotes de logs em diretórios temporários, por ferramenta de suporte | Exceção por processo |
| Usuários enviando arquivos | Volume pequeno ou moderado, arquivos do próprio usuário | Avaliar volume e destino |
| Desenvolvimento e build | Empacotamento em diretórios de projeto, por ferramentas de build | Exceção por host de desenvolvimento e processo |
| Instaladores e atualizadores | Pacotes em `\ProgramData\` criados por instaladores conhecidos | Exceção por processo e assinatura |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): linha de comando (incluindo a senha, se visível), lista e tamanho dos arquivos criados, arquivos de origem e conexões de saída. Se possível, preservar o próprio arquivo compactado.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Preparação de exfiltração confirmada é **Alta**, e **Crítica** se houver saída de dados, dados sensíveis envolvidos ou indício de ransomware em preparação.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - isolar o host para interromper uma possível saída em andamento;
   - bloquear os destinos de saída identificados;
   - identificar com precisão **quais dados** foram compactados, para apoiar a avaliação de impacto e as obrigações legais (por exemplo, comunicação de incidente com dados pessoais);
   - procurar arquivos compactados semelhantes em outros hosts.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Contas com acesso de leitura amplo a compartilhamentos sem necessidade | Média | Revisar permissões de compartilhamentos |
| Dados sensíveis em compartilhamentos sem classificação ou controle | Média | Classificar e restringir os dados |
| Ausência de registro de acesso aos servidores de arquivos | Média | Habilitar a auditoria de acesso a arquivos |
| Telemetria sem tamanho dos arquivos criados | Baixa | Ampliar a telemetria do endpoint |
| Ausência de controle de saída de dados por volume | Média | Avaliar monitoramento e limites de saída |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Parcialmente |
| **Ruído permite alerta contínuo?** | Compactação com senha e divisão em volumes fora das rotinas de backup podem virar alertas depois das exceções do cliente. Volume alto seguido de saída de dados é uma boa detecção se a plataforma permitir correlacionar processos, arquivos e rede. O stacking por origem continua no hunt manual. |
| **Tipos de plataforma alvo** | SIEM, EDR |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1560.001 Archive via Utility](https://attack.mitre.org/techniques/T1560/001/)
- MITRE ATT&CK — [T1074.001 Local Data Staging](https://attack.mitre.org/techniques/T1074/001/)
- MITRE ATT&CK — [T1074.002 Remote Data Staging](https://attack.mitre.org/techniques/T1074/002/)
- MITRE ATT&CK — [T1030 Data Transfer Size Limits](https://attack.mitre.org/techniques/T1030/)
- MITRE ATT&CK — [Collection (TA0009)](https://attack.mitre.org/tactics/TA0009/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
