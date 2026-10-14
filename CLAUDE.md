# Sabe Tudo — Central de análise de clientes (V4 Falcon)

Este repositório existe para uma única função: **quando o usuário perguntar sobre um cliente, cruzar Meta Ads + Google Ads + planilha de leads e entregar um resumo executivo.**

Exemplos de gatilho: "como está o resultado da GSC?", "como vai a Official Time?", "me dá um panorama da Miyamura".

## Perfil do usuário
Profissional de marketing de performance (SEO, tráfego pago, funis), analítico e prático. Comunicação direta, orientada a decisão. Responder em português do Brasil, sem enrolação.

## Fonte da verdade: planilha "Torre de Controle | Falcon"
- Google Sheets ID: `1RGaZMiM9z2bDjkuz2F186Fiyl8sK5EwJiYevjBfT7eg`
- Ler com `mcp__Google_Drive__read_file_content` (fileId acima).
- Aba usada para resolver o cliente: **`IDS DOS CLIENTES`**. Colunas:
  - `EMPRESA`: nome do cliente
  - `META ADS`: ID da conta de anúncios do Meta (sem prefixo `act_`)
  - `GOOGLE ADS`: ID da conta do Google Ads, sem hífens (ex.: `2953158507` = `295-315-8507`)
  - `GROWTHPACK`: nome do projeto/Growthpack
  - `leads`: nome da planilha de leads do cliente (pode estar vazio)
- Abas auxiliares úteis: `Clientes ativos` (fee, responsáveis, investimento em mídia) e `Investimento` (projetado x utilizado).
- **Não usar nem citar** a aba `Time` (tem endereço e data de nascimento de pessoas).

## Fluxo obrigatório a cada pergunta sobre cliente
1. **Resolver o cliente.** Ler a aba `IDS DOS CLIENTES` e achar a linha (match tolerante a caixa/acento/abreviação). Se houver ambiguidade ou o cliente não estiver na aba, perguntar antes de seguir. Nunca inventar ID.
2. **Meta Ads** via Windsor (`mcp__Windsor_ai__get_data`, connector `facebook`, filtrando pelo ID da coluna `META ADS`). Se necessário, confirmar a conta com `get_connectors`.
3. **Google Ads** via Windsor (`get_data`, connector `google_ads`, filtrando pelo ID da coluna `GOOGLE ADS`, formato `XXX-XXX-XXXX`).
4. **Conversão / leads.** Cada cliente tem um método de conversão próprio (ver seção "Método de conversão"). Buscar a planilha indicada na coluna `leads` (`mcp__Google_Drive__search_files` e depois `read_file_content`). Se a coluna estiver vazia, não existe planilha: dizer isso em uma linha e seguir só com a mídia.
5. **Analisar com as skills**: `anthropic-skills:ads` para conta de mídia (CPA, ROAS, CTR, CPM, frequência, pacing, campanhas/criativos que puxam resultado ou desperdício) e `anthropic-skills:argus` quando a pergunta for sobre gargalo de funil/conversão. Usar `anthropic-skills:seo-audit` / `scope-auditor` só se a pergunta for de SEO.
6. **Entregar o resumo** no formato abaixo.

Paralelizar as chamadas independentes (Meta, Google e leads ao mesmo tempo).

## Método de conversão (varia por cliente)
Não existe padrão único. Antes de calcular qualquer métrica, descobrir como aquele cliente converte:
- **E-commerce** (ex.: Official Time): resultado = compras, receita, ROAS, ticket médio, CPA de compra. Não falar em "lead" nem CPL.
- **Geração de leads / inside sales** (ex.: GSC, Miyamura): resultado = leads, CPL, e o que a planilha mostrar de qualificação.
- Qualquer outro modelo: inferir pelas conversões que a conta realmente registra no Windsor (campos de actions/conversions por campanha) e pelo conteúdo da planilha.

Planilhas de leads **não têm formato padrão**: cada uma é diferente. A cada consulta, abrir a planilha, identificar sozinho colunas de data, origem/canal, status/qualificação, valor e responsável, e dizer ao usuário quais colunas usou. Se a estrutura for ambígua, perguntar em vez de supor. Cruzar leads da planilha com os da plataforma por período e canal quando houver coluna de origem; sem ela, comparar só totais.

## Metas
Ainda **não há metas por cliente**. O veredito (bem / atenção / risco) deve se basear em variação contra o período anterior, tendência e anomalias, sem inventar benchmark de CPL/CPA/ROAS. Não cobrar meta do usuário. Se metas forem definidas no futuro, serão registradas em coluna da aba `IDS DOS CLIENTES`.

## Período
O usuário informa o período no início de cada chat (ex.: "últimos 7 dias", "setembro", "01/10 a 03/10"). Guardar esse período para o resto da conversa e usá-lo em todas as consultas.
- Se a primeira pergunta sobre um cliente vier **sem período**, perguntar o período antes de consultar qualquer fonte. Não assumir um padrão.
- Comparar com o período imediatamente anterior de mesmo tamanho, salvo pedido diferente.
- Sempre declarar o período usado na entrega.

## Formato da entrega (curto, no máximo uma tela)
1. **Veredito em 1–2 linhas**: o cliente está bem, em atenção ou em risco, e por quê.
2. **Placar** (tabela): Meta | Google | Total, com investimento, leads/conversões, CPL/CPA, ROAS (se houver) e variação vs. período anterior.
3. **Conversão real (planilha)**: para lead gen, volume, qualidade/status se existir e divergência entre leads das plataformas e da planilha; para e-commerce, vendas e receita da fonte disponível versus o reportado nas plataformas. Dizer quais colunas foram usadas.
4. **Principais pontos** (3–5 bullets): o que está puxando resultado, o que está desperdiçando verba, anomalias (queda de entrega, CPL subindo, frequência alta, conta sem gasto).
5. **Próximas ações** (2–4 itens priorizados, com responsável sugerido se der para inferir de `Clientes ativos`).

Regras de entrega:
- Números sempre com fonte e período. Se um dado não veio, escrever "não disponível" e dizer por quê. Nunca estimar ou preencher lacunas.
- Destacar discrepâncias entre fontes em vez de escolher uma silenciosamente.
- Sem relatório longo, sem enfeite. Só aprofundar se o usuário pedir.

## Limitações conhecidas
- O conector **Meta_Ads** (MCP direto) exige autorização manual; usar o Windsor como caminho padrão.
- O connector `googlesheets` do Windsor não está conectado; planilhas são lidas pelo Google Drive MCP.
- Várias células da planilha mestre mostram `#REF!`/`#N/A`; ignorar esses campos e não citar como dado.
- Cliente com conta em mais de um ID (ex.: Miyamura tem "Cartao" e "Pix" no Meta): somar e mostrar o detalhamento.

## Particularidades por cliente

### Empório Mega 100 (lead gen / inside sales via WhatsApp, B2B para pizzarias, lanchonetes e restaurantes)
Aplicar sempre que o usuário perguntar sobre "Empório Mega 100", "Mega 100" ou "Emporio".
- **Linha na `IDS DOS CLIENTES`:** não preenchida (confirmado pelo usuário). Mapeamento confirmado por ele: Meta Ads = conta **CTA MEGA 100** (`1051821444101775`). **Google Ads: não tem**, não consultar nem citar como lacuna. Se a linha for preenchida na aba, a aba passa a ser a fonte e este bloco deve ser limpo.
- **Responsáveis** (`Clientes ativos`): conta Luma, tráfego Wallace. Fee R$ 6.027,00, mídia projetada R$ 2.000,00/mês, CRM "Não tem".
- **Mídia (Windsor, `facebook`):** campanha `01-[LEAD/WPP] CBO`. Resultado = leads (`actions_lead`) e conversas WhatsApp (`actions_onsite_conversion_messaging_conversation_started_7d`). Os dois eventos provavelmente se sobrepõem: mostrar os dois, não somar, e dizer qual usou no CPL. Não falar em compra, receita ou ROAS.
- **Planilha de leads:** arquivo separado (não é aba da mestre), **"[Empório Mega 100] Leads formulário"**, fileId `1x15ASW-nhI-csH5i-ejNbZlcgyiNgO8q80IPy6UrV4w`.
  - `read_file_content` só devolve amostra das primeiras linhas. Para contar, usar `download_file_content` com `exportMimeType` xlsx (o resultado é salvo em arquivo), decodificar o base64 e processar com Python (`openpyxl`, `python3 -I`, em pasta nova do scratchpad).
  - **Aba `Dados`** (cabeçalho na linha 2, dados da linha 3): é onde está o funil e o acompanhamento. Coluna A = timestamp (UTC, pode vir como texto ISO ou datetime), B Nome, C Tel, E tipo de estabelecimento, F cidade, G–M 1º a 7º contato, N Temperatura (frio/morno/quente/desqualificado), O Status, P Informações, Q Fechou, R–V UTMs.
  - **Abas `Vend1`, `Vend2`, `vend3`, `vend4`:** os 4 vendedores, distribuição em rodízio. Coluna A = timestamp. Servem para conferir volume (soma das 4 = total de leads). Nelas o acompanhamento costuma estar vazio.
  - Ignorar `_controle` (contador do rodízio). Não usar dados pessoais dos leads (nome, telefone, e-mail) na entrega.
- **Como calcular:** período em datas de calendário, comparado com o período anterior de mesmo tamanho. Leads da planilha = linhas da `Dados` com data no período (UTC; citar a diferença de fuso se relevante). Calcular: leads, % com 1º contato (G = TRUE ou "x"), distribuição de Temperatura, Status, e fechamentos.
  - **Fechou:** o campo é texto livre ("Sim", "Fechou", "Fechou?", "Negociação", "Cadastrado", "Perdido"). Só contar venda confirmada quando for "Sim" ou "Fechou" sem interrogação. "Fechou?" e "Negociação" são pendentes e devem aparecer à parte. Perguntar ao usuário em caso de dúvida.
  - **Cruzamento:** plataforma (leads Meta) x planilha (total de leads no período). Divergência pequena (<2%) é normal. UTMs costumam estar vazias, então comparar só totais e dizer isso.
- **Foco da análise (Argus):** a restrição do funil é o comercial, não a mídia. Sempre mostrar a taxa de 1º contato, qualificação (morno/quente) e fechamentos ao lado do CPL. CPL baixo sem contato não é resultado. Não recomendar escalar verba enquanto o contato com os leads estiver baixo (referência dada na última análise: voltar a escalar com 1º contato acima de 50%, ajustável pelo usuário).
- **Ações sugeridas por padrão:** Luma cobra o cliente por SLA de 1º contato e atualização diária da planilha pelos 4 vendedores; Wallace mantém ou escala verba conforme o contato e preenche as UTMs; pedir os fechamentos reais da semana para calcular custo por venda.
- **Referência (07/10 a 13/10/2026, só para comparação futura):** investimento 479,96; 485 leads Meta (CPL 0,99); 639 conversas; 481 leads na planilha; 14 com 1º contato (2,9%); 0 vendas confirmadas.

## Manutenção
Se novos clientes entrarem, a fonte é a aba `IDS DOS CLIENTES`; não duplicar IDs neste arquivo.
