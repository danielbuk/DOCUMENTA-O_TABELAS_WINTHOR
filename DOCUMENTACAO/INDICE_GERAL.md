# 📚 ÍNDICE GERAL - QUERIES ORACLE WINTHOR DICON

## 🎯 **BUSCA RÁPIDA POR FUNCIONALIDADE**

### 🎯 **CAMPANHAS PROMOCIONAIS**
- **[Campanha BRF](campanhas/BRF_CAMPANHA.md)**
  - Vendas por mês específico
  - Dados consolidados (3 meses)
  - Meta: 170 toneladas
  - Departamento: 103 (BRF)

- **[Campanha Bem Brasil](campanhas/BEM_BRASIL_CAMPANHA.md)**
  - Positivação de CNPJ
  - Carinha e Anel de Cebola
  - Crescimento em kg
  - Departamento: 102 (Bem Brasil)

### 📊 **RELATÓRIOS E DASHBOARDS**
- **[Relatório Cliente 2206](dashboards/RELATORIO_CLIENTE_2206.md)**
  - Vendas dos últimos 6 meses
  - Análise de produtos
  - Resumo mensal
  - Faturamento e custos

- **[Dashboard DICON](dashboards/DASHBOARD_DICON.md)**
  - Google Sheets API
  - Planilha de preços
  - Estoque VILOG

### 🔄 **SINCRONIZAÇÃO E ATUALIZAÇÃO**
- **[Atualizador VILOG](atualizadores/ATUALIZADOR_VILOG.md)**
  - Estoque Oracle → Google Sheets
  - Produtos, custos, departamentos
  - Exclui filial 98 (depósito)

### 🗄️ **ESTRUTURA DO BANCO**
- **[Tabelas Principais](banco/TABELAS_PRINCIPAIS.md)**
  - PCPEDC, PCPEDI, PCPRODUT
  - PCEST, PCCLIENT, PCUSUARI
  - PCDEPTO, PCSECAO

---

## 🔍 **BUSCA POR TIPO DE QUERY**

### 📈 **ANÁLISE DE VENDAS**
| Sistema | Query | Descrição |
|---------|-------|-----------|
| **BRF** | `get_dados_vendas_brf()` | Vendas mensais por vendedor |
| **Bem Brasil** | `get_positivacao_cnpj()` | Positivação de clientes |
| **Cliente 2206** | `get_vendas_ultimos_6_meses()` | Histórico de vendas |

### 📦 **CONTROLE DE ESTOQUE**
| Sistema | Query | Descrição |
|---------|-------|-----------|
| **VILOG** | `SQL_QUERY` | Estoque disponível por produto |
| **VILOG** | Análise por departamento | Estoque por departamento |
| **VILOG** | Produtos com baixo estoque | Estoque < 5 caixas |

### 👥 **ANÁLISE DE VENDEDORES**
| Sistema | Query | Descrição |
|---------|-------|-----------|
| **BRF** | Ranking de vendedores | Performance por peso |
| **Bem Brasil** | Crescimento em kg | Performance por volume |
| **Geral** | Vendedores ativos | Lista de vendedores |

---

## 🏷️ **BUSCA POR TABELA PRINCIPAL**

### **PCPEDC (Pedidos de Venda)**
- **Campanha BRF**: Filtros por período, departamento 103
- **Campanha Bem Brasil**: Filtros por período, departamento 102
- **Cliente 2206**: Filtros por cliente específico, últimos 6 meses
- **Filtros comuns**: POSICAO = 'F', DTCANCEL IS NULL

### **PCPEDI (Itens dos Pedidos)**
- **Cálculos**: QT × PESOLIQ, QT × PVENDA, QT × VLCUSTOFIN
- **Filtros**: BONIFIC = 'N'
- **Joins**: PCPEDC, PCPRODUT

### **PCPRODUT (Produtos)**
- **Departamentos**: 102 (Bem Brasil), 103 (BRF)
- **Campos**: PESOBRUTO, PESOLIQ, QTUNITCX, CUSTOREP
- **Joins**: PCDEPTO, PCSECAO

### **PCEST (Estoque)**
- **Cálculo**: QTESTGER - QTRESERV - QTBLOQUEADA
- **Filtros**: CODFILIAL <> '98'
- **Conversão**: Unidades para caixas

---

## 🔧 **BUSCA POR CONFIGURAÇÃO**

### **Conexão Oracle**
```python
DB_HOST = '10.0.0.10'
DB_PORT = 1521
DB_SERVICE = 'WINT'
DB_USER = 'dicon'
DB_PASSWORD = 'wdicon01'
```

### **Oracle Client**
```python
oracledb.init_oracle_client(lib_dir=r"C:\oracle\instantclient_23_9")
```

### **Vendedores Selecionados**
```python
VENDEDORES_CAMPANHA = [
    33, 66, 6, 41, 63, 15, 31, 111, 86, 130,  # Primeira linha
    108, 9, 40, 91, 118, 23, 1, 2, 12, 51     # Segunda linha
]
```

### **Departamentos**
- **102**: Bem Brasil
- **103**: BRF
- **104**: Laticínios
- **105**: Frios

---

## 📊 **BUSCA POR CÁLCULO**

### **Estoque Disponível**
```sql
-- Em unidades
QTESTGER - QTRESERV - QTBLOQUEADA

-- Em caixas
(QTESTGER - QTRESERV - QTBLOQUEADA) / NULLIF(QTUNITCX, 0)
```

### **Valores de Venda**
```sql
-- Valor total
QT × PVENDA

-- Custo total
QT × VLCUSTOFIN

-- Peso total
QT × PESOBRUTO (ou PESOLIQ)
```

### **Crescimento Percentual**
```sql
-- Crescimento CNPJ
((ATUAL - REFERENCIA) / REFERENCIA) × 100

-- Crescimento KG
((ATUAL_KG - REFERENCIA_KG) / REFERENCIA_KG) × 100
```

---

## 🚀 **CASOS DE USO COMUNS**

### **1. Relatório de Campanha**
```python
# Buscar dados de vendas
dados_vendas = get_dados_vendas_brf(mes=8, ano=2025)

# Gerar ranking
ranking = dados_vendas.sort_values('QUANTIDADE EM PESO VENDIDO', ascending=False)

# Calcular total
total_campanha = dados_vendas['QUANTIDADE EM PESO VENDIDO'].sum()
```

### **2. Análise de Cliente**
```python
# Buscar vendas do cliente
vendas_cliente = get_vendas_ultimos_6_meses(2206)

# Agrupar por mês
resumo_mensal = vendas_cliente.groupby(['ANO', 'MES']).agg({
    'VALOR_TOTAL': 'sum',
    'PESO_TOTAL': 'sum',
    'NUMPED': 'count'
})
```

### **3. Controle de Estoque**
```python
# Buscar estoque atual
estoque = fetch_oracle_data()

# Filtrar produtos com baixo estoque
baixo_estoque = estoque[estoque['Disponível (Caixas)'] < 5]

# Atualizar planilha
update_google_sheet(estoque)
```

---

## ⚠️ **FILTROS CRÍTICOS**

### **Sempre Aplicar**
- **POSICAO = 'F'**: Apenas pedidos finalizados
- **DTCANCEL IS NULL**: Excluir pedidos cancelados
- **BONIFIC = 'N'**: Excluir bonificações

### **Por Sistema**
- **BRF**: CODEPTO = 103
- **Bem Brasil**: CODEPTO = 102
- **VILOG**: CODFILIAL <> '98'

### **Por Período**
- **Campanhas**: Mês/ano específico
- **Cliente 2206**: Últimos 6 meses
- **VILOG**: Estoque atual

---

## 🔍 **QUERIES MAIS UTILIZADAS**

### **Top 5 Queries**
1. **Vendas por Período** - Campanhas BRF/Bem Brasil
2. **Estoque Atual** - Sistema VILOG
3. **Histórico de Cliente** - Relatório 2206
4. **Ranking de Vendedores** - Análise de performance
5. **Produtos por Departamento** - Categorização

### **Queries de Manutenção**
- Verificar vendedores ativos
- Validar produtos por departamento
- Analisar condições de venda
- Monitorar estoque por filial

---

## 📝 **COMO USAR ESTA DOCUMENTAÇÃO**

### **Para Desenvolvedores**
1. **Identifique a funcionalidade** desejada
2. **Localize o sistema** correspondente
3. **Copie a query** e adapte para suas necessidades
4. **Verifique os filtros** e configurações

### **Para Analistas**
1. **Entenda o contexto** da query
2. **Identifique os campos** retornados
3. **Personalize os filtros** conforme necessário
4. **Teste a query** antes de usar em produção

### **Para Manutenção**
1. **Verifique as configurações** atuais
2. **Atualize listas** de vendedores/produtos
3. **Monitore performance** das queries
4. **Documente mudanças** realizadas

---

## 🤝 **CONTRIBUIÇÃO**

### **Adicionar Novas Queries**
1. **Crie o arquivo** na pasta apropriada
2. **Documente completamente** a query
3. **Atualize este índice** geral
4. **Teste a implementação**

### **Atualizar Queries Existentes**
1. **Modifique o arquivo** correspondente
2. **Atualize a documentação** se necessário
3. **Verifique compatibilidade** com sistemas existentes
4. **Teste as mudanças**

---

**📋 Última Atualização**: Janeiro 2025  
**🔧 Versão**: 1.0  
**📚 Total de Sistemas Documentados**: 5  
**📊 Total de Queries Documentadas**: 15+
