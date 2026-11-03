# TH-EXF-001 — Exfiltração de dados via DNS

| Campo | Valor |
|-------|-------|
| **ID** | TH-EXF-001 |
| **Versão** | 1.1 |
| **Status** | Proposto |
| **Tática principal** | Exfiltration (TA0010) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 2–3 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, hosts que enviaram dados para fora do ambiente codificando-os em consultas DNS: muitos subdomínios únicos, longos e de alta aleatoriedade para um mesmo domínio registrado, em volume acima do baseline do host.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** DNS é permitido em praticamente todos os ambientes e raramente é inspecionado em detalhe. Isso o torna um canal atraente quando outros caminhos de saída estão bloqueados ou monitorados. Na técnica mais comum, o adversário controla um domínio e seu servidor autoritativo; o host comprometido divide os dados em blocos, codifica cada bloco (por exemplo, em base32, base64 ou hexadecimal) e o envia como subdomínio desse domínio. Cada consulta chega ao servidor do adversário pela cadeia normal de resolução, sem conexão direta entre o host e o adversário.

O volume por consulta é pequeno (um rótulo DNS tem no máximo 63 caracteres e um nome completo, 253), então exfiltrar qualquer quantidade relevante de dados exige **muitas** consultas. É essa assinatura de volume e de forma que o hunt procura.

**Ameaças associadas:** o MITRE ATT&CK registra, entre outros, o uso de DNS para exfiltração pelo grupo APT32 (dados codificados no campo de subdomínio) e pelo APT41 na campanha C0017 (dados codificados e prefixados como subdomínios de um domínio controlado pelo adversário). Ver referências.

## 3. Hipótese

- **Dado que** adversários usam DNS para tirar dados de ambientes onde outros canais são bloqueados ou monitorados,
- **quando** um host codifica dados em consultas DNS para um domínio controlado pelo adversário,
- **então** aparecerão, para um mesmo domínio registrado, muitos subdomínios únicos, longos e de alta aleatoriedade, vindos de poucos hosts e em volume acima do baseline desses hosts.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Exfiltration | Exfiltration Over Alternative Protocol: Exfiltration Over Unencrypted Non-C2 Protocol | T1048.003 |
| Command and Control | Application Layer Protocol: DNS | T1071.004 |
| Command and Control | Protocol Tunneling | T1572 |

A técnica principal é a **T1048.003**, quando o DNS é usado apenas para a saída dos dados. Se o mesmo canal DNS também for usado para comando e controle, o achado se encaixa em **T1071.004**, e a exfiltração passa a ser pelo próprio canal de C2. O hunt encontra os dois casos; a classificação final é feita na triagem.

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Logs do resolvedor DNS interno | Data e hora, host de origem (IP ou nome), nome consultado completo, tipo de registro, código de resposta | 30 dias | Sim |
| Logs de firewall ou proxy com consultas DNS | Consultas DNS que saem do ambiente, destino (resolvedor usado), porta e protocolo | 30 dias | Complementar |
| Telemetria de processos do endpoint (EDR) | Processo que gerou as consultas DNS, linha de comando, usuário | 30 dias | Complementar |
| Reputação e idade de domínios | Data de registro, reputação, categoria | — | Complementar |

**Escopo de ativos:** todos os hosts que usam o resolvedor interno: estações de trabalho, servidores e controladores de domínio.

**Requisito crítico:** os logs do resolvedor precisam identificar o **host de origem real**. Se as consultas passam por encaminhadores e o log só registra o encaminhador, o hunt mostra que há exfiltração, mas não de onde. Nesse caso, a lacuna deve ser registrada como "outro achado" (seção 13).

## 6. Comportamento normal (baseline)

Nomes DNS longos e aparentemente aleatórios são **comuns** em ambientes legítimos. Exemplos de origem legítima:

- serviços de reputação e de antimalware que consultam hashes ou identificadores via DNS;
- serviços de nuvem, CDN e telemetria que usam nomes gerados automaticamente;
- listas de bloqueio baseadas em DNS consultadas por servidores de e-mail;
- resolução reversa (`in-addr.arpa`, `ip6.arpa`).

O que diferencia esses casos da exfiltração costuma ser a **distribuição**: um serviço legítimo é consultado por **muitos** hosts do ambiente, enquanto um canal de exfiltração costuma envolver **um ou poucos** hosts.

**Como levantar o baseline em um cliente novo:**

1. Em um período de pelo menos 7 dias, liste os domínios registrados com mais subdomínios únicos.
2. Para cada um, conte quantos hosts distintos o consultam.
3. Domínios consultados por grande parte do ambiente e conhecidos como legítimos entram na **lista de exceções do cliente** (mantida fora deste repositório).
4. Registre, para referência, a média e o percentil 95 de subdomínios únicos por host por dia.

## 7. Estratégia de busca

**Técnicas de análise:** stacking, análise de comprimento e entropia, agrupamento por domínio registrado e enriquecimento.

**Lógica:**

1. Coletar as consultas DNS do período, excluindo resolução reversa e domínios da lista de exceções do cliente.
2. Separar cada nome em **domínio registrado** (ex.: `example.com`) e **subdomínio** (tudo à esquerda dele).
3. Para cada par **(host, domínio registrado)**, calcular:
   - quantidade de subdomínios únicos;
   - comprimento médio e máximo do subdomínio;
   - entropia média do subdomínio (aleatoriedade dos caracteres);
   - proporção de consultas de tipos incomuns para o host (TXT, NULL);
   - quantidade de hosts distintos que consultam o mesmo domínio registrado.
4. Destacar os pares que combinam **volume alto** de subdomínios únicos com **comprimento ou entropia altos**, consultados por **poucos hosts**.
5. Enriquecer os destaques com idade e reputação do domínio e, se houver telemetria de endpoint, com o processo que gerou as consultas.
6. Ordenar por quantidade de subdomínios únicos e encaminhar para a triagem (seção 10).

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Subdomínios únicos por (host, domínio registrado) em 24 h | ≥ 300 |
| Comprimento de algum rótulo do subdomínio | ≥ 50 caracteres |
| Entropia média do subdomínio | ≥ 3,5 bits por caractere |
| Hosts distintos consultando o domínio registrado | ≤ 3 |
| Consultas TXT ou NULL de um host para um mesmo domínio em 24 h | ≥ 100 |
| Idade do domínio registrado (enriquecimento) | < 30 dias aumenta a suspeita |

Nenhum indicador isolado é suficiente. A combinação de volume alto, nomes longos ou aleatórios e poucos hosts é o que sustenta a hipótese.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Logs do resolvedor DNS interno, últimos 30 dias

FILTRO:     Excluir nomes terminados em .in-addr.arpa e .ip6.arpa
            Excluir domínios registrados da lista de exceções do cliente

CAMPOS:     host_origem
            dominio_registrado   = domínio registrável do nome consultado
            subdominio           = parte do nome à esquerda de dominio_registrado
            tipo_registro

AGREGAÇÃO:  Agrupar por (host_origem, dominio_registrado), por dia:
              subdominios_unicos = contagem distinta de subdominio
              tam_max            = maior comprimento de rótulo em subdominio
              entropia_media     = média da entropia de Shannon de subdominio
              consultas_txt_null = contagem onde tipo_registro em (TXT, NULL)
            Para cada dominio_registrado, no período:
              hosts_distintos    = contagem distinta de host_origem

LIMIAR:     (   subdominios_unicos >= 300
              E (tam_max >= 50 OU entropia_media >= 3,5)
              E hosts_distintos <= 3 )
            OU (consultas_txt_null >= 100 E hosts_distintos <= 3)
            (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      host_origem, dominio_registrado, dia, subdominios_unicos, tam_max,
            entropia_media, consultas_txt_null, hosts_distintos,
            3 exemplos de nomes consultados
            Ordenar por subdominios_unicos (decrescente)
```

**Observações:**

- Separar o domínio registrado exige considerar sufixos públicos com mais de um nível (ex.: `.com.br`). Se a plataforma não tiver uma função para isso, use os dois ou três últimos rótulos, conforme o sufixo, e revise os resultados.
- Se a plataforma não calcular entropia, use apenas o comprimento e a proporção de dígitos no subdomínio como aproximação.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-exf-001-exfiltracao-via-dns.yml`](th-exf-001-exfiltracao-via-dns.yml)

O arquivo contém duas regras, com sintaxe verificada por um analisador Sigma de código aberto:

1. **Rótulo longo** (`level: low`): consulta DNS cujo primeiro rótulo tem 50 caracteres ou mais, excluindo resolução reversa.
2. **Correlação por host** (`level: medium`): um mesmo host gerando 100 ou mais nomes longos distintos em uma hora.

Limitações da versão Sigma em relação à pseudoconsulta:

- O Sigma não separa o domínio registrado, então a correlação agrupa **por host**, e não por (host, domínio).
- Entropia e contagem de hosts distintos por domínio não são expressas nas regras; essa parte da análise é feita na execução do hunt.
- Os nomes de campos (`query`, `src_ip`) seguem a taxonomia genérica do Sigma e precisam ser mapeados na conversão.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Dados de teste (somente em laboratório ou com autorização formal):**

1. Configurar um domínio de teste controlado pelo time, com um servidor autoritativo de laboratório que registre as consultas recebidas.
2. Em um host de laboratório, gerar um arquivo fictício (sem dados reais), codificá-lo em base32 e dividi-lo em blocos de até 60 caracteres.
3. Enviar cada bloco como subdomínio do domínio de teste (ex.: `<bloco>.<sequencial>.teste.example`), com intervalos variados.
4. Repetir uma variação com consultas do tipo TXT e outra com intervalos longos entre as consultas (exfiltração lenta).
5. Confirmar no servidor de laboratório que o arquivo pode ser reconstruído a partir das consultas, para comprovar que o teste representa exfiltração real.

Alternativa sem laboratório: usar datasets públicos de telemetria ou capturas de rede gravadas durante simulações de tunelamento DNS.

> **Nunca** gere esses dados no ambiente de um cliente sem autorização formal.

**Validação histórica:** executar o hunt em pelo menos 30 dias de logs DNS reais, com autorização, para medir o ruído e montar a lista de exceções do ambiente.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | O domínio pertence a um serviço legítimo identificado (reputação, nuvem, telemetria, segurança); é consultado por vários hosts ou o processo de origem é o software legítimo correspondente. Incluir na lista de exceções do cliente. |
| **Suspeito — investigar** | Volume e forma batem com a hipótese, mas o domínio não tem reputação conhecida e não há telemetria de processo para concluir; ou o domínio é recente; ou o processo de origem é legítimo, mas não deveria fazer consultas desse tipo. |
| **Malicioso — escalar** | Os subdomínios decodificam para dados do ambiente; ou o processo de origem é desconhecido, não assinado, um interpretador de scripts ou uma ferramenta de tunelamento; ou o domínio aparece em inteligência de ameaças; ou o padrão persiste por dias a partir de um único host sem justificativa do cliente. |

**Perguntas de enriquecimento:**

- Quantos hosts do ambiente consultam este domínio? Se for só um, por que esse host?
- Qual é a idade e a reputação do domínio registrado? Quem é o servidor autoritativo?
- Qual processo gerou as consultas? É assinado? Qual é a linha de comando?
- Os subdomínios **decodificam** para algo legível (base32, base64, hexadecimal)? Tentar a decodificação de alguns exemplos é o teste mais rápido.
- O padrão é contínuo ou em rajadas? Coincide com horário de trabalho ou ocorre fora dele?
- O host é um ativo crítico ou tem acesso a dados sensíveis?
- Houve preparação de dados no host antes das consultas, como compactação de grandes volumes? (ver [TH-COL-001](../collection/th-col-001-compactacao-grandes-volumes.md))
- O host mostra outros sinais de comunicação com infraestrutura do adversário? (ver [TH-C2-001](../command-and-control/th-c2-001-beaconing-destinos-raros.md))
- O cliente reconhece o domínio? (consultar o ponto focal)

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Serviços de reputação e antimalware via DNS | Domínio do fornecedor do software de segurança; consultado por muitos hosts; processo é o agente de segurança | Lista de exceções do cliente |
| Serviços de nuvem, CDN e telemetria | Domínio conhecido e de reputação boa; muitos hosts; processo é o software do serviço | Lista de exceções do cliente |
| Listas de bloqueio DNS em servidores de e-mail | Origem é o servidor de e-mail; subdomínios são IPs invertidos ou hashes | Excluir o servidor de e-mail para esses domínios |
| Resolução reversa | Nomes terminados em `.in-addr.arpa` ou `.ip6.arpa` | Já excluída na consulta |
| Ferramentas de inventário ou varredura internas | Origem é um servidor de gestão conhecido; muitas consultas a nomes internos | Excluir domínios internos do cliente |

## 12. Ação ao encontrar atividade maliciosa

1. Interromper o hunt e preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): consultas DNS do host, exemplos de subdomínios e, se houver, a telemetria do processo.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Exfiltração confirmada é sempre de severidade **Alta** ou **Crítica**; é **Crítica** se estiver em andamento ou envolver ativos críticos ou dados sensíveis.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - bloquear a resolução do domínio registrado no resolvedor interno;
   - isolar o host de origem para investigação;
   - identificar e preservar o processo responsável;
   - verificar se outros hosts consultaram o mesmo domínio ou domínios com o mesmo servidor autoritativo.
4. Tentar estimar o **volume exfiltrado** (quantidade de consultas × tamanho médio do subdomínio) e o **conteúdo**, decodificando os subdomínios. Essa informação é essencial para a resposta a incidentes e para obrigações legais do cliente.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Hosts usando resolvedores externos diretamente, contornando o resolvedor interno | Média | Bloquear DNS de saída que não venha do resolvedor interno |
| Uso de DNS criptografado (DoH/DoT) fora do padrão do ambiente | Média | Definir política para DNS criptografado; ele torna as consultas invisíveis a este hunt |
| Logs do resolvedor sem o host de origem real | Baixa | Ajustar o registro para identificar o cliente DNS original |
| Retenção de logs DNS menor que 30 dias | Baixa | Ampliar a retenção |
| Domínios internos resolvendo para endereços externos | Média | Revisar a configuração de DNS |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Parcialmente |
| **Ruído permite alerta contínuo?** | A regra de rótulo longo, isolada, não. A correlação por host, após a calibração de exceções do cliente, pode gerar alerta de prioridade baixa ou média para revisão humana. |
| **Tipos de plataforma alvo** | SIEM |
| **Regra criada** | — |

A parte de baseline e de distribuição por hosts continua dependendo do hunt manual.

## 15. Referências

- MITRE ATT&CK — [T1048.003 Exfiltration Over Unencrypted Non-C2 Protocol](https://attack.mitre.org/techniques/T1048/003/)
- MITRE ATT&CK — [Exfiltration (TA0010)](https://attack.mitre.org/tactics/TA0010/)
- MITRE ATT&CK — [T1071.004 Application Layer Protocol: DNS](https://attack.mitre.org/techniques/T1071/004/)
- MITRE ATT&CK — [T1572 Protocol Tunneling](https://attack.mitre.org/techniques/T1572/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |
| 1.1 | 2026-11-03 | Eduardo Fernandes Mateus | Perguntas de enriquecimento com referência aos hunts TH-COL-001 e TH-C2-001. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
