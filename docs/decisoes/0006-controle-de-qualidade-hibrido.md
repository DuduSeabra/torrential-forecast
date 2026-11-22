# 0006 — Controle de qualidade híbrido (QC próprio + auditoria com RainfallQC)

Data: 2026-11-22 · Status: aceita · Pesquisa: [004](../research/004-controle-de-qualidade-pluviometros.md) (Opção C)

## Decisão
- **QC oficial do projeto:** implementação própria em `src/petropolis_nowcast/qc.py`, com checagens
  pequenas e documentadas: estrutura (duplicatas, frequência, lacunas) → limites físicos →
  repetições → zeros falsos por vizinhos → picos isolados por vizinhos → revisão manual de todo valor ≥40 mm/h.
- **Auditoria:** RainfallQC (UKCEH, GPLv3) roda em paralelo, com limiares adaptados; discordâncias são investigadas.
  Usado apenas como ferramenta de análise, sem incorporar seu código ao projeto.

## Regras fixas
1. Dados brutos nunca são alterados; o QC adiciona `qc_flag` (motivo) a cada observação.
2. Valores altos nunca são removidos automaticamente — só após revisão com estações vizinhas.
3. Os eventos de 15/02/2022 e 20/03/2022 (e 11/01/2011, se houver dado) são testes obrigatórios:
   o QC não pode marcá-los como inválidos.
4. Limiares calibrados com dados de Petrópolis (EDA), não importados de outros climas.

## Consequências
- As checagens só serão escritas depois que a EDA revelar o formato, a resolução e os erros reais dos dados.
