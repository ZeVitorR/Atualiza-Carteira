# Atualiza Carteira

## 📌 1. Objetivo
Esta rotina consolida os dados da carteira financeira (títulos, contratos e indicadores de cobrança) em uma tabela histórica denominada ZA5. O objetivo é servir como fonte de dados otimizada para ferramentas de BI (Power BI, Tableau, etc), permitindo análises de time-series através do controle de versões e datas de execução.

## ✏️ 2. Rotina

### Autor: José Vitor Rodrigues
### Data: 03/11/2025
### Linguagem: tlpp (protheus)

O código está estruturado em blocos funcionais que garantem desde a interface com o usuário até a performance na extração de dados.

### I. Interface e Seleção (SelecionaFilial / defineFiliais)
Interface: Utiliza a classe FWMarkBrowse com uma tabela temporária (FWTemporaryTable) para exibir as filiais disponíveis (origem SM0 cruzada com ZA2).
Interação: O usuário marca as filiais desejadas. A função defineFiliais converte essas marcações em uma string formatada para a cláusula IN do SQL.

### II. Inteligência da Extração (Consulta)
A extração é realizada via Embedded SQL (TCQUERY) para máxima performance. Pontos de destaque da Query:
Joins Complexos: Cruza SE1 (Títulos) com SB1 (Produtos), ZZC (Notificações), ZB2 (Situação de Cobrança) e SE5 (Baixas).
Outer Apply: Utilizado para buscar de forma performática:
   O último motivo de baixa da SE5.
   A última situação de cobrança da ZB2.
   Somas de valores recebidos, juros e descontos diretamente no banco de dados.
Tratamento de Dados: 
   Converte datas do Protheus (YYYYMMDD) para o tipo Date do SQL.
   Trata campos nulos e vazios com ISNULL e CASE WHEN.
   Filtra títulos que possuem motivos de baixa específicos como 'RES' ou 'DIST' (Rescisão/Distrato) para evitar dados inconsistentes no BI.

### III. Gestão de Histórico e Versão (realizaSalvamento / retornaVersao)
Versionamento: A função retornaVersao busca o maior número de versão já existente para a filial e incrementa +1, garantindo que cada carga seja única.
Gravação: Percorre o resultado da consulta e utiliza RecLock para inserir os registros na ZA5.
Feedback: Exibe uma régua de processamento para o usuário informando "Salvando X de Y".

### IV. Manutenção de Dados (realizaDeletacao)
A rotina possui uma regra de limpeza automática: ela busca registros na ZA5 que foram executados há exatamente 2 anos (baseado no mês/ano) e realiza a exclusão física (DbDelete) para evitar o crescimento infinito da base de dados.

## 💾 3. Dicionário de Dados (Estrutura ZA5)
| Campo | Título | Tipo | Tam | Descrição |
| :--- | :--- | :--- | :--- | :--- |
| **ZA5_FILIAL** | Filial | Caractere | 6 | Filial do Sistema |
| **ZA5_VERSAO** | Versão | Numérico | 6 | Número incremental da carga (Snapshot) |
| **ZA5_DTEXEC** | Data Exec | Data | 8 | Data real da execução da rotina |
| **ZA5_ZZCEMP** | Cód. Emp Ori | Caractere | 6 | Código da Empresa de Origem |
| **ZA5_PREFIX** | Prefixo | Caractere | 3 | Prefixo do Título (SE1->E1_PREFIXO) |
| **ZA5_NUMERO** | Num Título | Caractere | 6 | Número do Título (SE1->E1_NUM) |
| **ZA5_PARCEL** | Parcela | Caractere | 3 | Parcela do Título (SE1->E1_PARCELA) |
| **ZA5_CODCLI** | Cod Cliente | Caractere | 6 | Código do Cliente (SA1->A1_COD) |
| **ZA5_NOMCLI** | Nome Cliente | Caractere | 50 | Nome/Razão Social do Cliente |
| **ZA5_CODPRO** | Cod Produto | Caractere | 15 | Código do Produto/Serviço |
| **ZA5_NOMPRO** | Nome Produto | Caractere | 60 | Nome do Produto/Serviço |
| **ZA5_VALOR** | Val Original | Numérico | 16 | Valor Nominal do Título |
| **ZA5_SALDO** | Saldo Devido | Numérico | 16 | Saldo Devedor Atual na Data do Snapshot |
| **ZA5_VLPARC** | Val Parcela | Numérico | 16 | Valor da Parcela Individual |
| **ZA5_VLRECB** | Val Recebido | Numérico | 16 | Total já recebido (Baixas acumuladas) |
| **ZA5_DTVENC** | Vencimento | Data | 8 | Data de Vencimento Real |
| **ZA5_DTEMIS** | Data Emissao | Data | 8 | Data de Emissão do Título |
| **ZA5_DTBX** | Data Baixa | Data | 8 | Data da Última Baixa registrada |
| **ZA5_MOTBX** | Motivo Baixa | Caractere | 15 | Motivo da Baixa do Título |
| **ZA5_SITCOB** | Sit Cobrança | Caractere | 30 | Status/Situação da Cobrança atual |
| **ZA5_BANCO** | Banco | Caractere | 3 | Código do Banco Portador |
| **ZA5_AGENCI** | Agencia | Caractere | 6 | Agência Bancária |
| **ZA5_CONTA** | Numero Conta | Caractere | 15 | Número da Conta Bancária |
| **ZA5_INDICE** | Índice | Caractere | 10 | Nome do Índice (IPCA, IGPM, etc) |
| **ZA5_TPINDI** | Tipo Índice | Caractere | 10 | Classificação do Tipo de Índice |
| **ZA5_VLINDX** | Valor Indice | Numérico | 16 | Valor do Índice Aplicado no período |
| **ZA5_JUROS** | Juros | Numérico | 16 | Valor de Juros Calculados/Acumulados |
| **ZA5_DESCNT** | Desconto | Numérico | 16 | Valor de Desconto Concedido |
| **ZA5_FIXVAR** | Fixa/Variáve | Caractere | 10 | Classificação de Taxa (Fixa ou Variável) |
| **ZA5_TPCONT** | Tp Contrato | Caractere | 15 | Tipo de Contrato Relacionado |
| **ZA5_SERASA** | Dta Serasa | Data | 8 | Data de Registro no Serasa |
| **ZA5_INCSER** | Inc Serasa | Caractere | 5 | Indicador de Inclusão no Serasa |
| **ZA5_RETSER** | Ret Serasa | Caractere | 5 | Indicador de Retirada do Serasa |
| **ZA5_DTENOT** | Env Notifica | Data | 8 | Data de Envio da Notificação |
| **ZA5_DTNOTI** | Noticação | Data | 8 | Data de Recebimento da Notificação |
| **ZA5_DTINTI** | Dt Intimaçao | Data | 8 | Data da Intimação Judicial/Extrajudicial |
| **ZA5_DTREGI** | Dta Registro | Data | 8 | Data de Registro em Cartório |
| **ZA5_LEILAO** | Leilão | Caractere | 5 | Status de Leilão do Ativo |
| **ZA5_SECURI** | Secur | Caractere | 6 | Identificador da Securitizadora |
| **ZA5_PARCE** | Parceiro | Caractere | 6 | Código do Parceiro de Negócio |

