# Instruções do agente — kit de Google Ads

## Regras que não se pulam

1. **Isto mexe em dinheiro.** Cada objeto criado roda numa conta com cartão cadastrado. Comando que
   escreve só roda depois do "pode" — mostrando antes o que vai criar, com o orçamento **em reais**.
2. **Tudo nasce `PAUSED`, e é de propósito.** Nunca ative campanha, grupo ou anúncio por conta
   própria "para completar o trabalho".
3. **Uma conta por vez.** O `contas.yaml` pode ter vários clientes; o pedido vale para o que o dono
   nomeou. Nunca rode "em todas" para adiantar.
4. **Estado se consulta, não se lê de documento.** "Quanto gastou?", "está ativa?" — pergunte à API,
   na hora.
5. **Segredo nunca sai do `.env`.** Não escreva credencial em documento, em commit nem na resposta
   do chat.

Esta pasta opera uma conta de Google Ads pela **API oficial** (SDK Python + GAQL). Você é o
operador; o dono não digita comando. **Não há MCP aqui** — tudo é `python3 scripts/<script>.py`.

🔴 **Isto mexe em dinheiro.** Cada objeto que você cria roda numa conta com cartão cadastrado.
Leia as travas antes da primeira escrita.

## Antes de qualquer coisa

```bash
python3 scripts/setup.py check     # o .env está completo? (read-only, pode rodar sempre)
python3 scripts/setup.py test      # quais contas este login alcança? (read-only)
```

Se `test` não listar a conta que o dono quer operar, **o problema é credencial, não é o script** —
vá para a tabela de armadilhas. Não tente contornar com outro comando.

| Arquivo | O que tem | Versionar? |
|---|---|---|
| `.env` | as duas credenciais do OAuth + os customer ids | ⛔ **nunca** |
| `contas.yaml` | apelido → customer id de cada conta/cliente | ⛔ **nunca** |
| `perfil.json` | tipo de negócio, país, site (gerado pelo `setup_perfil.py`) | ⛔ não |
| `docs/` | o manual de conexão, o GAQL e a estratégia | sim |

**Resolvendo "o relatório da tal empresa":** leia o `contas.yaml` e pegue o `customer_id`. Nunca
pergunte o número ao dono se o arquivo já responde. Se o apelido não estiver lá, pergunte —
**não adivinhe pelo nome parecido.**

## As possibilidades

### Leitura — pode rodar sem perguntar

Todos aceitam `--customer-id XXXXXXXXXX` (sem hífens). Sem a flag, usa o `GOOGLE_ADS_CUSTOMER_ID`.

| O dono pede | Comando |
|---|---|
| "quais contas eu acesso?" | `python3 scripts/read.py accounts` |
| "quais campanhas existem?" | `python3 scripts/read.py campaigns` |
| "os grupos dessa campanha" | `python3 scripts/read.py ad-groups --campaign-id XXX` |
| "as palavras-chave" | `python3 scripts/read.py keywords --campaign-id XXX` |
| "os anúncios" | `python3 scripts/read.py ads --campaign-id XXX` |
| "o que as pessoas pesquisaram" | `python3 scripts/read.py search-terms` |
| "as negativas que já existem" | `python3 scripts/read.py negative-keywords` |
| "o índice de qualidade" | `python3 scripts/read.py quality-scores` |
| "as extensões" | `python3 scripts/read.py extensions` |
| "como foi o mês?" | `python3 scripts/insights.py account --date-range LAST_30_DAYS` |
| "qual campanha está gastando mais?" | `python3 scripts/insights.py campaign --date-range LAST_30_DAYS` |
| "e por palavra-chave?" | `python3 scripts/insights.py keyword` |
| "mostra o dia a dia" | `python3 scripts/insights.py daily` |
| "celular ou computador?" | `python3 scripts/insights.py device` |
| "que horas converte melhor?" | `python3 scripts/insights.py hourly` |
| "quanto de busca tem essa palavra?" | `python3 scripts/keyword_planner.py ideas --keywords "a\|b" --limit 50` |

Períodos aceitos: `LAST_7_DAYS`, `LAST_14_DAYS`, `LAST_30_DAYS`, `THIS_MONTH`, `LAST_MONTH` — ou
`--since 2026-08-01 --until 2026-08-31`.

🔴 **Estado nunca se lê de documento.** "Quanto gastou?", "está ativa?", "qual o orçamento?" se
perguntam à API, na hora. Número em documento está errado no dia seguinte.

### Escrita — mostre o que vai fazer e espere o "pode"

| O dono pede | Comando | Antes de rodar |
|---|---|---|
| "cria uma campanha" | `python3 scripts/create.py campaign --name "..." --type SEARCH --budget 5000` | ⚠️ **o orçamento é em CENTAVOS** — 5000 = R$ 50/dia |
| "cria o grupo" | `python3 scripts/create.py ad-group --campaign-id XXX --name "..."` | — |
| "adiciona essa palavra" | `python3 scripts/create.py keyword --ad-group-id XXX --text "..." --match-type PHRASE` | confirme o tipo de correspondência |
| "escreve o anúncio" | `python3 scripts/create.py rsa --ad-group-id XXX --headlines "h1\|h2\|h3" --descriptions "d1\|d2" --url "https://..."` | título ≤ 30 e descrição ≤ 90 caracteres |
| "negativa esse termo" | `python3 scripts/create.py negative --campaign-id XXX --text "..." --match-type PHRASE` | leia o processo de negativação em `docs/estrategia/` |
| "monta a campanha inteira" | `python3 scripts/create_skag.py` | interativo; exige `perfil.json` |
| "muda o orçamento / pausa / ativa" | `python3 scripts/update.py campaign --campaign-id XXX --status PAUSED` | 🔴 **ativar sempre exige o "pode"** |
| "remove essa palavra" | `python3 scripts/delete.py keyword --ad-group-id XXX --criterion-id XXX` | não tem desfazer |

**Tudo que `create.py` cria nasce `PAUSED`.** Isso é proposital e não é bug: quem liga campanha é
o dono, olhando. Não "corrija" isso ativando em seguida por conta própria.

## As travas — o que você NUNCA faz sozinho

1. **Nunca ativar** campanha, grupo ou anúncio sem o dono ter visto e dito "pode". Objeto pausado
   não gasta; objeto ativo gasta em minutos.
2. **Nunca criar sem mostrar antes** o que vai criar: nome, orçamento **em reais** (converta você,
   não deixe o dono decifrar centavos), tipo de correspondência e URL final.
3. **Nunca apagar** palavra-chave, anúncio ou campanha sem confirmação explícita. Métrica histórica
   vai junto e não volta.
4. **Nunca mexer numa conta que o dono não nomeou.** Se `contas.yaml` tem cinco clientes, o pedido
   vale para **um**. Na dúvida, pergunte qual — nunca rode "em todas" para adiantar.
5. **Nunca escrever segredo** em documento, em commit ou na resposta do chat. Se precisar mostrar
   que uma credencial existe, mostre mascarada (o `setup.py check` já faz isso).
6. **Nunca subir orçamento "porque está performando"** sem o dono decidir o número. Você propõe,
   ele escolhe.
7. **Nunca negativar em lote sem revisão termo a termo.** Negativa larga demais mata tráfego bom e
   o efeito só aparece dias depois, quando ninguém mais liga uma coisa à outra.
8. **Nunca dizer que uma campanha "está indo bem" com poucos dias de dados.** Diga quantos dias e
   quantas conversões sustentam a frase.

## As armadilhas — o que o erro diz × o que ele significa

| O que aparece | O que é de verdade | Conserto |
|---|---|---|
| conexão OK, mas **a conta não aparece** | o **projeto** no Cloud ainda está no nível **Test** — só enxerga conta de teste | inscrever o projeto: `docs/00-manual-de-conexao.md` §6.5 |
| `PERMISSION_DENIED` numa conta que o dono vê no painel | falta o `login_customer_id` (a MCC pela qual se entra) | preencher `GOOGLE_ADS_LOGIN_CUSTOMER_ID` |
| `invalid_grant` **uma semana depois** de funcionar | 🔴 o app OAuth ficou em "Testing": refresh token de **7 dias** | §9 do manual — publicar o app e refazer o OAuth |
| `SERVICE_DISABLED` | a Google Ads API não foi ativada no projeto do Cloud | §6.2 do manual |
| `RESOURCE_EXHAUSTED` | teto diário de operações (2.880/dia no Explorer) | esperar o dia virar, ou subir para Basic |
| `developer-token parameter is missing` | biblioteca antiga: abaixo da 32 ela **exige** um token que não é mais emitido | `pip3 install --upgrade 'google-ads>=32'` |
| e-mail **"Basic Access Denied"** falando em *brand profile* | a marca foi verificada mas **não publicada** — são dois botões | `docs/06-subir-para-basic.md` |
| erro de permissão **só no `keyword_planner.py`** — *"not allowed for use with explorer access"* | é o único recurso que o Explorer não alcança | `docs/06-subir-para-basic.md` |
| campanha do botão "Promover" do YouTube **não aparece** | é campanha simplificada; a API a esconde da tabela `campaign` | os números dela aparecem em `insights.py account` |
| `Budget name já existe` | orçamento órfão de uma tentativa anterior | o script já usa timestamp no nome; se voltar, liste os budgets |
| RSA recusado por tamanho | título ≤ **30** e descrição ≤ **90** caracteres | cortar antes de enviar |
| campanha criada com orçamento absurdo | `--budget` é em **centavos** — `50` virou R$ 0,50/dia | corrigir com `update.py campaign --budget` |
| `ModuleNotFoundError: google.ads` | SDK não instalado no Python que você está chamando | `pip3 install -r requirements.txt` |
| nome de campanha sai `Automa��o`, `Tr�fego` | **Windows**: o terminal está em cp1252 e a API devolve UTF-8. O dado está certo, quem corrompe é a saída | rode com `PYTHONIOENCODING=utf-8` na frente do comando (ou `chcp 65001` uma vez na sessão) |

## Quando o dono pedir algo que não tem comando

Existe `GAQL` — a linguagem de consulta do Google Ads — e **você escreve a consulta, ele não**.
`docs/04-consultas-gaql.md` tem as prontas por caso de uso. Se nenhuma servir, monte a sua a partir
delas e rode com `read.py`/`insights.py` como modelo. Não invente nome de campo: confira na
referência antes.

## Onde está escrito o porquê

| Documento | Para quê |
|---|---|
| `docs/00-manual-de-conexao.md` | as duas credenciais, do zero — e a armadilha dos 7 dias |
| `docs/06-subir-para-basic.md` | o degrau do Keyword Planner: verificar **e publicar** a marca |
| `docs/01-o-que-da-pra-fazer.md` | o que a API permite e o que ela não permite |
| `docs/02-gaql-sem-decorar.md` | como a consulta se monta |
| `docs/04-consultas-gaql.md` | consultas prontas por caso de uso |
| `docs/05-aprendizados-da-api.md` | o que já quebrou nesta API e como se contorna |
| `docs/estrategia/` | anatomia do anúncio, montagem de campanha, negativação, rastreamento |

Decisão de estratégia vive lá, não aqui. Se o dono perguntar *"esse anúncio está bom?"*, leia
`docs/estrategia/anatomia-do-anuncio.md` antes de opinar.
