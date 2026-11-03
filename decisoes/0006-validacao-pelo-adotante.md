# 0006 — Validação dos hunts pelo adotante

- **Status:** Aceita
- **Data:** 2026-11-03

## Contexto

O processo de desenvolvimento prevê duas validações para um hunt ser usado em clientes: com dados de teste (prova de que o hunt encontra a técnica) e histórica (medida do ruído). O projeto não dispõe de laboratório nem de plataformas para executá-las, e publicar hunts como "Em validação" sem que ninguém os teste passaria uma informação falsa.

## Decisão

Este repositório é uma **biblioteca de referência**:

- Os hunts são publicados com o status **Proposto**: completos, com consultas de sintaxe verificada, mas **não validados** em plataforma real pelo projeto.
- A validação é responsabilidade de **quem adota** o hunt, no próprio ambiente, seguindo o processo de desenvolvimento. O status operacional (Em validação, Ativo, Automatizado) é controlado por cada time, fora do repositório.
- Validações feitas por adotantes podem ser contribuídas ao projeto. Um hunt com pelo menos uma validação registrada passa a **Validado pela comunidade**.

## Consequências

- O template e o processo passam a ter dois níveis de status: o da biblioteca e o operacional.
- O README e o índice da biblioteca avisam que os hunts devem ser validados antes do uso em produção.
- O `CONTRIBUTING.md` descreve como contribuir com validações, sem citar produtos nem incluir dados reais.
- Substitui, para a biblioteca pública, a exigência de validação prévia descrita no processo de desenvolvimento; essa exigência continua valendo para o uso operacional dos hunts.
