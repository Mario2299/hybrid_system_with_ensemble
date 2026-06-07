# Perguntas Prováveis do Professor — Defesa do Projeto de Ensemble

> **Disciplina:** Reconhecimento de Padrões
> **Projeto:** Sistema Híbrido de Omni-Ensemble com Seleção Multiobjetivo para Estimação de Esforço de Software (SEE)

---

## 1. GERAÇÃO — Como os modelos foram criados?

---

**❓ Como vocês garantiram diversidade entre os modelos do pool?**

A diversidade foi garantida por **diversidade de arquitetura** — escolhemos modelos de paradigmas completamente diferentes: LR (linear), SVR (kernel), MLP (rede neural), kNN (lazy learning) e DT (árvore). Cada um tem um viés indutivo diferente, o que naturalmente gera predições diversas.

---

**❓ Como os modelos foram treinados? Usaram bagging, boosting?**

Não usamos bagging nem boosting. Cada modelo foi treinado diretamente no conjunto de treino (70%) com predições **out-of-fold** para evitar vazamento de dados na etapa de seleção.

---

**❓ Por que vocês não usaram Random Forest ou XGBoost?**

Justamente porque Random Forest e XGBoost já são ensembles por dentro. Usá-los como componentes de outro ensemble seria redundante e quebraria a ideia de modelos base simples e diversos — o que vai contra a teoria de ensembles que exige modelos **precisos e diversos** individualmente.

---

## 2. SELEÇÃO — Quais modelos usar?

---

**❓ Qual a diferença entre a seleção estática e dinâmica de vocês?**

Implementamos as duas abordagens:

- **SES-GA** e **SES-GA-Multi** → seleção **estática**: escolhem um subconjunto fixo de modelos antes de ver os dados de teste
- **DES** → seleção **dinâmica**: para cada nova instância de teste, seleciona os modelos mais competentes na vizinhança local daquele ponto

---

**❓ O que exatamente é a contribuição de vocês na seleção?**

A contribuição central é substituir o algoritmo genético **mono-objetivo** do artigo original (que otimizava só R²) por um **NSGA-II multiobjetivo** que otimiza simultaneamente:

- **Precisão** (R²)
- **Parcimônia** (usar menos modelos)

Isso gera uma **frente de Pareto** de soluções ao invés de uma única resposta.

---

**❓ Por que parcimônia é um objetivo relevante?**

Usar menos modelos reduz custo computacional e pode reduzir overfitting. A ideia é que um ensemble menor e bem selecionado pode ser tão bom quanto um maior — e mais interpretável.

---

## 3. COMBINAÇÃO — Como os resultados são unidos?

---

**❓ Como vocês combinam as predições? A combinação é treinável?**

Usamos **média aritmética simples** — não é treinável. Essa é uma limitação explícita do trabalho. A combinação ponderada por competência estava planejada mas não foi implementada, sendo listada como trabalho futuro em `combination.py`.

---

**❓ *(Pegadinha)* Faz sentido usar média simples com modelos tão diferentes entre si?**

É uma limitação real. O ideal seria uma combinação ponderada, por exemplo dando mais peso aos modelos com melhor desempenho local. A média simples pode diluir o poder de modelos mais fortes com modelos mais fracos — e não aproveita a seleção inteligente feita pelo NSGA-II.

---

## 4. JUSTIFICATIVA TEÓRICA

---

**❓ Com base na teoria, faz sentido o que vocês fizeram?**

Sim, parcialmente. A teoria de ensembles diz que combinar modelos **precisos e diversos** tende a superar modelos individuais. Nossa geração respeita isso. A seleção multiobjetivo tem embasamento em otimização — a frente de Pareto é uma abordagem mais rica que um único ótimo.

**Porém**, a combinação por média simples enfraquece a teoria, pois não aproveita a competência diferenciada de cada modelo.

---

## 5. RESULTADOS E TESTES ESTATÍSTICOS

---

**❓ O resultado foi negativo — o que explica isso?**

Três fatores principais:

1. **Poucas bases de dados** — 4 no lugar de 5 (a base China estava ausente, justamente a maior)
2. **Bases pequenas** — pouco poder estatístico para o teste de Friedman
3. **Combinação simples** — a média não aproveita a seleção inteligente feita pelo NSGA-II

---

**❓ Por que usaram Friedman e não só Wilcoxon?**

O Wilcoxon compara dois métodos por vez. Fazer múltiplas comparações assim infla o erro de Tipo I. O **Friedman controla o erro familiar** de comparações múltiplas, sendo mais rigoroso — conforme recomendado por Demšar (2006), referência padrão na área. O post-hoc de Bonferroni-Dunn complementa identificando quais pares diferem significativamente.

---

**❓ O sklearn foi usado para tudo?**

Não. Os **regressores base** vêm do sklearn, mas os seguintes foram **implementados do zero** em NumPy/SciPy:

- Algoritmo genético (SES-GA)
- NSGA-II (não-dominância + crowding distance)
- DES por competência local
- Combinação de predições
- Testes de Friedman e Bonferroni-Dunn

---

## 6. PERGUNTA MAIS DIFÍCIL

---

**❓ Se o resultado é nulo, qual o valor real do trabalho?**

O valor é **metodológico**:

- Formalizamos a seleção de modelos como um problema multiobjetivo
- Implementamos um NSGA-II próprio (não de biblioteca)
- Usamos validação estatística mais rigorosa que o artigo original
- Identificamos claramente as limitações e trabalhos futuros

O resultado nulo com 4 bases pequenas **não invalida a abordagem** — invalida a conclusão com esses dados. Com a base China incluída e combinação adaptativa implementada, os resultados poderiam ser diferentes.

---

## Referências Citadas

| Referência | Relevância |
|---|---|
| Jadhav et al., IEEE Access, 2023 | Artigo base estendido |
| Demšar, JMLR, 2006 | Justifica uso do Friedman |
| Deb et al., IEEE TEC, 2002 | Base teórica do NSGA-II |
| Britto et al., Pattern Recognition, 2014 | Base teórica do DES |

---

> 💡 **Dica para a defesa:** o professor citou o journal *Information Fusion* como referência de qualidade na área de ensemble. Se possível, mencionar que o trabalho se alinha metodologicamente com trabalhos publicados nesse journal pode agregar valor à apresentação.
