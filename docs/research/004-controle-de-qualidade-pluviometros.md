# 004 — Controle de qualidade (QC) de pluviômetros automáticos
Data: 2026-11-22 · Status: aguardando decisão

## Pergunta
Como detectar e tratar erros nos dados sub-horários dos pluviômetros CEMADEN de Petrópolis sem
apagar os extremos reais — que são exatamente o que queremos prever?

## Por que importa para Petrópolis
- Um pico espúrio de 80 mm em 10 min vira um "evento extremo" falso no treino e na avaliação.
- Um pluviômetro entupido registra **zero** durante uma tempestade real ("zero falso") — o modelo
  aprende que aquele padrão não gera chuva.
- Mas o oposto também é perigoso: regras de QC de climas temperados cortariam eventos legítimos
  de Petrópolis (2011: 61,8 mm/h; 2022: ~258 mm/3 h).

### Conceitos rápidos
- **Pluviômetro de báscula (tipping bucket):** um "balde" de volume fixo vira a cada ~0,2 mm;
  conta-se os tombos. Erros típicos: entupimento (zeros falsos), **subestimação em chuva muito intensa**
  (água escapa durante o tombo; relatada acima de ~50–100 mm/h, [ver revisão](https://encyclopedia.pub/entry/history/compare_revision/103622)),
  tombos rápidos espúrios, falhas de transmissão/energia (lacunas).
- **Flag vs remoção:** boas práticas marcam (flag) a observação suspeita com o motivo, em vez de
  apagá-la; a decisão de excluir fica explícita e reversível.
- **Checagem por vizinhos:** compara a estação com as próximas — base para detectar zeros falsos
  e picos isolados.

## Fontes analisadas
| Ref | Ano | Venue | O que traz | Relevância | Lido? |
|---|---|---|---|---|---|
| [Villalobos-Herrera et al.](https://pmc.ncbi.nlm.nih.gov/articles/PMC9826504/) | 2022 | QJRMS | QC sub-horário (SHQC) em 1.301 pluviômetros do Reino Unido, estendendo o GSDR-QC horário (25 testes). Limiares mensais de 30–40 mm/1 h e 13–20 mm/15 min; teste de tombos rápidos (moda do intervalo entre tombos < 2 s); remove eventos suspeitos em janela de 3 h. QC sub-horário muda significativamente os níveis de retorno de extremos curtos. Validação: ~19% falsos positivos em eventos >40 mm/h. Código: [intense-qc](https://github.com/nclwater/intense-qc), [SubHourlyQC](https://github.com/nclwater/SubHourlyQC) | **alta** (método) / **baixa** (limiares) | parcial |
| [RainfallQC (UKCEH)](https://pypi.org/project/rainfallqc/) | 2026 | Pacote Python | 33 checagens (25 IntenseQC + 6 SubHourlyQC + 2 pyPWSQC): percentis, k-maiores, dias/horas suspeitos, intermitência, streaks (valores repetidos), acumulações, vizinhos (úmido/seco, offset de tempo, correlação), zeros falsos, outlier de estação. Suporta <15 min, 15 min, horário. Limiares e distâncias editáveis. **GPLv3**, polars, v1.1.2 (out/2026), ativo | **alta** | sim (página) |
| [El Hachem et al.](https://hess.copernicus.org/articles/28/4715/2024/) | 2024 | HESS | Compara PWSQC (vizinhos: zeros falsos, "high influx", outlier de estação, correção de viés), PWS-pyQC (exige rede de referência; kriging) e GSDR-QC (conservador, rápido, mais retenção, mais fraco em zeros falsos). Conclusão: todos melhoram os dados; não há vencedor universal — depende do contexto | média | parcial |
| [Ostrowski et al. RainGaugeQC](https://amt.copernicus.org/articles/15/5581/2022/) | 2022 | AMT | QC em tempo real sub-horário com índice de qualidade por medida, usando radar | baixa (precisa radar) | só abstract |
| Alcântara et al. 2023 (pesquisa 002) | 2023 | NHESS | Evento 2022 confirmado por múltiplos pluviômetros (São Sebastião 258 mm; Dr. Thouzet 221 mm/24 h) — serve de **teste de sanidade**: um bom QC não pode marcar esse evento | alta | parcial |

Não foi encontrado estudo de QC específico da rede CEMADEN.

## O que a literatura converge / onde diverge
**Converge:** (1) marcar com flags, não apagar; (2) combinar checagens de uma estação (limites
físicos, repetições, lacunas, frequência de registro) com checagens por vizinhos; (3) QC sub-horário
encontra erros invisíveis no dado horário; (4) a maioria dos erros é de equipamento (entupimento,
tombos espúrios, transmissão).

**Diverge / atenção:** limiares absolutos são **climáticos**. Os do Reino Unido (30–40 mm/h)
removeriam eventos reais de Petrópolis. Em clima tropical serrano, extremos devem ser validados
por **coerência com vizinhos** (estações próximas também registraram chuva forte?), não por um teto fixo.

## Opções para o projeto
### Opção A — Usar RainfallQC com limiares adaptados
- **Prós:** 33 checagens testadas e mantidas; economiza tempo. **Contras:** caixa-preta para o
  aprendizado; limiares e regras pensados para o Reino Unido; GPLv3 (ok para projeto pessoal,
  contamina a licença se distribuir código derivado).

### Opção B — Implementar um QC próprio, pequeno e explicado
Checagens em ordem: (1) estrutura (timestamps duplicados/fora de ordem, frequência de registro,
lacunas); (2) limites físicos (negativos; acima de recorde mundial); (3) repetições (mesmo valor
>0 repetido por muito tempo); (4) zeros falsos por vizinhos (estação seca enquanto a mediana dos
vizinhos chove forte); (5) picos isolados por vizinhos (valor alto sem nenhum vizinho com chuva);
(6) revisão manual de todo valor ≥40 mm/h, com gráfico da estação + vizinhos.
- **Prós:** você entende cada regra; limiares calibrados com dados de Petrópolis. **Contras:** mais trabalho; risco de esquecer casos.

### Opção C — Híbrido
B como QC oficial do projeto; RainfallQC rodado em paralelo como **auditoria**: onde discordarem,
investigar.
- **Prós:** aprendizado + rede de segurança. **Contras:** um pouco mais de tempo.

## Recomendação (e por quê)
**Opção C.** O QC define o que é "evento extremo" — é decisão central demais para ficar numa
caixa-preta, mas a ferramenta madura ajuda a pegar o que esquecemos. Regras fixas do projeto:
- **Nunca remover automaticamente valores altos**: extremos só são invalidados após revisão com vizinhos.
- Eventos conhecidos (11/01/2011 se houver dado, 15/02/2022, 20/03/2022) são testes obrigatórios:
  o QC não pode marcá-los.
- Toda observação ganha uma coluna `qc_flag` com o motivo; os dados brutos nunca são alterados.

## Perguntas em aberto
1. Qual o volume do tombo (resolução) dos pluviômetros CEMADEN? (0,2 mm? ver nos dados)
2. O CEMADEN já entrega os dados com algum QC aplicado?
3. Distância típica entre estações em Petrópolis — vizinhos a ~2–5 km bastam para as checagens?

## Referências (links)
Ver tabela acima e [referencias.bib](referencias.bib).
