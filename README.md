# Seu Google Ads, operado pela IA — sem MCP, pela API oficial

Esta pasta liga o seu agente de IA direto na **API oficial do Google Ads**. Ele lê a conta,
monta relatório, planeja palavra-chave, cria campanha, escreve anúncio e negativa termo —
conversando com você em português.

Você não vai decorar comando. Abra o **Claude Code dentro desta pasta** e peça o que quer:
ela já sabe se explicar para ele.

> *"me mostra quanto cada campanha gastou nos últimos 30 dias"*
> *"quais termos de pesquisa gastaram e não converteram?"*
> *"monta uma campanha de busca para 'conserto de notebook', orçamento de R$ 30 por dia"*

## Por que não é MCP

O Google publicou um MCP oficial, e ele é bom — mas **só lê**. Não muda lance, não pausa campanha,
não cria anúncio. Para análise, resolve. Aqui a proposta é outra: **operar**. Isso só existe pela
API, e é o que esta pasta faz.

De quebra, você fica com a lógica na mão: a regra que decide negativar um termo é um arquivo seu,
não uma caixa-preta de uma ferramenta que você aluga.

## O que você precisa ter

| | O que | Detalhe |
|---|---|---|
| 1 | Uma **conta do Google Ads** com faturamento configurado | é onde as campanhas rodam |
| 2 | Uma **conta administradora (MCC)** | *opcional* — só quem opera **várias** contas pela mesma configuração |
| 3 | ~~Um **developer token**~~ | ❌ **não existe mais** — o Google desligou em 09/set/2026 |
| 4 | Um **projeto no Google Cloud** com a Google Ads API ativada **e inscrito** | de graça — e é **ele** que define o seu nível de acesso |
| 5 | **Python 3** e `pip` | `pip3 install -r requirements.txt` |
| 6 | **Claude Code**. Codex e Antigravity funcionam igual | é quem conversa e executa |

⚠️ **Você não precisa esperar dias, e não precisa de token nenhum.** Em 09/set/2026 o Google
desligou o developer token: o acesso passou a ser do **projeto no Google Cloud**. Tutorial que
manda pedir token na Central de API é anterior a essa data, e o pedido lá **não é mais processado**.
O **Explorer** sai na própria inscrição do projeto e já alcança **conta de produção** — a única
coisa que ele não alcança é a pesquisa de palavra-chave. Qual pedir:
[manual de conexão](docs/00-manual-de-conexao.md) §4 · o degrau do Basic:
[docs/06-subir-para-basic.md](docs/06-subir-para-basic.md).

⚠️ **E talvez você não precise disto.** Se você só quer relatório bonito e alerta pronto, existe
ferramenta paga que entrega isso hoje, sem configurar nada. Isto aqui compensa quando **a regra
precisa ser sua**: o seu jeito de negativar, o seu critério de pausar, o seu nicho.

## A ordem de leitura

1. **[COMECE-AQUI.md](COMECE-AQUI.md)** — a configuração, na ordem, uma vez só.
2. **[docs/00-manual-de-conexao.md](docs/00-manual-de-conexao.md)** — o passo a passo completo, do
   zero, com o catálogo de erros. É para onde o COMECE-AQUI aponta quando você travar.
3. `docs/` — o que a API permite, GAQL, e a estratégia (anúncio, campanha, negativação).
4. `AGENTS.md` — não é para você: é o que o seu agente lê.

## O que tem aqui dentro

```
.env.exemplo            copie para .env e preencha as duas credenciais do OAuth
contas.yaml.exemplo     copie para contas.yaml — apelido → conta, para o agente resolver sozinho
requirements.txt        as três bibliotecas
scripts/
  setup.py              check · oauth · test · full   ← comece por aqui
  setup_perfil.py       define o seu tipo de negócio (local × online)
  read.py               campanhas, grupos, palavras, anúncios, termos, qualidade
  insights.py           métricas: conta, campanha, dia, dispositivo, hora
  create.py             campanha, grupo, palavra, anúncio, extensão, negativa
  update.py             status, orçamento, lance
  delete.py             remover palavra, negativa, anúncio, campanha
  keyword_planner.py    volume de busca, CPC e concorrência
  create_skag.py        monta uma campanha inteira, uma palavra por grupo
AGENTS.md / CLAUDE.md   o manual do seu agente de IA
docs/                   o porquê, o GAQL e a estratégia
```

## Três coisas que valem mais que o resto

**1. Tudo nasce pausado.** Campanha, grupo e anúncio criados pelos scripts começam `PAUSED`, de
propósito. Quem liga é você, olhando. Um agente prestativo que ativa sozinho gasta dinheiro de
verdade enquanto você toma café.

**2. O orçamento é em centavos.** `--budget 5000` é **R$ 50 por dia**. Digitar `50` achando que são
cinquenta reais cria uma campanha de cinquenta centavos — e você passa a tarde achando que a conta
está com problema de entrega.

**3. Se você deixar o app OAuth em "Testing", tudo para em 7 dias.** Funciona hoje, funciona amanhã,
e morre na semana que vem com um erro que não diz o que aconteceu. É o defeito mais comum de
automação de Google Ads e tem uma seção só para ele:
[§9 do manual](docs/00-manual-de-conexao.md).

## Se algo der errado

O `AGENTS.md` tem uma tabela de *"o que o erro diz × o que ele significa"* — nesta API, quase todo
erro aponta para o lugar errado. Peça ao agente:

> *"deu esse erro aqui, o que é de verdade?"*
