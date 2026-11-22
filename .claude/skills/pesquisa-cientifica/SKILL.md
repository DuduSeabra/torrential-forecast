---
name: pesquisa-cientifica
description: Pesquisa literatura científica recente ANTES de introduzir qualquer técnica, arquitetura, fonte de dados, métrica, loss ou ferramenta nova no projeto de nowcasting de Petrópolis. Use sempre que for propor algo que ainda não tem um documento em docs/research/, ou quando o usuário pedir "pesquise", "o que a literatura diz", "estado da arte". Gera um relatório em docs/research/ e termina pedindo uma decisão ao usuário.
---

# Pesquisa científica antes de implementar

Objetivo: o projeto só adota uma técnica depois de ver o que pesquisadores usam **hoje**,
com referências verificáveis. O usuário quer aprender e decidir, não receber código pronto.

## Quando usar
- Antes de introduzir: arquitetura de modelo, loss, métrica, fonte de dados, pré-processamento,
  estratégia de validação, biblioteca central (ex.: pySTEPS, Earthformer).
- Antes, verifique `docs/research/` — se já existe um documento cobrindo o tema e ele tem < 6 meses,
  reutilize-o e cite-o em vez de pesquisar de novo.

## Processo
1. **Defina a pergunta** em uma frase, conectada a Petrópolis (orografia da Serra do Mar,
   chuva convectiva localizada, eventos raros, sem radar denso, dados CEMADEN/INMET/GOES).
2. **Busque** (WebSearch; envie várias buscas em paralelo):
   - arXiv, periódicos (GMD, ESSD, NHESS, AIES, QJRMS, MWR, JHM, npj Clim Atmos Sci, Nature),
     conferências (NeurIPS/ICLR/ICML climate workshops, AMS), Google Scholar.
   - Prioridade: últimos 2–3 anos. Inclua trabalhos clássicos só como base.
   - Sempre procure também trabalhos brasileiros/regionais: INPE, CEMADEN, IMPA, Cefet/RJ, UFRJ, USP.
3. **Leia** os 4–8 papers mais relevantes (use o agente web-fetch na página arXiv/HTML):
   abstract, método, dados, resultados e limitações. Não conclua com base só no snippet da busca.
   Se não conseguiu ler um paper, marque como "não lido".
4. **Escreva** `docs/research/NNN-<tema-kebab>.md` (NNN = próximo número sequencial) usando o modelo abaixo.
5. **Atualize** `docs/research/referencias.bib` com as entradas BibTeX (chave `sobrenomeAnoPalavra`).
6. **Apresente** ao usuário um resumo curto (5–10 linhas) + as opções e **pergunte qual decisão tomar**.
   Não implemente nada antes da resposta.
7. Após a decisão, crie `docs/decisoes/NNNN-<tema>.md` (ADR) citando o documento de pesquisa.

## Modelo do documento de pesquisa
```markdown
# NNN — <Tema>
Data: AAAA-MM-DD · Status: aguardando decisão | decidido (ADR NNNN)

## Pergunta
## Por que importa para Petrópolis
## Papers analisados
| Ref | Ano | Venue | Dados | Método | Resultado principal | Limitações | Relevância (alta/média/baixa) | Lido? |
## O que a literatura converge / onde diverge
## Opções para o projeto
### Opção A — ... (prós, contras, custo computacional no Colab/Kaggle grátis)
### Opção B — ...
## Recomendação (e por quê)
## Perguntas em aberto
## Referências (links)
```

## Regras
- Nunca invente referência, número ou link. Todo resultado citado tem link.
- Diferencie claramente "o paper mostra" de "eu infiro".
- Explique conceitos novos em linguagem direta — o usuário é intermediário/avançado em ML,
  mas novo em meteorologia.
- Considere sempre a restrição de hardware: treino em Colab/Kaggle grátis (GPU T4/P100, ~16 GB).
