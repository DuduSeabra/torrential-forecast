# Nowcasting de chuvas torrenciais — Petrópolis/RJ

Objetivo: prever chuva torrencial/destrutiva em Petrópolis com horizonte de 0–3 h (nowcasting) usando IA.

## Como trabalhamos (regras do projeto)
1. **Pesquisa antes de implementar.** Toda técnica, arquitetura, fonte de dados, métrica ou loss nova
   passa pela skill `pesquisa-cientifica` e gera um documento em `docs/research/`.
   Se já houver um documento cobrindo o tema, cite-o.
2. **O usuário decide.** Apresente opções com prós e contras; só implemente depois da decisão.
   Cada decisão vira um ADR em `docs/decisoes/`.
3. **Modo didático.** Explique o porquê de cada passo. Conceitos de meteorologia merecem
   explicação direta (o usuário é forte em ML, novo em meteorologia).
4. **Passos pequenos.** Código curto e legível; um notebook/módulo por vez; nada de gerar o projeto
   inteiro de uma vez.
5. **Baselines primeiro.** Nenhum modelo de deep learning sem comparar contra persistência,
   climatologia e extrapolação.

## Ambiente
- Local: Python 3.12 + `uv`; CPU (GPU AMD RX 470/580 não é suportada pelo ROCm — ver ADR 0002).
  Usar local para download, processamento e EDA.
- Treino: Google Colab / Kaggle (GPU gratuita). Notebooks devem rodar nos dois.
- Dados brutos em `data/` (fora do git).

## Estrutura
- `docs/research/` — relatórios de pesquisa (NNN-tema.md) + `referencias.bib`
- `docs/decisoes/` — ADRs (NNNN-tema.md)
- `notebooks/` — exploração e treino
- `src/petropolis_nowcast/` — código reutilizável

## Fase atual
Ver `README.md` → seção "Roteiro".
