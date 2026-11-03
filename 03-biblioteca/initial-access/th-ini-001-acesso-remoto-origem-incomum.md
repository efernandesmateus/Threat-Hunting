# TH-INI-001 — Acesso remoto externo de origem incomum

| Campo | Valor |
|-------|-------|
| **ID** | TH-INI-001 |
| **Versão** | 1.0 |
| **Status** | Proposto |
| **Tática principal** | Initial Access (TA0001) |
| **Tipos de plataforma com consulta validada** | Nenhum (ver seção 9) |
| **Versão do ATT&CK** | v20 |
| **Esforço estimado por execução** | 2–4 h por cliente (estimativa inicial, recalibrar após as primeiras execuções) |
| **Autor** | Eduardo Fernandes Mateus |
| **Revisor** | — |
| **Criado em** | 2026-11-03 |
| **Última revisão** | 2026-11-03 |

## 1. Objetivo

Identificar, nos últimos 30 dias, autenticações **bem-sucedidas** no acesso remoto externo do cliente (VPN, gateways de acesso remoto, portais) que fujam do padrão da própria conta: origem geográfica ou de rede nunca vista, deslocamento impossível, horário atípico, dispositivo novo, sucesso após uma sequência de falhas ou uso de contas inativas e de serviço.

## 2. Origem e contexto

**Origem:** Inteligência de ameaças e superfície de ataque.

**Contexto:** credenciais válidas são um dos caminhos mais comuns de acesso inicial. Elas chegam ao adversário por vazamentos, phishing, malware de roubo de credenciais, força bruta ou compra de acessos de terceiros. Com elas, o adversário não precisa explorar nenhuma vulnerabilidade: autentica-se pelo mesmo serviço que os funcionários usam para trabalhar remotamente, e o acesso parece legítimo para os controles.

O que denuncia o acesso não é a autenticação em si, mas o **contexto**: de onde veio, quando, de qual dispositivo, depois de quê e com qual conta. O hunt compara cada acesso com o histórico da própria conta.

**Ameaças associadas:** o MITRE ATT&CK registra o uso de serviços remotos externos e de contas válidas por diversos grupos e operadores de ransomware, frequentemente com acessos comprados de intermediários. Ver referências.

## 3. Hipótese

- **Dado que** adversários usam credenciais válidas para entrar pelo acesso remoto externo,
- **quando** eles se autenticam a partir da própria infraestrutura,
- **então** aparecerão autenticações bem-sucedidas com atributos raros para a conta (origem, horário, dispositivo, sequência de falhas) ou com contas que não deveriam estar em uso.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| Initial Access | External Remote Services | T1133 |
| Initial Access | Valid Accounts | T1078 |
| Credential Access | Brute Force (relacionada: sucesso após falhas) | T1110 |

A técnica principal é a **T1133**. A **T1078** descreve o uso das credenciais em si, e a **T1110** entra quando o sucesso vem depois de uma sequência de falhas, indicando que a credencial foi obtida por força bruta ou por password spraying.

## 5. Fontes de dados necessárias

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| Logs do serviço de acesso remoto externo | Data e hora, conta, resultado (sucesso/falha), IP de origem, identificação do dispositivo ou cliente de acesso, duração da sessão | 90 dias (para histórico da conta) | Sim |
| Autenticação e diretório | Status da conta (ativa, desabilitada), último logon, grupos, tipo de conta (pessoal, serviço, administrativa) | — | Complementar |
| Geolocalização e reputação de IP | País, cidade aproximada, rede de origem (sistema autônomo), indicação de proxy anônimo, VPN comercial ou hospedagem | — | Complementar |
| Lista de colaboradores e viagens | Localização habitual, férias, desligamentos | — | Complementar |

**Escopo de ativos:** todos os serviços de acesso remoto externo do cliente.

**Requisito crítico:** o histórico da conta é a base do hunt. Com retenção menor que 90 dias, o "nunca visto" fica pouco confiável: uma origem rara, mas legítima (uma viagem trimestral, por exemplo), pode parecer nova.

## 6. Comportamento normal (baseline)

Para cada conta, levantar no período disponível:

- países e redes de origem habituais;
- faixa de horário em que ocorre a maior parte dos acessos (por exemplo, o intervalo que concentra 95% dos acessos da conta);
- dispositivos ou clientes de acesso habituais;
- frequência de acesso.

Causas legítimas comuns de desvio: viagens, trabalho em outro fuso horário, troca de dispositivo, uso de redes móveis (que mudam de rede de origem com frequência), prestadores de serviço e plantões.

**Como levantar o baseline em um cliente novo:**

1. Com pelo menos 30 dias de logs (90 recomendados), monte o perfil de cada conta conforme acima.
2. Identifique as contas com padrão naturalmente variável (equipes de viagem, prestadores) e trate-as com limiares próprios.
3. Registre as redes de origem legítimas compartilhadas (escritórios, redes de parceiros) como exceções do cliente, fora deste repositório.

## 7. Estratégia de busca

**Técnicas de análise:** comparação com o baseline da conta, stacking, análise temporal e enriquecimento.

**Lógica:**

1. Coletar as autenticações **bem-sucedidas** do período e, separadamente, as falhas.
2. Enriquecer cada sucesso com geolocalização, rede de origem e reputação do IP.
3. Para cada sucesso, comparar com o baseline da conta e marcar os indicadores presentes:
   - **origem nova:** país ou rede de origem nunca vistos para a conta;
   - **deslocamento impossível:** distância entre dois acessos consecutivos incompatível com o tempo entre eles;
   - **horário atípico:** fora da faixa habitual da conta;
   - **dispositivo novo:** identificação de dispositivo ou cliente de acesso nunca vista;
   - **sucesso após falhas:** sequência de falhas para a conta, ou do mesmo IP para várias contas, antes do sucesso;
   - **conta que não deveria acessar:** conta inativa há muito tempo, de serviço, desabilitada recentemente ou de colaborador desligado;
   - **origem de anonimização:** IP de proxy anônimo, VPN comercial ou provedor de hospedagem.
4. Pontuar cada acesso pela quantidade e pelo peso dos indicadores. Um indicador isolado é comum; a **combinação** é que sustenta a hipótese.
5. Para os acessos destacados, verificar o que aconteceu **depois**: duração da sessão, recursos acessados e atividade dos hosts alcançados.

**Valores de referência** (ponto de partida; calibrar pelo baseline de cada cliente):

| Indicador | Referência inicial | Peso sugerido |
|-----------|--------------------|---------------|
| País nunca visto para a conta (histórico de 90 dias) | Qualquer ocorrência | 2 |
| Rede de origem nunca vista para a conta | Qualquer ocorrência | 1 |
| Deslocamento impossível | Velocidade aparente acima de 900 km/h entre acessos consecutivos | 3 |
| Horário atípico | Fora do intervalo que concentra 95% dos acessos da conta | 1 |
| Dispositivo ou cliente de acesso novo | Qualquer ocorrência | 1 |
| Sucesso após falhas | ≥ 10 falhas da conta, ou do mesmo IP para ≥ 5 contas, na hora anterior | 3 |
| Conta inativa | Sem acesso há ≥ 60 dias | 2 |
| Conta de serviço, desabilitada ou de desligado | Qualquer acesso pelo serviço remoto | 3 |
| Origem de anonimização ou hospedagem | Qualquer ocorrência | 2 |

**Limiar de destaque de referência:** soma dos pesos **≥ 4**.

## 8. Consultas

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      Logs do serviço de acesso remoto externo
            Período analisado: últimos 30 dias
            Histórico para baseline: 90 dias anteriores ao período

FILTRO:     resultado = sucesso (análise principal)
            resultado = falha (contexto para "sucesso após falhas")
            Excluir redes de origem da lista de exceções do cliente

ENRIQUECER: ip_origem -> pais, rede_origem, anonimizacao (sim/não)
            conta     -> tipo_conta, status_conta, ultimo_acesso_anterior

INDICADORES, para cada sucesso (conta, data_hora):
            pais_novo        = pais não consta no histórico da conta
            rede_nova        = rede_origem não consta no histórico da conta
            desloc_impossivel = distância(acesso anterior, atual) ÷ tempo entre eles > 900 km/h
            horario_atipico  = hora fora da faixa de 95% dos acessos da conta
            dispositivo_novo = dispositivo não consta no histórico da conta
            sucesso_pos_falha = falhas(conta, 1 h antes) >= 10
                                OU contas_com_falha(mesmo ip, 1 h antes) >= 5
            conta_inativa    = data_hora − ultimo_acesso_anterior >= 60 dias
            conta_indevida   = tipo_conta = serviço OU status_conta em (desabilitada, desligado)
            anonimizacao     = anonimizacao = sim

PONTUAÇÃO:  2·pais_novo + 1·rede_nova + 3·desloc_impossivel + 1·horario_atipico
            + 1·dispositivo_novo + 3·sucesso_pos_falha + 2·conta_inativa
            + 3·conta_indevida + 2·anonimizacao

LIMIAR:     pontuação >= 4  (valores de referência; calibrar pelo baseline do cliente)

SAÍDA:      conta, data_hora, ip_origem, pais, rede_origem, dispositivo,
            indicadores presentes, pontuação, duração da sessão
            Ordenar por pontuação (decrescente)
```

**Observações:**

- A distância do "deslocamento impossível" usa a geolocalização aproximada do IP. Redes móveis e VPNs corporativas podem gerar falsos deslocamentos; considere a precisão da geolocalização na triagem.
- Se não houver identificação de dispositivo nos logs, remova o indicador e redistribua o limiar.

### 8.2 Regra Sigma

**Status:** Não se aplica

A lógica deste hunt depende do **histórico de cada conta** (origens, horários e dispositivos já vistos) e de **enriquecimento externo** (geolocalização e reputação). Nenhum desses elementos é expresso no formato Sigma, e os logs de serviços de acesso remoto não têm uma taxonomia de campos padronizada no Sigma. A execução usa a pseudoconsulta.

### 8.3 Implementações por plataforma

Traduções para plataformas específicas ficam fora do repositório (ADR 0005).

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| — | — | Nenhuma validação registrada até o momento (ver seção 9.2) |

## 9. Validação do hunt

> Este hunt está como **Proposto**: não foi validado em plataforma real pelo projeto. Valide-o no seu ambiente antes de usá-lo em clientes (ver [ADR 0006](../../decisoes/0006-validacao-pelo-adotante.md)).

### 9.1 Como validar no seu ambiente

**Dados de teste (somente com autorização formal):**

1. Criar uma conta de teste com acesso remoto e acumular histórico normal por alguns dias, a partir da rede habitual.
2. Autenticar com a mesma conta a partir de uma origem diferente: outra rede ou um serviço de anonimização, em horário atípico e a partir de um dispositivo novo.
3. Gerar uma sequência de falhas de autenticação seguida de um sucesso.
4. Confirmar que o hunt destaca os acessos dos passos 2 e 3 e não destaca o histórico normal do passo 1.

**Validação histórica:** executar o hunt em pelo menos 30 dias de logs reais, com 90 dias de histórico, com autorização, para medir o ruído e calibrar pesos e limiar.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| — | — | — | — | — | — | Nenhuma validação contribuída até o momento |

Validou este hunt? Veja como contribuir no [CONTRIBUTING](../../CONTRIBUTING.md#6-contribuindo-com-validações).

## 10. Triagem dos resultados

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | O desvio tem explicação confirmada: viagem conhecida, troca de dispositivo registrada, trabalho remoto em outro local informado, rede móvel. A atividade após o acesso é compatível com a função do usuário. |
| **Suspeito — investigar** | Combinação de indicadores sem explicação imediata; usuário ou gestor ainda não confirmou; sessão curta ou sem atividade clara depois do acesso. |
| **Malicioso — escalar** | O usuário não reconhece o acesso; deslocamento impossível sem explicação técnica; conta de serviço, desabilitada ou de desligado usada pelo acesso remoto; sucesso após força bruta seguido de atividade incomum; IP associado a ameaça em inteligência. |

**Perguntas de enriquecimento:**

- O usuário reconhece o acesso? (confirmar pelo ponto focal ou pelo gestor, nunca pelo próprio canal possivelmente comprometido)
- O usuário estava viajando ou em outro local?
- A conta tem segundo fator de autenticação? Ele foi solicitado e aprovado? Houve várias solicitações seguidas antes da aprovação?
- O que a sessão fez depois do acesso? Quais hosts e recursos foram alcançados?
- A mesma origem tentou ou conseguiu acessar outras contas?
- A conta teve alterações recentes (senha, segundo fator, grupos)?

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| Viagens e trabalho em outro local | Confirmação do usuário ou do gestor; padrão consistente durante a viagem | Registrar a explicação; não criar exceção permanente |
| Redes móveis | Rede de origem de operadora móvel; geolocalização imprecisa | Reduzir o peso de "rede nova" e de deslocamento para essas redes |
| Escritórios, parceiros e prestadores | Redes conhecidas, compartilhadas por várias contas | Lista de exceções do cliente |
| VPN corporativa encadeada ou serviços de segurança na nuvem | Origem é uma rede do próprio cliente ou de um serviço contratado | Lista de exceções do cliente |
| Troca de dispositivo | Dispositivo novo seguido de uso contínuo e normal | Confirmar com o suporte de TI |

## 12. Ação ao encontrar atividade maliciosa

1. Preservar as evidências no sistema de atendimento do SOC (nunca neste repositório): autenticações da conta, IPs, dispositivos e atividade da sessão.
2. Escalar conforme o [processo de escalonamento](../../01-processo/escalonamento.md). Acesso confirmado por credencial comprometida é **Crítico** se a sessão ainda estiver ativa ou se houve acesso a ativos críticos.
3. Recomendar ao cliente, conforme o modo de resposta contratado:
   - encerrar as sessões ativas da conta e bloquear temporariamente o acesso remoto dela;
   - redefinir a senha e os fatores de autenticação da conta;
   - bloquear o IP ou a rede de origem no acesso remoto;
   - verificar se a mesma origem acessou outras contas;
   - investigar os hosts alcançados durante a sessão.

## 13. Outros achados possíveis

| Achado | Severidade sugerida | Recomendação |
|--------|---------------------|--------------|
| Contas com acesso remoto sem segundo fator de autenticação | Média | Exigir segundo fator para todo acesso remoto |
| Contas de serviço habilitadas para acesso remoto | Média | Remover o acesso remoto de contas de serviço |
| Contas de colaboradores desligados ainda ativas | Média | Integrar o desligamento à revogação de acessos |
| Contas inativas há muito tempo com acesso remoto | Baixa | Revisão periódica de contas e acessos |
| Logs do acesso remoto sem identificação de dispositivo ou com retenção curta | Baixa | Ampliar os campos registrados e a retenção |
| Serviço de acesso remoto com versão vulnerável exposto | Média | Atualizar o serviço; avaliar hunt de exploração |

## 14. Potencial de automação

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | Parcialmente |
| **Ruído permite alerta contínuo?** | Indicadores de peso alto e baixo ruído podem virar detecções: uso de conta de serviço ou de desligado pelo acesso remoto e sucesso após força bruta. Origem nova e horário atípico, isolados, geram ruído demais para alerta contínuo. |
| **Tipos de plataforma alvo** | SIEM |
| **Regra criada** | — |

## 15. Referências

- MITRE ATT&CK — [T1133 External Remote Services](https://attack.mitre.org/techniques/T1133/)
- MITRE ATT&CK — [T1078 Valid Accounts](https://attack.mitre.org/techniques/T1078/)
- MITRE ATT&CK — [T1110 Brute Force](https://attack.mitre.org/techniques/T1110/)
- MITRE ATT&CK — [Initial Access (TA0001)](https://attack.mitre.org/tactics/TA0001/)

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 1.0 | 2026-11-03 | Eduardo Fernandes Mateus | Publicação como Proposto. Sem regra Sigma (ver seção 8.2); não validado em plataforma real. |

---

Estrutura baseada em: OTHF, módulo 6 — Operational e módulo 7 — Measurement. Licença MIT, ver [`referencias/OTHF/`](../../referencias/OTHF/).
