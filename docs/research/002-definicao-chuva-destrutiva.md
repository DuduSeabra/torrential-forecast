# 002 — O que é "chuva destrutiva" em Petrópolis (definição do alvo)
Data: 2026-11-22 · Status: decidido (ADR 0004 — Opção A)

## Pergunta
Quais limiares de intensidade/acumulado definem, de forma operacional e defensável, a chuva que
queremos prever (ADR 0003: probabilidade de exceder limiares nos pluviômetros)?

## Por que importa para Petrópolis
O limiar define o alvo do modelo. Alto demais → quase nenhum exemplo positivo para treinar.
Baixo demais → o modelo aprende a prever "chuva comum", não o que destrói. Ele também precisa
conversar com quem usa o alerta (Defesa Civil).

### Conceitos rápidos
- **Intensidade × acumulado:** deslizamentos dependem tanto da chuva intensa de curta duração
  (mm/h) quanto do solo já encharcado (acumulado de 24–96 h). Inundação urbana responde rápido
  à intensidade.
- **Limiar intensidade-duração (I-D):** curva empírica acima da qual eventos de deslizamento
  costumam ocorrer — método clássico de sistemas de alerta (LEWS).

## Fontes analisadas
| Ref | Ano | Tipo | O que traz | Lido? |
|---|---|---|---|---|
| [Alcântara et al., NHESS](https://nhess.copernicus.org/articles/23/1157/2023/) | 2023 | Artigo | Evento 15/02/2022: **258 mm em ~3 h** nos pluviômetros; causado por sistema convectivo de mesoescala com influência orográfica; previsão da véspera indicava só "chuva convectiva isolada"; CHIRPS/MERGE/IMERG **subestimaram o acumulado em 90%**; recorde anterior (jan/2011): 241,8 mm/24 h, **pico de 61,8 mm/1 h**; 20/03/2022: 415 mm em 10 h (217,4 mm em 4 h); CEMADEN tem dados em Petrópolis desde 2015 | parcial (40% verbatim) |
| [Plano de contingência DRM-RJ 2016/17](https://www.rj.gov.br/drm/sites/default/files/arquivos_paginas/pc_2016_2017_definitivo-otimizado_1.pdf) | 2016 | Doc. técnico | Índices críticos para a **Região Serrana: 40 mm/h, 100 mm/24 h, 115 mm/96 h, 270 mm/30 dias** | só snippet |
| [Metrópoles/IstoÉ — monitoramento](https://istoe.com.br/como-e-feito-o-monitoramento-que-emite-alertas-de-chuvas/) | — | Notícia | Em Petrópolis, **10 mm em 1 h** em pluviômetros específicos → Defesa Civil já espera transbordamento de rio na área urbana | só snippet |
| [Cereto et al., SINAGEO](https://www.editorarealize.com.br/editora/anais/sinageo/2025/TRABALHO_COMPLETO_EV217_ID1537_TB609_26072025091233.pdf) | 2025 | Artigo | Percentil 95 do acumulado diário na cidade = **36 mm/dia**; 11.043 atendimentos da Defesa Civil (2015–2024) relacionados a deslizamentos | sim |
| [Baião (UNESP), relatório pós-doc](https://repositorio.unesp.br/entities/publication/c571446b-ddba-4e42-b71b-f97c7feccf4e/full) | 2026 | Relatório | Limiares de deslizamento em Petrópolis com LR e MGWR (1985–2022); AUC 0,77–0,99. Valores dos limiares não acessíveis (PDF com embargo) | só metadados |
| Rosi et al., Geosciences | 2019 | Artigo | Limiares I-D (MaCumBa) com 3 níveis de alerta por município da Região Serrana | **não lido** (403) |
| Plano de Contingência de Petrópolis | — | Doc. oficial | Sirenes têm 2 níveis (chuva forte / risco de deslizamentos generalizados); limiares numéricos não encontrados publicamente | **não lido** |

## O que as fontes convergem / onde divergem
- **40 mm/h** aparece como índice crítico regional (DRM-RJ) e também em Niterói (≥40 mm/h).
  Eventos catastróficos de Petrópolis ultrapassam isso com folga (2011: 61,8 mm/h; 2022: ~258 mm/3 h).
- Limiares oficiais combinam **curto prazo (1 h)** e **acumulado (24–96 h)**. Nowcasting de 0–3 h
  só consegue prever a parte de curto prazo; o acumulado anterior é *observado* e pode entrar como
  feature (contexto de solo encharcado).
- Não há limiar público e atual da Defesa Civil de Petrópolis; os valores operacionais provavelmente
  existem no plano de contingência municipal.

## Opções para o projeto
### Opção A — Limiares fixos em múltiplos níveis (acumulado de 1 h)
Prever P(chuva em 1 h ≥ L) para **L = 10, 20, 40 mm** em cada pluviômetro (+ ≥60 mm só para avaliação de casos).
- 10 mm/h: impacto urbano (transbordamento) — muitos exemplos, aprendível.
- 20 mm/h: chuva forte — intermediário.
- 40 mm/h: índice crítico DRM-RJ — raro, o que realmente importa.
- **Prós:** interpretável, conectado a referências oficiais, níveis crescentes servem de "escada"
  para lidar com desbalanceamento. **Contras:** 40 mm/h pode ter poucos casos por estação.

### Opção B — Limiares por percentil local (ex.: p99, p99.9 de cada estação)
- **Prós:** adapta-se à climatologia de cada ponto. **Contras:** não tem significado operacional
  direto; difícil de explicar para a Defesa Civil.

### Opção C — Alvo contínuo (regressão de mm/h) e limiares só na avaliação
- **Prós:** usa toda a informação. **Contras:** regressão com MSE subestima extremos
  (ver pesquisa 001); probabilidades de exceder limiar são mais úteis para alerta.

## Recomendação (e por quê)
**Opção A**, com os números finais confirmados pela EDA (quantos eventos ≥40 mm/h existem).
Acumulados de 24/72/96 h entram como **features observadas**, não como alvo. Além da janela de 1 h,
avaliar também acumulado móvel de 3 h (onde o evento de 2022 se destaca), se a EDA mostrar que
ajuda. Pedir o Plano de Contingência à Defesa Civil de Petrópolis para alinhar limiares.

## Perguntas em aberto
1. Quantos eventos ≥20 e ≥40 mm/h existem por estação CEMADEN em Petrópolis (2015–2026)? → EDA.
2. Quais limiares a Defesa Civil de Petrópolis usa para acionar sirenes? (pedir via LAI/e-mail)
3. Janela de 1 h móvel (a cada 10 min) ou hora cheia? → decidir na EDA (móvel captura picos melhor).

## Referências (links)
Ver tabela acima e [referencias.bib](referencias.bib).
