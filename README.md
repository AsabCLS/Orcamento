# Fluxo de Caixa — App de Controle Financeiro Pessoal

Um app de controle de fluxo de caixa construído para substituir uma planilha de orçamento mantida manualmente por anos. Roda inteiramente no navegador, sem servidor — os dados ficam salvos no seu próprio computador.

## Arquivos

- **`fluxo-caixa.html`** — o app pronto pra usar. Baixe e dê duplo clique: abre no navegador padrão. Precisa de internet só na primeira abertura (carrega bibliotecas de um CDN); depois funciona offline até fechar a aba.
- **`fluxo-caixa.jsx`** — o código-fonte do componente React. Não abre sozinho em nada; serve como base caso o app evolua pra um projeto com ferramentas de build (Vite, Next.js) no futuro.

Se você só quer usar o app, precisa apenas do `.html`.

## Armazenamento de dados

Os dados ficam salvos no `localStorage` do navegador — ou seja, **no mesmo navegador, no mesmo computador**, enquanto você não limpar os dados de navegação. Não há nuvem, não há conta, não há sincronização entre dispositivos. Recomenda-se exportar o `.xlsx` periodicamente como backup.

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

## Tipos de lançamento

O botão **Novo lançamento** oferece quatro modalidades:

1. **Pagamento ou recebimento simples** — um único lançamento, entrada ou saída.
2. **Transferência entre minhas contas** — gera automaticamente duas pontas (saída na origem, entrada no destino).
3. **Compra no cartão** (à vista ou parcelada) — gera uma parcela por mês a partir da data do 1º vencimento; o valor total é dividido entre as parcelas (o arredondamento de centavos é absorvido na última). Sempre classificada como Dívida.
4. **Empréstimo** (ativo — você empresta — ou passivo — você toma emprestado) — gera uma saída/entrada do valor líquido na data do líquido, mais parcelas de recebimento/pagamento a partir do 1º vencimento.
5. **Pagamento/recebimento recorrente** — mesma descrição e valor repetidos mensal ou quinzenalmente (ex.: salário, aluguel), sem numeração de parcela.

## O modelo de "natureza" e "titularidade"

Esses dois campos resolvem o problema central de separar dinheiro real de dívida:

- **Natureza** — o que aquele lançamento representa:
  - *Conta líquida*: dinheiro seu, disponível.
  - *Dívida*: uso de crédito (cartão, financiamento) — não é saldo, é passivo.
  - *Saldo restrito*: dinheiro real, mas de uso limitado (vale-refeição, benefícios).
- **Titularidade** — de quem é a dívida: *Própria* ou *Terceiro* (ex.: alguém que gastou no seu cartão emprestado). Não afeta o cálculo de crédito utilizado — o limite consumido é o mesmo independentemente de quem gastou.

Cada cartão/conta tem uma natureza **padrão**, definida uma vez no painel "Cartões e limites". Lançamentos individuais podem sobrescrever esse padrão quando o mesmo nome de conta cobre mais de um caso (ex.: parte "Restrito", parte "Dívida").

## Painel "Cartões e limites"

Cadastro por cartão/conta com:
- **Natureza** (define o padrão de herança dos lançamentos daquele nome).
- **Crédito total** (só relevante pra Dívida).
- **Utilizado / Utilizado no mês / Disponível** — calculados a partir de lançamentos com status Pendente, excluindo os marcados como Previsto. Mostra também o detalhamento Própria/Terceiro quando há mistura.
- **Saldo inicial + "a partir de"** — pra contas líquidas/restritas, permite ignorar todo o histórico anterior a uma data de corte e partir de um saldo real conhecido (útil quando o histórico tem lacunas de dupla entrada).
- **Saldo** — soma apenas de lançamentos com status "Pago", refletindo sempre o saldo atual real (não o previsto/futuro).

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

- **Exportar tudo** — gera um `.xlsx` com todos os lançamentos (importados + novos), incluindo a natureza e titularidade já resolvidas (herdadas do cartão quando aplicável).
- **Exportar só novos** — só os lançamentos criados dentro do app, prontos pra colar no final da planilha original (as colunas calculadas por fórmula, como Saldo corrido, o próprio Excel recalcula ao colar dentro de uma Tabela estruturada).

**Atenção**: reimportar um arquivo exportado grava a natureza como valor fixo em cada linha (não mais "herdado do cartão"). Se depois você mudar a classificação padrão de um cartão, essas linhas reimportadas não seguem a mudança automaticamente.

## Limitações conhecidas

- O sistema usa partidas dobradas apenas na quitação de fatura (marcar parcela como Pago + lançar a saída real da conta que pagou) — hoje isso é manual, em dois passos.
- Sem gerenciamento dedicado de empréstimos (visão por pessoa/credor) ou investimentos — possíveis próximos passos.
- Um "fluxo de utilização por data de compra" (separado do vencimento) ainda não tem visualização própria, embora o dado já seja coletado.
- Dados vivem só no navegador local — sem backup automático em nuvem.

## Stack técnica

React 18 + Recharts (gráficos) + SheetJS/xlsx (leitura e escrita de planilhas), tudo carregado via CDN e transpilado no próprio navegador com Babel Standalone — não requer instalação, Node.js ou build step.
