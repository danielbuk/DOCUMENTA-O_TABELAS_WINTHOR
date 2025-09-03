# 🗄️ TABELAS PRINCIPAIS DO SISTEMA WINTHOR

## 📋 Visão Geral
Documentação completa das tabelas principais do sistema Winthor Oracle da DICON, incluindo estrutura, relacionamentos e campos importantes para desenvolvimento e consultas.

## 🏗️ ESTRUTURA GERAL

### Sistema de Vendas
- **PCPEDC**: Pedidos de venda (cabeçalho)
- **PCPEDI**: Itens dos pedidos de venda
- **PCCLIENT**: Cadastro de clientes
- **PCUSUARI**: Usuários/vendedores do sistema

### Sistema de Produtos
- **PCPRODUT**: Cadastro de produtos
- **PCEST**: Estoque de produtos
- **PCDEPTO**: Departamentos de produtos
- **PCSECAO**: Seções de produtos

### Sistema Financeiro
- **PCCOND**: Condições de venda
- **PCTABPR**: Tabelas de preços
- **PCCUST**: Custos dos produtos

---

## 📊 **TABELA: PCPEDC (PEDIDOS DE VENDA)**

### Descrição
Tabela principal que armazena o cabeçalho dos pedidos de venda do sistema.

### Estrutura Principal
```sql
CREATE TABLE PCPEDC (
    NUMPED NUMBER PRIMARY KEY,           -- Número do pedido
    CODCLI NUMBER,                       -- Código do cliente
    CODUSUR NUMBER,                      -- Código do usuário/vendedor
    DATA DATE,                           -- Data do pedido
    POSICAO CHAR(1),                    -- Status do pedido
    CODFILIAL CHAR(2),                  -- Código da filial
    CONDVENDA NUMBER,                    -- Condição de venda
    DTCANCEL DATE,                       -- Data de cancelamento
    VLTOTAL NUMBER(10,2),               -- Valor total do pedido
    -- ... outros campos
);
```

### Campos Principais
| Campo | Tipo | Descrição | Exemplo |
|-------|------|-----------|---------|
| **NUMPED** | NUMBER | Número único do pedido | 12345 |
| **CODCLI** | NUMBER | Código do cliente | 2206 |
| **CODUSUR** | NUMBER | Código do vendedor | 33 |
| **DATA** | DATE | Data do pedido | 01/01/2025 |
| **POSICAO** | CHAR(1) | Status do pedido | 'F' (Finalizado) |
| **CODFILIAL** | CHAR(2) | Código da filial | '1', '98' |
| **CONDVENDA** | NUMBER | Condição de venda | 1, 2, 3 |
| **DTCANCEL** | DATE | Data de cancelamento | NULL |
| **VLTOTAL** | NUMBER(10,2) | Valor total | 1500.50 |

### Valores de POSICAO
- **'A'**: Aberto
- **'B'**: Bloqueado
- **'C'**: Cancelado
- **'F'**: Finalizado
- **'P'**: Pendente

### Valores de CONDVENDA
- **1**: À vista
- **2**: 30 dias
- **3**: 60 dias
- **4**: 90 dias
- **7**: 15 dias
- **8**: 45 dias
- **9**: 75 dias
- **14**: 21 dias
- **15**: 28 dias
- **17**: 120 dias
- **18**: 150 dias
- **19**: 180 dias
- **98**: Consignação
- **99**: Outros

---

## 📦 **TABELA: PCPEDI (ITENS DOS PEDIDOS)**

### Descrição
Tabela que armazena os itens individuais de cada pedido de venda.

### Estrutura Principal
```sql
CREATE TABLE PCPEDI (
    NUMPED NUMBER,                       -- Número do pedido (FK)
    NUMITEM NUMBER,                      -- Número do item
    CODPROD NUMBER,                      -- Código do produto
    QT NUMBER(10,3),                     -- Quantidade
    PVENDA NUMBER(10,4),                 -- Preço de venda
    VLCUSTOFIN NUMBER(10,4),             -- Custo financeiro
    BONIFIC CHAR(1),                     -- Se é bonificação
    -- ... outros campos
);
```

### Campos Principais
| Campo | Tipo | Descrição | Exemplo |
|-------|------|-----------|---------|
| **NUMPED** | NUMBER | Número do pedido (FK) | 12345 |
| **NUMITEM** | NUMBER | Número sequencial do item | 1, 2, 3 |
| **CODPROD** | NUMBER | Código do produto | 1001 |
| **QT** | NUMBER(10,3) | Quantidade vendida | 10.000 |
| **PVENDA** | NUMBER(10,4) | Preço de venda unitário | 2.5000 |
| **VLCUSTOFIN** | NUMBER(10,4) | Custo financeiro unitário | 1.8000 |
| **BONIFIC** | CHAR(1) | Se é bonificação | 'N' ou 'S' |

### Relacionamentos
- **NUMPED** → **PCPEDC.NUMPED** (Pedido pai)
- **CODPROD** → **PCPRODUT.CODPROD** (Produto)

---

## 👥 **TABELA: PCCLIENT (CLIENTES)**

### Descrição
Tabela que armazena o cadastro completo de clientes do sistema.

### Estrutura Principal
```sql
CREATE TABLE PCCLIENT (
    CODCLI NUMBER PRIMARY KEY,           -- Código do cliente
    CLIENTE VARCHAR2(100),               -- Nome/razão social
    CGC VARCHAR2(18),                    -- CNPJ/CPF
    ENDERECO VARCHAR2(100),              -- Endereço
    BAIRRO VARCHAR2(50),                 -- Bairro
    CIDADE VARCHAR2(50),                 -- Cidade
    ESTADO CHAR(2),                      -- Estado
    CEP VARCHAR2(10),                    -- CEP
    -- ... outros campos
);
```

### Campos Principais
| Campo | Tipo | Descrição | Exemplo |
|-------|------|-----------|---------|
| **CODCLI** | NUMBER | Código único do cliente | 2206 |
| **CLIENTE** | VARCHAR2(100) | Nome/razão social | "EMPRESA ABC LTDA" |
| **CGC** | VARCHAR2(18) | CNPJ/CPF | "12.345.678/0001-90" |
| **ENDERECO** | VARCHAR2(100) | Endereço completo | "Rua das Flores, 123" |
| **BAIRRO** | VARCHAR2(50) | Bairro | "Centro" |
| **CIDADE** | VARCHAR2(50) | Cidade | "São Paulo" |
| **ESTADO** | CHAR(2) | Estado | "SP" |
| **CEP** | VARCHAR2(10) | CEP | "01234-567" |

---

## 👤 **TABELA: PCUSUARI (USUÁRIOS/VENDEDORES)**

### Descrição
Tabela que armazena o cadastro de usuários e vendedores do sistema.

### Estrutura Principal
```sql
CREATE TABLE PCUSUARI (
    CODUSUR NUMBER PRIMARY KEY,          -- Código do usuário
    NOME VARCHAR2(100),                  -- Nome completo
    ATIVO CHAR(1),                       -- Se está ativo
    EMAIL VARCHAR2(100),                 -- Email
    -- ... outros campos
);
```

### Campos Principais
| Campo | Tipo | Descrição | Exemplo |
|-------|------|-----------|---------|
| **CODUSUR** | NUMBER | Código único do usuário | 33 |
| **NOME** | VARCHAR2(100) | Nome completo | "João Silva" |
| **ATIVO** | CHAR(1) | Se está ativo | 'S' ou 'N' |
| **EMAIL** | VARCHAR2(100) | Email | "joao@empresa.com" |

### Vendedores Selecionados (Campanhas)
```python
VENDEDORES_CAMPANHA = [
    33, 66, 6, 41, 63, 15, 31, 111, 86, 130,  # Primeira linha
    108, 9, 40, 91, 118, 23, 1, 2, 12, 51     # Segunda linha
]
```

---

## 🏷️ **TABELA: PCPRODUT (PRODUTOS)**

### Descrição
Tabela que armazena o cadastro completo de produtos do sistema.

### Estrutura Principal
```sql
CREATE TABLE PCPRODUT (
    CODPROD NUMBER PRIMARY KEY,          -- Código do produto
    DESCRICAO VARCHAR2(100),             -- Descrição do produto
    CODEPTO NUMBER,                      -- Código do departamento
    CODSEC NUMBER,                       -- Código da seção
    PESOBRUTO NUMBER(10,3),              -- Peso bruto
    PESOLIQ NUMBER(10,3),                -- Peso líquido
    QTUNITCX NUMBER(10,3),               -- Unidades por caixa
    EMBALAGEM VARCHAR2(20),              -- Tipo de embalagem
    CUSTOREP NUMBER(10,4),               -- Custo de reposição
    -- ... outros campos
);
```

### Campos Principais
| Campo | Tipo | Descrição | Exemplo |
|-------|------|-----------|---------|
| **CODPROD** | NUMBER | Código único do produto | 1001 |
| **DESCRICAO** | VARCHAR2(100) | Descrição completa | "Leite Integral 1L" |
| **CODEPTO** | NUMBER | Código do departamento | 102 (Bem Brasil) |
| **CODSEC** | NUMBER | Código da seção | 15 |
| **PESOBRUTO** | NUMBER(10,3) | Peso bruto em kg | 1.050 |
| **PESOLIQ** | NUMBER(10,3) | Peso líquido em kg | 1.000 |
| **QTUNITCX** | NUMBER(10,3) | Unidades por caixa | 12.000 |
| **EMBALAGEM** | VARCHAR2(20) | Tipo de embalagem | "CX" |
| **CUSTOREP** | NUMBER(10,4) | Custo de reposição | 2.5000 |

### Departamentos Importantes
- **102**: Bem Brasil
- **103**: BRF
- **104**: Laticínios
- **105**: Frios

---

## 📦 **TABELA: PCEST (ESTOQUE)**

### Descrição
Tabela que armazena o controle de estoque de todos os produtos por filial.

### Estrutura Principal
```sql
CREATE TABLE PCEST (
    CODPROD NUMBER,                      -- Código do produto
    CODFILIAL CHAR(2),                   -- Código da filial
    QTESTGER NUMBER(10,3),               -- Estoque geral
    QTRESERV NUMBER(10,3),               -- Estoque reservado
    QTBLOQUEADA NUMBER(10,3),            -- Estoque bloqueado
    -- ... outros campos
);
```

### Campos Principais
| Campo | Tipo | Descrição | Exemplo |
|-------|------|-----------|---------|
| **CODPROD** | NUMBER | Código do produto | 1001 |
| **CODFILIAL** | CHAR(2) | Código da filial | '1', '98' |
| **QTESTGER** | NUMBER(10,3) | Estoque geral | 150.000 |
| **QTRESERV** | NUMBER(10,3) | Estoque reservado | 10.000 |
| **QTBLOQUEADA** | NUMBER(10,3) | Estoque bloqueado | 5.000 |

### Cálculo de Estoque Disponível
```sql
-- Estoque disponível em unidades
QTESTGER - QTRESERV - QTBLOQUEADA

-- Estoque disponível em caixas
(QTESTGER - QTRESERV - QTBLOQUEADA) / NULLIF(QTUNITCX, 0)
```

### Filiais
- **'1'**: Filial principal
- **'98'**: Depósito (excluído das consultas)

---

## 🏢 **TABELA: PCDEPTO (DEPARTAMENTOS)**

### Descrição
Tabela que armazena a classificação de departamentos dos produtos.

### Estrutura Principal
```sql
CREATE TABLE PCDEPTO (
    CODEPTO NUMBER PRIMARY KEY,          -- Código do departamento
    DESCRICAO VARCHAR2(100),             -- Nome do departamento
    -- ... outros campos
);
```

### Campos Principais
| Campo | Tipo | Descrição | Exemplo |
|-------|------|-----------|---------|
| **CODEPTO** | NUMBER | Código único | 102 |
| **DESCRICAO** | VARCHAR2(100) | Nome do departamento | "Bem Brasil" |

### Departamentos Principais
| Código | Nome | Descrição |
|--------|------|-----------|
| **102** | Bem Brasil | Produtos da marca Bem Brasil |
| **103** | BRF | Produtos da marca BRF |
| **104** | Laticínios | Produtos lácteos em geral |
| **105** | Frios | Produtos refrigerados |

---

## 📂 **TABELA: PCSECAO (SEÇÕES)**

### Descrição
Tabela que armazena a classificação de seções dentro dos departamentos.

### Estrutura Principal
```sql
CREATE TABLE PCSECAO (
    CODSEC NUMBER PRIMARY KEY,           -- Código da seção
    DESCRICAO VARCHAR2(100),             -- Nome da seção
    -- ... outros campos
);
```

### Campos Principais
| Campo | Tipo | Descrição | Exemplo |
|--------|------|-----------|---------|
| **CODSEC** | NUMBER | Código único | 15 |
| **DESCRICAO** | VARCHAR2(100) | Nome da seção | "Leite" |

---

## 🔗 **RELACIONAMENTOS PRINCIPAIS**

### Diagrama de Relacionamentos
```
PCPEDC (Pedidos)
    ↓ (1:N)
PCPEDI (Itens)
    ↓ (N:1)
PCPRODUT (Produtos)
    ↓ (N:1)
PCDEPTO (Departamentos)
    ↓ (N:1)
PCSECAO (Seções)

PCPEDC (Pedidos)
    ↓ (N:1)
PCCLIENT (Clientes)

PCPEDC (Pedidos)
    ↓ (N:1)
PCUSUARI (Vendedores)

PCPRODUT (Produtos)
    ↓ (1:N)
PCEST (Estoque)
```

### Chaves Estrangeiras
```sql
-- PCPEDI → PCPEDC
ALTER TABLE PCPEDI ADD CONSTRAINT FK_PCPEDI_PCPEDC 
FOREIGN KEY (NUMPED) REFERENCES PCPEDC(NUMPED);

-- PCPEDI → PCPRODUT
ALTER TABLE PCPEDI ADD CONSTRAINT FK_PCPEDI_PCPRODUT 
FOREIGN KEY (CODPROD) REFERENCES PCPRODUT(CODPROD);

-- PCPEDC → PCCLIENT
ALTER TABLE PCPEDC ADD CONSTRAINT FK_PCPEDC_PCCLIENT 
FOREIGN KEY (CODCLI) REFERENCES PCCLIENT(CODCLI);

-- PCPEDC → PCUSUARI
ALTER TABLE PCPEDC ADD CONSTRAINT FK_PCPEDC_PCUSUARI 
FOREIGN KEY (CODUSUR) REFERENCES PCUSUARI(CODUSUR);

-- PCPRODUT → PCDEPTO
ALTER TABLE PCPRODUT ADD CONSTRAINT FK_PCPRODUT_PCDEPTO 
FOREIGN KEY (CODEPTO) REFERENCES PCDEPTO(CODEPTO);

-- PCPRODUT → PCSECAO
ALTER TABLE PCPRODUT ADD CONSTRAINT FK_PCPRODUT_PCSECAO 
FOREIGN KEY (CODSEC) REFERENCES PCSECAO(CODSEC);

-- PCEST → PCPRODUT
ALTER TABLE PCEST ADD CONSTRAINT FK_PCEST_PCPRODUT 
FOREIGN KEY (CODPROD) REFERENCES PCPRODUT(CODPROD);
```

---

## 📊 **ÍNDICES RECOMENDADOS**

### Para Performance de Consultas
```sql
-- Índices principais
CREATE INDEX IDX_PCPEDC_DATA ON PCPEDC(DATA);
CREATE INDEX IDX_PCPEDC_CODCLI ON PCPEDC(CODCLI);
CREATE INDEX IDX_PCPEDC_CODUSUR ON PCPEDC(CODUSUR);
CREATE INDEX IDX_PCPEDC_POSICAO ON PCPEDC(POSICAO);
CREATE INDEX IDX_PCPEDC_CODFILIAL ON PCPEDC(CODFILIAL);

CREATE INDEX IDX_PCPEDI_NUMPED ON PCPEDI(NUMPED);
CREATE INDEX IDX_PCPEDI_CODPROD ON PCPEDI(CODPROD);

CREATE INDEX IDX_PCPRODUT_CODEPTO ON PCPRODUT(CODEPTO);
CREATE INDEX IDX_PCPRODUT_CODSEC ON PCPRODUT(CODSEC);

CREATE INDEX IDX_PCEST_CODPROD ON PCEST(CODPROD);
CREATE INDEX IDX_PCEST_CODFILIAL ON PCEST(CODFILIAL);

-- Índices compostos
CREATE INDEX IDX_PCPEDC_CLI_DATA ON PCPEDC(CODCLI, DATA);
CREATE INDEX IDX_PCPEDC_USU_DATA ON PCPEDC(CODUSUR, DATA);
CREATE INDEX IDX_PCPEDC_FIL_POS ON PCPEDC(CODFILIAL, POSICAO);
```

---

## ⚠️ **CONSIDERAÇÕES IMPORTANTES**

### Performance
- **Joins**: Sempre usar índices nas chaves estrangeiras
- **Filtros**: Aplicar filtros mais restritivos primeiro
- **Datas**: Usar índices em campos de data para consultas temporais

### Integridade
- **Chaves estrangeiras**: Sempre verificar existência antes de inserir
- **Transações**: Usar transações para operações complexas
- **Validações**: Validar dados antes de inserir/atualizar

### Manutenção
- **Backup**: Fazer backup regular das tabelas
- **Estatísticas**: Atualizar estatísticas do Oracle regularmente
- **Fragmentação**: Monitorar fragmentação das tabelas

---

## 🔍 **QUERIES DE EXEMPLO**

### Consulta Básica de Pedidos
```sql
SELECT 
    C.NUMPED,
    C.DATA,
    CLI.CLIENTE,
    U.NOME AS VENDEDOR,
    C.POSICAO,
    C.VLTOTAL
FROM PCPEDC C
JOIN PCCLIENT CLI ON C.CODCLI = CLI.CODCLI
JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
WHERE C.POSICAO = 'F'
AND C.DATA >= SYSDATE - 30
ORDER BY C.DATA DESC;
```

### Consulta de Estoque por Departamento
```sql
SELECT 
    D.DESCRICAO AS DEPARTAMENTO,
    COUNT(P.CODPROD) AS PRODUTOS,
    SUM(E.QTESTGER) AS ESTOQUE_TOTAL
FROM PCPRODUT P
JOIN PCEST E ON P.CODPROD = E.CODPROD
JOIN PCDEPTO D ON P.CODEPTO = D.CODEPTO
WHERE E.CODFILIAL = '1'
GROUP BY D.DESCRICAO
ORDER BY ESTOQUE_TOTAL DESC;
```

---

**📋 Sistema**: Winthor Oracle DICON  
**🗄️ Banco**: Oracle Database  
**🔧 Versão**: Sistema ERP Completo  
**📊 Finalidade**: Gestão de Distribuição e Vendas
