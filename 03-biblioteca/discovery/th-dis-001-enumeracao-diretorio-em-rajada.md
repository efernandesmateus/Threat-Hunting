# TH-DIS-001 — Enumeração do diretório e de grupos privilegiados em rajada

| Campo | Valor |
|-------|-------|
| **ID** | TH-DIS-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Discovery (TA0007) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 2–3 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, **rajadas** de enumeração do domínio Active Directory (contas, grupos privilegiados, relações de confiança, controladores e computadores) a partir de um único host ou conta em pouco tempo, e o uso de ferramentas de enumeração conhecidas, que indiquem um adversário mapeando o ambiente.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças.

**Contexto:** depois de obter o primeiro acesso, o adversário precisa saber **onde está** e **para onde ir**: quem são os administradores do domínio, quais grupos têm privilégios, que outros domínios confiam neste, onde ficam os controladores e os servidores importantes. Essa etapa de reconhecimento interno costuma acontecer cedo, logo depois da entrada, e muitas vezes de forma automatizada.

Consultar o diretório é uma atividade comum: administradores, scripts e aplicações fazem isso o tempo todo. O que diferencia o reconhecimento de um adversário é o **padrão**: muitas consultas diferentes, em sequência rápida, a partir de um host ou conta que normalmente não faz isso, cobrindo exatamente o que interessa a um atacante (grupos privilegiados, confianças, controladores). Ferramentas especializadas de enumeração deixam esse padrão ainda mais evidente.

**Ameaças associadas:** o MITRE ATT&CK registra o uso de ferramentas como o AdFind (S0552) e o BloodHound (S0521) para enumerar contas, grupos e relações de confiança do domínio, por diversos grupos e operadores de ransomware, além do uso de utilitários nativos como `net` e `nltest`. Ver referências.

## 3. Hipótese

- **Dado que** adversários mapeiam o domínio logo depois de obter acesso, para encontrar contas privilegiadas e caminhos de ataque,
- **quando** eles enumeram contas, grupos, confianças e computadores,
- **então** aparecerá uma rajada de comandos ou consultas de enumeração vindos de um único host ou conta em pouco tempo, ou a execução de ferramentas de enumeração conhecidas, frequentemente a partir de hosts e contas que não fazem administração do diretório.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Discovery | Account Discovery: Domain Account | T1087.002 |
| Discovery | Permission Groups Discovery: Domain Groups | T1069.002 |
| Discovery | Domain Trust Discovery | T1482 |
| Discovery | Remote System Discovery | T1018 |

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Telemetria de processos do endpoint (EDR) ou logs do sistema operacional com criação de processos | Criação de processos com linha de comando completa, usuário, host | 30 dias | Sim, **ou** a fonte abaixo |
| Logs do diretório (controladores de domínio) | Consultas LDAP com origem e filtro (quando o registro detalhado estiver habilitado); acesso a objetos do diretório e do banco de contas (4661, 4662); enumeração de grupos locais (4798, 4799) | 30 dias | Sim, **ou** a fonte acima |
| Inventário de contas administrativas e de hosts de administração | Quem administra o diretório e a partir de onde | — | Complementar |

**Escopo de ativos:** estações de trabalho, servidores e controladores de domínio.

**Limitação importante:** com apenas a telemetria de processos, o hunt detecta a enumeração feita por **comandos e ferramentas**, mas não a feita por **consultas diretas ao diretório** (por exemplo, por bibliotecas dentro de outro processo). Com apenas os logs do diretório, ele detecta as consultas, mas perde o contexto do processo que as fez. As duas fontes juntas dão a melhor cobertura.

## 6. Comportamento normal (baseline)

A enumeração legítima vem de administradores do domínio, de scripts de inventário e de auditoria, de aplicações que consultam o diretório (autenticação, provisionamento) e de ferramentas de gestão. Ela tende a partir de **hosts de administração** e **contas administrativas** conhecidas, ou de servidores de aplicação com padrão estável.

**Como levantar o baseline em um cliente novo:**

1. Em pelo menos 14 dias, faça o stacking das execuções de comandos de enumeração (`net` com `/domain`, `nltest`, cmdlets de consulta ao diretório) por **(host, conta)**.
2. Identifique as contas e os hosts de administração e as rotinas automatizadas de inventário. Registre-os como exceções do cliente, fora deste repositório.
3. Se houver registro de consultas LDAP, levante o volume típico de consultas por origem e identifique as aplicações que fazem consultas em massa legitimamente.
4. Registre se ferramentas como o AdFind ou o BloodHound são usadas pelo time do cliente em avaliações autorizadas, e em que períodos.

## 7. Estratégia de busca

**Técnicas de análise:** agrupamento temporal (rajada), stacking, busca por padrões e enriquecimento.

**Lógica:**

1. Coletar as execuções de comandos e ferramentas de enumeração do domínio:
   - `net` (`net.exe`, `net1.exe`) com `group`, `user` ou `accounts` e `/domain`, especialmente consultando grupos privilegiados (Domain Admins, Enterprise Admins, Administrators);
   - `nltest` com `/domain_trusts`, `/all_trusts`, `/trusted_domains`, `/dclist` ou `/dsgetdc`;
   - cmdlets de consulta ao diretório no PowerShell com filtros amplos (por exemplo, todos os usuários, computadores ou grupos);
   - ferramentas de enumeração do catálogo do ATT&CK: AdFind (parâmetros como `trustdmp`, `domainlist`, `dcmodes`, `objectcategory=`) e o coletor do BloodHound (parâmetros como `-CollectionMethod`).
2. **Rajada:** agrupar as execuções por **(host, conta)** em janelas curtas e contar quantos comandos e quantos **tipos diferentes** de enumeração aparecem.
3. Se houver logs do diretório, destacar origens com volume de consultas muito acima do baseline ou com consultas a grupos privilegiados e a relações de confiança em sequência.
4. Destacar a enumeração feita a partir de **hosts e contas fora do baseline de administração**, especialmente estações de usuários comuns.
5. Enriquecer com a atividade anterior no host (acesso inicial, execução) e com o que aconteceu depois (acesso a credenciais, movimentação lateral).

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial |
|-----------|--------------------|
| Comandos de enumeração pelo mesmo (host, conta) | ≥ 5 em 15 minutos |
| Tipos diferentes de enumeração na mesma rajada | ≥ 3 (por exemplo, usuários, grupos privilegiados e confianças) |
| Origem fora do baseline de administração | Qualquer rajada |
| Consultas LDAP por origem | Acima de 10 vezes a mediana da origem no baseline |

**Prioridade:** uso de ferramentas de enumeração conhecidas (AdFind, BloodHound) e rajadas a partir de estações de usuários comuns já justificam a triagem, mesmo isolados.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Criação de processos com linha de comando (EDR ou logs do sistema operacional)
            Complementar: logs do diretório nos controladores de domínio
            Últimos 30 dias

FILTRO:     Excluir (host, conta) de administração e rotinas de inventário da lista de
            exceções do cliente

TIPOS DE ENUMERAÇÃO, por execução:
            contas      = net user /domain; cmdlets de consulta a usuários com filtro amplo
            grupos      = net group /domain; consulta a grupos privilegiados
            confiancas  = nltest /domain_trusts, /all_trusts, /trusted_domains
            controladores = nltest /dclist, /dsgetdc
            computadores = consulta ampla a objetos de computador
            ferramenta  = parâmetros do AdFind (trustdmp, domainlist, dcmodes, adinfo,
                          -f objectcategory=) ou do coletor do BloodHound
                          (-CollectionMethod, --CollectionMethods)

AGREGAÇÃO:  Por (host, conta), em janelas de 15 minutos:
              total_comandos  = contagem de execuções de enumeração
              tipos_distintos = contagem distinta de tipos de enumeração
            origem_admin = (host, conta) está no baseline de administração

DESTAQUE:   ferramenta
            OU (total_comandos >= 5 E NÃO origem_admin)
            OU (tipos_distintos >= 3)
            (valores de referência; calibrar pelo baseline do cliente)

COMPLEMENTO (logs do diretório):
            consultas_origem = consultas LDAP por origem, por hora
            DESTAQUE adicional: consultas_origem > 10 × mediana da origem
                                OU consultas a grupos privilegiados e confianças em
                                sequência, a partir de origem fora do baseline

SAÍDA:      host, conta, janela, total_comandos, tipos_distintos, linhas de comando,
            origem_admin, atividade anterior e posterior no host
            Ordenar por tipos_distintos e total_comandos
```

**Observações:**

- Rajadas costumam ter comandos em sequência de poucos segundos, típica de scripts ou ferramentas; comandos espaçados ao longo do dia tendem a ser administração manual.
- Ferramentas de enumeração podem ser renomeadas; os parâmetros da linha de comando são mais confiáveis que o nome do executável.

### 8.2 Regra Sigma

**Status:** Sintaxe verificada; não validada em plataforma real
**Arquivo:** [`th-dis-001-enumeracao-diretorio-em-rajada.yml`](th-dis-001-enumeracao-diretorio-em-rajada.yml)

O arquivo contém cinco regras, com sintaxe verificada por um analisador Sigma de código aberto:

| Regra | Nível |
|-------|-------|
| Enumeração de contas e grupos do domínio com `net` (base da correlação) | low |
| Enumeração de controladores e relações de confiança com `nltest` (base da correlação) | medium |
| Parâmetros de enumeração do AdFind | high |
| Coletor de dados do BloodHound | high |
| **Correlação:** rajada de 5 ou mais comandos de enumeração pelo mesmo usuário e host em 15 minutos | medium |

Limitações da versão Sigma em relação à pseudoconsulta:

- A correlação conta comandos, mas não **tipos distintos** de enumeração, nem exclui o baseline de administração; essa parte é feita na execução do hunt.
- As consultas diretas ao diretório (logs dos controladores) não têm regra, porque o formato desses registros varia muito conforme a configuração.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Telemetria esperada:** a criação dos processos de enumeração com a linha de comando completa, agrupados por host e conta em sequência curta; nos controladores, as consultas correspondentes, quando o registro estiver habilitado.

**Como gerar os dados (somente em laboratório ou com autorização formal):** em um domínio de laboratório, executar a partir de uma estação comum uma sequência de comandos nativos de enumeração (`net group "Domain Admins" /domain`, `net user /domain`, `nltest /domain_trusts`, `nltest /dclist:<domínio>`) em poucos minutos. Para ferramentas especializadas, use ferramentas de emulação de adversário com testes mapeados às técnicas T1087.002, T1069.002 e T1482. Confirme que a rajada aparece destacada e que a administração do baseline não aparece.

**Alternativa sem laboratório:** usar datasets públicos de telemetria gravados durante simulações de reconhecimento do domínio.

**Validação histórica:** executar o hunt em pelo menos 30 dias de dados reais, com autorização, para calibrar o limiar de rajada e montar a lista de exceções de administração.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | A enumeração vem de conta e host de administração conhecidos, em horário compatível; é rotina de inventário identificada; é avaliação de segurança autorizada e informada previamente. |
| **Suspeito — investigar** | Rajada a partir de conta administrativa conhecida, mas de host incomum; enumeração pontual a partir de estação de usuário comum, sem outros indicadores. |
| **Malicioso — escalar** | Ferramenta de enumeração sem avaliação autorizada; rajada com vários tipos de enumeração a partir de estação de usuário comum; enumeração precedida de acesso inicial incomum ou de execução suspeita, ou seguida de acesso a credenciais ou movimentação lateral. |

**Perguntas de enriquecimento:**

- A conta e o host fazem administração do diretório? O usuário confirma a atividade?
- Qual processo executou os comandos? Foi um shell interativo, um script ou outro processo?
- O que aconteceu antes no host? (ver [TH-INI-001](../initial-access/th-ini-001-acesso-remoto-origem-incomum.md), [TH-EXE-001](../execution/th-exe-001-interpretadores-comandos-ofuscados.md))
- O que aconteceu depois? Houve acesso a credenciais ([TH-CRD-001](../credential-access/th-crd-001-acesso-memoria-processo-autenticacao.md)) ou logons em outros hosts ([TH-LAT-001](../lateral-movement/th-lat-001-movimentacao-lateral-contas-administrativas.md))?
- Havia avaliação de segurança ou pentest autorizado no período?

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Administradores do domínio | Contas e hosts de administração conhecidos; horário de trabalho | Exceção por (host, conta) |
| Scripts de inventário e auditoria | Execução periódica, mesmo padrão e mesmo horário | Exceção por (host, conta, horário) |
| Aplicações que consultam o diretório | Servidores de aplicação com volume estável de consultas | Exceção por origem no baseline de consultas |
| Avaliações de segurança autorizadas | Período e responsáveis informados previamente | Confirmar com o ponto focal |
| Suporte técnico resolvendo problemas de acesso | Consultas pontuais a usuários e grupos, por equipe de suporte | Avaliar caso a caso |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): linhas de comando, árvore de processos, consultas ao diretório e saídas gravadas em disco pelas ferramentas (as ferramentas de enumeração costumam salvar resultados em arquivos).
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Reconhecimento malicioso confirmado é **Alta**, e **Crítica** se acompanhado de acesso a credenciais, movimentação lateral ou envolvimento de contas privilegiadas, porque indica um adversário ativo dentro do domínio.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - isolar o host de origem;
   - bloquear ou redefinir a conta usada;
   - procurar a mesma atividade a partir de outros hosts e contas;
   - tratar as informações enumeradas como conhecidas pelo adversário (por exemplo, a lista de administradores) e reforçar o monitoramento dessas contas.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Grupos privilegiados com membros em excesso ou desnecessários | Média | Revisar e reduzir os membros |
| Contas sem exigência de senha ou com senha que nunca expira | Média | Corrigir as configurações das contas |
| Relações de confiança não documentadas ou desnecessárias | Média | Revisar as relações de confiança |
| Qualquer usuário autenticado consegue enumerar todo o diretório sem restrição | Baixa | Avaliar restrições de enumeração |
| Ausência de registro de consultas ao diretório e de auditoria de acesso a objetos | Média | Habilitar a auditoria nos controladores, com filtros para controlar o volume |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Sim, para parte dos indicadores |
| **Ruído permite alerta contínuo?** | Ferramentas de enumeração conhecidas têm ruído baixo e devem virar alertas. A correlação de rajada pode virar alerta depois que as exceções de administração estiverem estáveis. Comandos isolados de `net` e `nltest` geram ruído demais para alerta. |
| **Tipos de plataforma alvo** | SIEM, EDR |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1087.002 Account Discovery: Domain Account](https://attack.mitre.org/techniques/T1087/002/)
- MITRE ATT&CK — [T1069.002 Permission Groups Discovery: Domain Groups](https://attack.mitre.org/techniques/T1069/002/)
- MITRE ATT&CK — [T1482 Domain Trust Discovery](https://attack.mitre.org/techniques/T1482/)
- MITRE ATT&CK — [T1018 Remote System Discovery](https://attack.mitre.org/techniques/T1018/)
- MITRE ATT&CK — [S0552 AdFind](https://attack.mitre.org/software/S0552/)
- MITRE ATT&CK — [S0521 BloodHound](https://attack.mitre.org/software/S0521/)
- [Especificação Sigma](https://github.com/SigmaHQ/sigma-specification)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Regras Sigma com sintaxe verificada; não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
