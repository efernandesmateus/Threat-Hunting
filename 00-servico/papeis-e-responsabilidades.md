# Papéis e responsabilidades

## 1. Papéis no SOC

A hierarquia segue a proposta do OTHF, adaptada a um time que atende vários clientes. Em times pequenos, uma pessoa pode acumular papéis, desde que as responsabilidades continuem cobertas.

### Líder de Hunting

Responsável pelo serviço como um todo.

- Define a estratégia, os objetivos e a evolução da metodologia.
- Aprova novos hunts na biblioteca e mudanças de processo.
- Prioriza o backlog entre os clientes e distribui o trabalho.
- Acompanha as métricas do serviço e reporta à gestão.
- Faz mentoria do time e garante a qualidade das entregas.
- É o ponto de escalonamento interno para achados críticos.

### Hunter Sênior

Executa os hunts mais complexos e desenvolve a biblioteca.

- Desenvolve e valida novos hunts, incluindo hunts personalizados para clientes.
- Executa hunts orientados por inteligência e hunts extraordinários.
- Revisa hunts e relatórios produzidos por outros analistas.
- Conduz a investigação inicial de achados suspeitos e maliciosos.
- Faz mentoria dos Hunters.

### Hunter

Executa os hunts da biblioteca.

- Executa os hunts planejados para cada cliente, seguindo os procedimentos.
- Faz a triagem dos resultados conforme os critérios de cada hunt.
- Documenta as execuções e escala achados ao Hunter Sênior ou ao Líder.
- Propõe melhorias em hunts existentes (falsos positivos, consultas, baseline).

### Competências esperadas

| Competência | Hunter | Hunter Sênior | Líder de Hunting |
|-------------|:------:|:-------------:|:----------------:|
| Uso das plataformas de busca (SIEM, EDR) | Intermediário | Avançado | Avançado |
| Conhecimento de MITRE ATT&CK | Intermediário | Avançado | Avançado |
| Sistemas operacionais e redes | Intermediário | Avançado | Avançado |
| Técnicas de análise (stacking, enriquecimento, agrupamento) | Intermediário | Avançado | Avançado |
| Uso de inteligência de ameaças | Básico | Avançado | Avançado |
| Conhecimento ofensivo (como o adversário age) | Básico | Intermediário | Avançado |
| Desenvolvimento de detecções | Básico | Avançado | Avançado |
| Comunicação com o cliente | Básico | Intermediário | Avançado |
| Gestão e priorização | — | Básico | Avançado |

O perfil comum aos três papéis é a **mentalidade investigativa**: curiosidade, atenção a detalhes e o hábito de perguntar "isso é normal?". Só se reconhece o anômalo quem conhece o normal.

### Interfaces internas

O serviço depende de outras áreas do provedor:

| Área | Interface com o hunting |
|------|-------------------------|
| **Monitoramento (SOC)** | Recebe detecções derivadas dos hunts; fornece contexto de alertas recentes que podem originar hipóteses. |
| **Resposta a incidentes** | Recebe os achados maliciosos escalados. |
| **Inteligência de ameaças** | Fornece campanhas, TTPs e IOCs que originam hunts orientados por inteligência. |
| **Engenharia de detecção** | Converte hunts maduros em regras, quando essa área existir separada do hunting. |
| **Gestão de contas** | Comunica escopo, mudanças de contrato e expectativas do cliente. |

## 2. Papéis no cliente

| Papel | Responsabilidade |
|-------|------------------|
| **Ponto focal técnico** | Responde dúvidas sobre o ambiente, confirma se comportamentos são esperados, apoia o onboarding. |
| **Responsável por decisões em incidentes** | Decide sobre contenção e remediação; contato para escalonamentos críticos, com disponibilidade combinada. |
| **Patrocinador** | Recebe os relatórios executivos e as recomendações estratégicas (geralmente gestor de segurança ou TI). |

## 3. Matriz RACI — SOC × cliente

**R** = executa · **A** = responde pelo resultado (um por linha) · **C** = consultado · **I** = informado

| Atividade | Líder de Hunting | Hunter Sênior | Hunter | Cliente |
|-----------|:----------------:|:-------------:|:------:|:-------:|
| Onboarding e inventário de fontes de dados | A | R | C | C |
| Manter a coleta e a retenção das fontes de dados | I | I | — | A/R |
| Conceder e manter acessos às plataformas | I | I | — | A/R |
| Priorizar os hunts do cliente | A | R | C | C |
| Desenvolver e validar hunts da biblioteca | A | R | C | — |
| Desenvolver hunts personalizados (Estratégico) | A | R | C | C |
| Executar hunts da biblioteca | I | A | R | — |
| Executar hunts orientados por inteligência | I | A/R | C | — |
| Triagem dos resultados | I | A | R | C |
| Esclarecer comportamentos do ambiente | — | C | C | A/R |
| Confirmar achado malicioso | A | R | C | I |
| Notificar o cliente sobre achado malicioso | A | R | — | I |
| Contenção — modo **somente notificação** | I | C | — | A/R |
| Contenção — modo **contenção pelo SOC** | A | R | — | C |
| Remediação | I | C | — | A/R |
| Recomendar novas detecções | A | R | C | I |
| Implementar detecções | A | R | C | C |
| Elaborar o relatório periódico | A | R | R | — |
| Apresentar resultados (reuniões) | A/R | C | — | I |

Notas:

- **Implementar detecções** só é responsabilidade do SOC no nível Estratégico e quando houver acesso administrativo às plataformas. Nos outros casos, o cliente é **A/R** e o SOC é **C**.
- No modo **contenção pelo SOC**, o cliente continua **C** para ações fora da lista pré-autorizada, que exigem a sua aprovação.

---

Baseado em: OTHF, módulo 3 — Structure (papéis, perfil e comunicação). Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
