# Fluxo de Caixa — App de Controle Financeiro Pessoal

Um app de controle de fluxo de caixa construído para substituir uma planilha de orçamento mantida manualmente por anos. Roda inteiramente no navegador, sem servidor — os dados ficam salvos no seu próprio computador.

## Arquivos

- **`fluxo-caixa.html`** — o app pronto pra usar. Baixe e dê duplo clique: abre no navegador padrão. Precisa de internet só na primeira abertura (carrega bibliotecas de um CDN); depois funciona offline até fechar a aba.

Não é necessário instalar nada, ter Node.js ou rodar um build step — é um único arquivo HTML com React, Recharts e SheetJS/xlsx carregados via CDN e transpilado no próprio navegador com Babel Standalone.

## Armazenamento de dados

Os dados ficam salvos no `localStorage` do navegador — ou seja, **no mesmo navegador, no mesmo computador**, enquanto você não limpar os dados de navegação. Não há nuvem, não há conta, não há sincronização entre dispositivos. Recomenda-se exportar o `.xlsx` periodicamente como backup (a partir da versão 22, o próprio arquivo exportado também carrega a configuração de cartões — ver Changelog).

## Como importar seus dados

O botão **Importar .xlsx** lê uma planilha com uma aba contendo, no mínimo, colunas "Data" e "Valor" (procura automaticamente por uma aba chamada "Lançamentos" ou similar). Colunas reconhecidas:

| Coluna | Uso |
|---|---|
| Data | Data de vencimento do lançamento |
| Data da compra | Data em que a compra/operação realmente ocorreu (pode ser diferente do vencimento, ex. em parcelas) |
| Descrição | Texto livre |
| Valor | Positivo = entrada, negativo = saída |
| Cartão | Nome da conta, cartão ou meio de pagamento |
| Status pgto | "Pago" ou qualquer outro texto (tratado como Pendente) |
| Fonte | Pessoa ou origem associada ao lançamento |
| Previsto | "Sim"/"Não" — marca um lançamento especulativo, ainda não confirmado |
| Natureza | "Conta líquida" / "Dívida" / "Saldo restrito" |
| Titularidade | "Própria" / "Terceiro" |

Nomes de cartão são comparados ignorando maiúsculas/minúsculas e espaços nas pontas (ex.: "Nubank" e "nubank" são tratados como a mesma conta).

A partir da versão 22, se a planilha tiver uma segunda aba com "cart" no nome (ex. "Cartões e limites") e coluna "Cartão", a configuração de natureza/crédito total/saldo inicial/data de corte de cada conta é lida e mesclada automaticamente — não precisa recadastrar os cartões ao importar em outro computador.

## Tipos de lançamento

O botão **Novo lançamento** oferece cinco modalidades:

1. **Pagamento ou recebimento simples** — um único lançamento, entrada ou saída.
2. **Transferência entre minhas contas** — gera automaticamente duas pontas (saída na origem, entrada no destino).
3. **Compra no cartão** (à vista ou parcelada) — gera uma parcela por mês a partir da data do 1º vencimento; o valor total é dividido entre as parcelas (o arredondamento de centavos é absorvido na última). Sempre classificada como Dívida.
4. **Empréstimo** (ativo — você empresta — ou passivo — você toma emprestado) — gera uma saída/entrada do valor líquido na data do líquido, mais parcelas de recebimento/pagamento a partir do 1º vencimento.
5. **Pagamento/recebimento recorrente** — mesma descrição e valor repetidos mensal ou quinzenalmente (ex.: salário, aluguel), sem numeração de parcela.

## O modelo de "natureza" e "titularidade"

Esses dois campos resolvem o problema central de separar dinheiro real de dívida:

- **Natureza** — o que aquele lançamento representa:
  - *Conta líquida*: dinheiro seu, disponível.
  - *Dívida*: uso de crédito (cartão, financiamento) — não é saldo, é passivo enquanto pendente.
  - *Saldo restrito*: dinheiro real, mas de uso limitado (vale-refeição, benefícios).
- **Titularidade** — de quem é a dívida: *Própria* ou *Terceiro* (ex.: alguém que gastou no seu cartão emprestado). Não afeta o cálculo de crédito utilizado — o limite consumido é o mesmo independentemente de quem gastou.

Cada cartão/conta tem uma natureza **padrão**, definida uma vez no painel "Cartões e limites". Lançamentos individuais podem sobrescrever esse padrão quando o mesmo nome de conta cobre mais de um caso (ex.: parte "Restrito", parte "Dívida").

## Painel "Cartões e limites"

Cadastro por cartão/conta com:

- **Natureza** (define o padrão de herança dos lançamentos daquele nome).
- **Crédito total** (só relevante pra Dívida).
- **Utilizado / Utilizado no mês / Disponível** — calculados a partir de lançamentos de natureza Dívida com status Pendente, excluindo os marcados como Previsto. Mostra também o detalhamento Própria/Terceiro quando há mistura.
- **Saldo inicial + "a partir de"** — permite ignorar todo o histórico anterior a uma data de corte e partir de um saldo real conhecido (útil quando o histórico tem lacunas de dupla entrada).
- **Saldo** — saldo *projetado* da conta: saldo inicial + todo lançamento dessa conta a partir da data de corte, **de qualquer natureza** (líquida, restrita ou dívida) e **pago, pendente ou previsto**. Saldo financeiro e limite de crédito são cálculos deliberadamente diferentes: uma dívida marcada como paga abate direto do saldo da própria conta (pode até ficar negativo, até você ajustar via transferência de outra conta), e uma conta a pagar/receber já lançada entra na projeção mesmo antes de ser paga. Pra ver só o saldo já realizado, filtre Status = Pago e Previsto = Não.

Clicar no nome de um cartão, ou em qualquer valor da linha, filtra a tabela principal só por ele.

## Filtros e busca

Período (De/Até, com botão "Aplicar" pra evitar recálculo a cada tecla), Status, Fonte, Previsto, Cartão, Titularidade, Natureza e busca por texto na descrição — todos combináveis, todos com seleção múltipla e opção "Selecionar tudo".

## Gráfico

Alterna entre duas visões:
- **Saldo** — evolução do saldo acumulado (linha), com granularidade diária ou por lançamento individual (quando o período filtrado é um único dia).
- **Entradas e saídas** — barras separadas por dia, com tooltip mostrando os dois valores.

## Edição

- **Inline**: ícone de lápis em cada linha abre edição direta (data, descrição, cartão, status, fonte, previsto, natureza, titularidade, valor).
- **Em massa**: marque várias linhas (ou "selecionar tudo" do filtro atual) e aplique uma mudança de uma vez — Natureza, Titularidade, Status, Fonte, Previsto, Cartão, Valor, ou Data (vencimento e data da compra, cada uma com modo "definir data exata" ou "deslocar em dias", sempre independentes uma da outra). Exclusão em massa também disponível. Ambas pedem confirmação antes de aplicar.

## Exportação

- **Exportar tudo** — gera um `.xlsx` com todos os lançamentos (importados + novos), incluindo a natureza e titularidade já resolvidas (herdadas do cartão quando aplicável), mais uma aba "Cartões e limites" com a configuração de cada conta.
- **Exportar só novos** — só os lançamentos criados dentro do app, prontos pra colar no final da planilha original (as colunas calculadas por fórmula, como Saldo corrido, o próprio Excel recalcula ao colar dentro de uma Tabela estruturada) — também inclui a aba "Cartões e limites".

**Atenção**: reimportar um arquivo exportado grava a natureza como valor fixo em cada linha (não mais "herdado do cartão"). Se depois você mudar a classificação padrão de um cartão, essas linhas reimportadas não seguem a mudança automaticamente.

## Limitações conhecidas

- Quitação de fatura paga com dinheiro de **outra** conta ainda é manual em dois passos: marcar a dívida como Pago (isso já abate da própria conta da dívida) e lançar uma "Transferência entre minhas contas" levando o dinheiro da conta que efetivamente pagou até a conta da dívida, cobrindo o saldo negativo.
- Sem gerenciamento dedicado de empréstimos (visão por pessoa/credor) ou investimentos — possíveis próximos passos.
- Um "fluxo de utilização por data de compra" (separado do vencimento) ainda não tem visualização própria, embora o dado já seja coletado.
- Dados vivem só no navegador local — sem backup automático em nuvem além da exportação manual do `.xlsx`.

## Stack técnica

React 18 + Recharts (gráficos) + SheetJS/xlsx (leitura e escrita de planilhas), tudo carregado via CDN e transpilado no próprio navegador com Babel Standalone — não requer instalação, Node.js ou build step.

## Changelog

**v26** — "Previsto" agora também conta no saldo. Saldo financeiro e limite de crédito são coisas diferentes: saldo é toda entrada/saída (pago, pendente ou previsto), sem exceção; só o "utilizado" do limite de crédito continua ignorando lançamentos "Previsto".

**v25** — O "Saldo" voltou a contar lançamentos pendentes, não só os pagos, pra servir de previsão de saldo futuro (a v24 tinha restringido demais, contando só pago).

**v24** — Dívida marcada como "Pago" passou a abater do saldo da própria conta (antes só natureza líquida/restrita contava, mesmo quando cartão e carteira eram a mesma conta). Se a conta não tiver saldo suficiente, fica negativa — ajuste manual via "Transferência entre minhas contas".

**v23** — Saldo por movimento (coluna "Saldo" da tabela, "saldo final do período", linha do gráfico) passou a usar o mesmo cálculo reconciliado do painel "Cartões e limites", em vez de somar cegamente todo lançamento (o que misturava dívida com dinheiro real).

**v22** —
1. Cabeçalho de exportação corrigido: a coluna de vencimento na planilha exportada passou a se chamar "Data" (compatível com o que a importação espera).
2. Backup de "Cartões e limites" na planilha exportada/importada.
3. Saldo do painel "Cartões e limites" passou a respeitar o filtro "até" do período selecionado.
