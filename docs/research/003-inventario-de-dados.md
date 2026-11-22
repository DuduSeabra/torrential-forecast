# 003 — Inventário de dados disponíveis para Petrópolis
Data: 2026-11-22 · Status: decidido (ADR 0005 — Opção A)

## Pergunta
Quais fontes de dados existem, cobrem Petrópolis com resolução adequada para 0–3 h, têm histórico
suficiente para treino e são acessíveis para nós?

## Por que importa para Petrópolis
Um modelo só é tão bom quanto os dados que ele vê. Em particular, o alvo (ADR 0003) são os
pluviômetros — precisamos garantir acesso, histórico e qualidade antes de qualquer coisa.

## Fontes analisadas
| Fonte | Tipo | Resolução | Histórico | Acesso | Papel no projeto | Evidência |
|---|---|---|---|---|---|---|
| **CEMADEN — pluviômetros** | Pluviômetro automático | 10 min, ponto | ~2015 em Petrópolis (rede nacional desde 2013) | **PED** ([ped.cemaden.gov.br](https://ped.cemaden.gov.br/)): cadastro com login → token temporário; 12 requisições/min para usuário externo; históricos agendados, CSV por estação, link válido 30 dias | **Alvo + entrada principal** | [Plataforma Clima Ambiente (Fiocruz)](https://plataforma-clima-ambiente-readthedocs.readthedocs.io/en/latest/cemaden_data.html) (lido); 33 estações, 24 ativas (Cereto 2025); 17 estações citadas por Alcântara 2023 |
| **INEA — pluviômetros** | Pluviômetro | horário | desde 1976 (algumas estações) | não verificado | Série longa para climatologia/eventos antigos (ex.: 2011) | Alcântara et al. 2023 |
| **INMET — estações automáticas** | Estação completa (chuva, T, UR, vento, pressão) | horária | variável | portal INMET (público) | Features meteorológicas locais (umidade, pressão) | Alcântara 2023 cita estações A603/A610 (não verificado) |
| **GOES-16 → GOES-19 (ABI)** | Satélite geoestacionário, 16 canais | 2 km, 10 min (full disk) | GOES-16 2017–abr/2025; **GOES-19 é o GOES-East desde 07/04/2025** | gratuito (NOAA, nuvem AWS) | Contexto de nuvens/convecção ao redor | [NOAA NESDIS](https://www.nesdis.noaa.gov/news/noaas-goes-19-now-operational-goes-east-providing-critical-new-data-forecasters) |
| **GOES GLM** | Raios (lightning) | ~8 km, contínuo | 2018– | gratuito (NOAA) | Indicador de convecção intensa ("lightning jump") | literatura em 001 |
| **GOES RRQPE / Hydronn / TUPANN** | Chuva estimada / prevista por satélite | 2 km, 10 min | idem GOES | gratuito / código aberto | Baseline e feature — **não** como alvo | 001; satélites subestimaram 2022 em 90% (Alcântara 2023) |
| **Radar banda X do Morin (Petrópolis)** | Radar dupla polarização | alta | chegou set/2025, montagem out/2025 → histórico ≲ 1 ano | Defesa Civil municipal, sem acesso público conhecido | Futuro (inferência / estudo de caso); **histórico curto demais para treino** | [MPRJ](https://www.mprj.mp.br/web/guest/visualizar?noticiaId=200402); [Correio Petropolitano 2026](https://www.correiodamanha.com.br/petropolitano/2026/09/322980-radar-meteorologico-ganha-destaque-em-audiencia.html) |
| **Radares INEA (Guaratiba)** / DECEA (Pico do Couto) | Radar banda S | — | anos | não verificado; cobertura da serra incerta | Possível no futuro | busca sem resultado conclusivo |
| **ERA5** | Reanálise | 0,25°, horária | 1940– | gratuito (CDS), atraso ~5 dias | Contexto sinótico no treino apenas (não disponível em tempo real) | Curcio 2025 |
| **Atendimentos da Defesa Civil** | Ocorrências de deslizamento | diário | 2015–2024 (11.043) | cedido a pesquisadores (Cereto 2025) | Validação de impacto (opcional, futuro) | Cereto 2025 |

## O que converge
- **CEMADEN é indispensável e acessível**, mas exige cadastro e pedido de histórico (assíncrono).
  Citação obrigatória: "DADOS DA REDE OBSERVACIONAL DO CEMADEN/MCTI".
- **GOES é o único sensor remoto com histórico longo e acesso livre**; a troca GOES-16 → GOES-19
  em abril/2025 exige tratar as duas séries (mesmo instrumento ABI, possível pequena diferença de calibração).
- **Radar hoje não é viável para treino** (Morin: histórico curto; INEA/DECEA: acesso incerto).

## Opções para o projeto
### Opção A — Começar só com CEMADEN (pluviômetros)
EDA, catálogo de eventos, baselines de persistência/climatologia e primeiro modelo tabular só com
a rede de pluviômetros. GOES entra num segundo momento.
- **Prós:** foco, rápido, já responde as perguntas abertas (nº de eventos, qualidade). **Contras:** adia o satélite.

### Opção B — CEMADEN + GOES desde o início
- **Prós:** pipeline completo cedo. **Contras:** GOES é pesado (terabytes no full disk); baixar
  antes de saber quais datas importam é desperdício.

## Recomendação (e por quê)
**Opção A.** A EDA do CEMADEN define o catálogo de eventos, e com ele baixamos GOES só para as
janelas que importam (eventos + amostra de dias normais), economizando disco e tempo.
INEA e INMET entram como complemento se a EDA mostrar lacunas. Em paralelo e sem bloquear:
pedir à Defesa Civil de Petrópolis os limiares do plano de contingência e informações sobre o radar.

## Perguntas em aberto
1. Quantas estações CEMADEN de Petrópolis têm série contínua desde 2015? Taxa de falhas?
2. O PED entrega dados em 10 min ou só horário? (doc não deixa claro)
3. INEA: há acesso público às séries horárias? INMET: quais estações cobrem a serra?
4. Radar do Morin: a Defesa Civil compartilharia os dados para pesquisa?

## Referências (links)
Ver tabela acima e [referencias.bib](referencias.bib).
