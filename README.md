# Atualiza Carteira

## 📌 1. Objetivo
Esta rotina consolida os dados da carteira financeira (títulos, contratos e indicadores de cobrança) em uma tabela histórica denominada ZA5. O objetivo é servir como fonte de dados otimizada para ferramentas de BI (Power BI, Tableau, etc), permitindo análises de time-series através do controle de versões e datas de execução.

## 💾 2. Dicionário de Dados (Estrutura ZA5)
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

---

## ⚙️ 3. Fluxo de Processamento

1.  **Versionamento:**
    * A cada execução bem-sucedida, o sistema incrementa o campo `ZA5_VERSAO`.
    * Caso a rotina seja executada mais de uma vez na mesma data, o sistema mantém a versão anterior e cria uma nova versão para a filial selecionada.
2.  **Lógica de Carga:**
    * Olha a ZB2 mais recente por cliente e retorna a situação
    * Olha a SE5 e obtem o ultimo motivo de baixa.
    * Concatena na sequencia todos os motivos de baixa
    * TOTAIS DE VALOR/JUROS/DESCONTO NA SE5020 (mesma chave)
    * Exclui contrato inteiro se houver qualquer RES ou DIST em qualquer parcela (case-insensitive)
  
