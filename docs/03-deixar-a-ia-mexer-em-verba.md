# Deixar a IA mexer em verba — onde ela ajuda e onde ela custa caro

Este documento existe porque a pergunta certa não é *"a IA consegue operar Google Ads?"*. Consegue.
A pergunta é **o que você delega e o que continua sendo seu** — e essa fronteira, aqui, tem preço.

---

## A diferença entre errar num relatório e errar numa campanha

| Erro | Como você descobre | Quanto custa |
|---|---|---|
| A IA lê um número errado | você confere e ri | nada |
| A IA escreve um anúncio ruim | reprovação ou CTR baixo | dias |
| A IA ativa uma campanha sozinha | **pela fatura** | o orçamento diário inteiro, vezes os dias até alguém olhar |
| A IA negativa demais | o volume some | uma semana de tráfego bom, e ninguém liga uma coisa à outra |

As duas últimas linhas são a razão de existirem as travas do `AGENTS.md`. Não é desconfiança do
modelo: é que o **feedback demora**. Erro que aparece na hora se conserta; erro que aparece na
fatura já custou.

---

## As três travas, e o porquê de cada uma

**1. Tudo nasce `PAUSED`.**
Objeto pausado não gasta. A trava transforma "a IA criou algo errado" — que é irrecuperável — em
"a IA criou algo errado e eu vi antes de ligar" — que é uma conversa. Custa um clique.

**2. Escrita só depois do "pode", com o orçamento em reais.**
O parâmetro da API é em centavos (`--budget 5000` = R$ 50/dia). Confirmação que mostra `5000` não é
confirmação — o agente é obrigado a te dizer *"cinquenta reais por dia"*, na sua língua.

**3. Uma conta por vez.**
Se você atende clientes, o `contas.yaml` tem vários. "Roda em todas para adiantar" é o jeito mais
rápido de aplicar a decisão de um cliente na conta de outro.

---

## O que delegar sem medo

Tudo que é **leitura e leitura pesada**:

- ler 400 termos de pesquisa e classificar por intenção
- cruzar dia da semana com dispositivo com hora
- comparar dois períodos e vir com hipótese
- ler o índice de qualidade decomposto e dizer se o problema é o anúncio, a página ou o CTR
- escrever a consulta GAQL que você não sabia que era possível

Aqui a IA é melhor do que você — não porque pensa melhor, mas porque **não cansa na linha 200**, e
é exatamente na linha 200 que mora o desperdício.

## O que não delegar

- **Quanto você pode perder testando.** É decisão de caixa, não de otimização.
- **Ligar campanha.**
- **Qual promessa o anúncio faz.** A IA escreve bem; o que você pode prometer sem se queimar é você
  que sabe.
- **Quando parar.** "Está ruim, mas é sazonalidade" é contexto que não está na API.

---

## Duas frases que denunciam análise fraca

Peça ao agente que evite as duas, e cobre quando aparecerem:

> ❌ *"a campanha está performando bem"*
> ✅ *"nos últimos 14 dias: 23 conversões, CPA de R$ 41, contra R$ 58 no período anterior"*

> ❌ *"recomendo pausar essa palavra-chave"*
> ✅ *"essa palavra gastou R$ 180 em 30 dias, 41 cliques, zero conversão — o CPA da conta é R$ 41,
> então ela já custou o equivalente a 4 vendas. Pausar?"*

A segunda forma é a que deixa **você** decidir. A primeira é a que faz você aceitar sem pensar — e
é onde a automação começa a errar em silêncio.

---

## O teste que vale mais que qualquer regra

Antes de deixar rodar sozinho, faça o agente operar **na sua frente** por duas semanas. Não é
desconfiança: é como você descobre o que ele entende do **seu** negócio e o que ele ainda não sabe
— o cliente que não vale a pena, o termo que parece ruim e converte, a época em que tudo cai e não
é problema.

Esse contexto não está na API. Ele entra escrevendo, aqui nos `docs/` e no `AGENTS.md` desta pasta.
**Pasta que ensina o negócio ao agente vale mais do que qualquer script.**
