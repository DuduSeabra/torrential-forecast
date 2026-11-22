# Nowcasting de chuvas torrenciais em Petrópolis/RJ

Previsão de curtíssimo prazo (0–3 h) de chuvas destrutivas em Petrópolis usando IA,
construída passo a passo e fundamentada em literatura científica recente.

## Roteiro
- [x] **Fase 0 — Fundação:** estrutura, skill de pesquisa, ambiente.
- [ ] **Fase 1 — Estado da arte:** `docs/research/001-estado-da-arte-nowcasting.md` → ADR de direção.
- [ ] **Fase 2 — Entender o problema:** definição de "chuva destrutiva", inventário de dados, EDA e catálogo de eventos.
- [ ] **Fase 3 — Baselines e avaliação:** split temporal, persistência/climatologia/extrapolação, métricas para extremos.
- [ ] **Fase 4 — Modelo de deep learning:** escolhido com base nas fases anteriores.

## Setup
```bash
uv sync            # cria .venv com as dependências
uv run jupyter lab
```

## Documentação
- [docs/research/](docs/research/) — pesquisas que fundamentam cada escolha
- [docs/decisoes/](docs/decisoes/) — decisões tomadas (ADRs)
