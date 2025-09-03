# 📊 RELATÓRIO CLIENTE 2206 - QUERIES ORACLE

## 📋 Visão Geral
Sistema que gera relatórios PDF detalhados para análise de cliente específico (código 2206), incluindo vendas dos últimos 6 meses, análise de produtos, faturamento e custos.

## 🏗️ Estrutura do Sistema
- **Cliente Foco**: Código 2206
- **Período**: Últimos 6 meses
- **Análises**: Vendas, produtos, faturamento, custos, peso
- **Filiais**: 1 e 98
- **Formato**: Relatório PDF com gráficos

## 🔍 QUERIES PRINCIPAIS

### 1. 📋 **INFORMAÇÕES BÁSICAS DO CLIENTE**

#### Descrição
Busca informações básicas do cliente para identificação e cabeçalho do relatório.

#### Contexto
- Identificação do cliente
- Cabeçalho do relatório
- Validação de existência

#### SQL Completo
```sql
SELECT CODCLI, CLIENTE
FROM PCCLIENT 
WHERE CODCLI = {codcli}
```

#### Campos Retornados
- **CODCLI**: Código do cliente
- **CLIENTE**: Nome/razão social do cliente

#### Exemplo de Implementação
```python
def get_cliente_info(codcli):
    """Busca informações básicas do cliente"""
    conn = get_db_connection()
    if not conn:
        return None
    
    sql = f"""
    SELECT CODCLI, CLIENTE
    FROM PCCLIENT 
    WHERE CODCLI = {codcli}
    """
    
    try:
        df = pd.read_sql(sql, conn)
        conn.close()
        return df.iloc[0] if not df.empty else None
    except Exception as e:
        print(f"Erro ao buscar dados do cliente: {e}")
        conn.close()
        return None
```

---

### 2. 📈 **VENDAS DOS ÚLTIMOS 6 MESES**

#### Descrição
Busca vendas detalhadas dos últimos 6 meses para o cliente específico, incluindo produtos, quantidades, preços e custos.

#### Contexto
- Análise de histórico de vendas
- Performance de produtos
- Análise de faturamento e custos
- Tendências temporais

#### SQL Completo
```sql
WITH VENDAS_VALIDAS AS (
    SELECT 
        C.DATA,
        I.CODPROD,
        P.DESCRICAO,
        P.PESOBRUTO,
        I.QT,
        I.PVENDA,
        I.VLCUSTOFIN,
        C.CONDVENDA,
        I.BONIFIC,
        (I.QT * I.PVENDA) AS VALOR_TOTAL,
        (I.QT * I.VLCUSTOFIN) AS CUSTO_TOTAL,
        (I.QT * P.PESOBRUTO) AS PESO_TOTAL
    FROM PCPEDC C 
    JOIN PCPEDI I ON C.NUMPED = I.NUMPED 
    JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
    WHERE C.CODCLI = {codcli}
    AND C.DATA BETWEEN TO_DATE('{data_inicio}', 'DD/MM/YYYY') 
                   AND TO_DATE('{data_fim}', 'DD/MM/YYYY')
    AND C.CODFILIAL IN ('1', '98') 
    AND C.POSICAO = 'F' 
    AND C.DTCANCEL IS NULL 
    AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
    AND NVL(I.BONIFIC, 'N') = 'N'
)
SELECT 
    DATA,
    CODPROD,
    DESCRICAO,
    PESOBRUTO,
    QT,
    PVENDA,
    VLCUSTOFIN,
    CONDVENDA,
    BONIFIC,
    VALOR_TOTAL,
    CUSTO_TOTAL,
    PESO_TOTAL
FROM VENDAS_VALIDAS
ORDER BY DATA DESC, CODPROD
```

#### Campos Retornados
- **DATA**: Data da venda
- **CODPROD**: Código do produto
- **DESCRICAO**: Descrição do produto
- **PESOBRUTO**: Peso bruto do produto
- **QT**: Quantidade vendida
- **PVENDA**: Preço de venda unitário
- **VLCUSTOFIN**: Custo financeiro unitário
- **CONDVENDA**: Condição de venda
- **BONIFIC**: Se é bonificação
- **VALOR_TOTAL**: Valor total da venda (QT × PVENDA)
- **CUSTO_TOTAL**: Custo total (QT × VLCUSTOFIN)
- **PESO_TOTAL**: Peso total (QT × PESOBRUTO)

#### Filtros Aplicados
- **CODCLI = {codcli}**: Cliente específico
- **CODFILIAL IN ('1', '98')**: Filiais específicas
- **POSICAO = 'F'**: Apenas pedidos finalizados
- **DTCANCEL IS NULL**: Pedidos não cancelados
- **CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)**: Condições de venda válidas
- **BONIFIC = 'N'**: Sem bonificação

#### Cálculos Realizados
```python
# Valores calculados na query
VALOR_TOTAL = QT × PVENDA
CUSTO_TOTAL = QT × VLCUSTOFIN
PESO_TOTAL = QT × PESOBRUTO
```

---

### 3. 📊 **RESUMO MENSAL DE VENDAS**

#### Descrição
Agrupa vendas por mês para análise de tendências e performance temporal.

#### Contexto
- Análise de tendências mensais
- Identificação de sazonalidade
- Performance por período

#### SQL Completo
```sql
SELECT 
    EXTRACT(YEAR FROM C.DATA) AS ANO,
    EXTRACT(MONTH FROM C.DATA) AS MES,
    COUNT(DISTINCT C.NUMPED) AS PEDIDOS,
    COUNT(I.CODPROD) AS ITENS,
    SUM(I.QT) AS QUANTIDADE_TOTAL,
    SUM(I.QT * I.PVENDA) AS FATURAMENTO_TOTAL,
    SUM(I.QT * I.VLCUSTOFIN) AS CUSTO_TOTAL,
    SUM(I.QT * P.PESOBRUTO) AS PESO_TOTAL
FROM PCPEDC C 
JOIN PCPEDI I ON C.NUMPED = I.NUMPED 
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
WHERE C.CODCLI = {codcli}
AND C.DATA BETWEEN TO_DATE('{data_inicio}', 'DD/MM/YYYY') 
               AND TO_DATE('{data_fim}', 'DD/MM/YYYY')
AND C.CODFILIAL IN ('1', '98') 
AND C.POSICAO = 'F' 
AND C.DTCANCEL IS NULL 
AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
AND NVL(I.BONIFIC, 'N') = 'N'
GROUP BY EXTRACT(YEAR FROM C.DATA), EXTRACT(MONTH FROM C.DATA)
ORDER BY ANO DESC, MES DESC
```

#### Campos Retornados
- **ANO**: Ano da venda
- **MES**: Mês da venda
- **PEDIDOS**: Quantidade de pedidos únicos
- **ITENS**: Quantidade total de itens
- **QUANTIDADE_TOTAL**: Quantidade total vendida
- **FATURAMENTO_TOTAL**: Faturamento total do mês
- **CUSTO_TOTAL**: Custo total do mês
- **PESO_TOTAL**: Peso total vendido

---

### 4. 🏷️ **ANÁLISE DE PRODUTOS**

#### Descrição
Analisa performance de produtos individuais para o cliente, incluindo ranking de vendas.

#### Contexto
- Identificação de produtos mais vendidos
- Análise de performance por produto
- Estratégias de vendas

#### SQL Completo
```sql
SELECT 
    I.CODPROD,
    P.DESCRICAO,
    P.PESOBRUTO,
    SUM(I.QT) AS QUANTIDADE_TOTAL,
    SUM(I.QT * I.PVENDA) AS FATURAMENTO_TOTAL,
    SUM(I.QT * I.VLCUSTOFIN) AS CUSTO_TOTAL,
    SUM(I.QT * P.PESOBRUTO) AS PESO_TOTAL,
    AVG(I.PVENDA) AS PRECO_MEDIO,
    COUNT(DISTINCT C.NUMPED) AS PEDIDOS
FROM PCPEDC C 
JOIN PCPEDI I ON C.NUMPED = I.NUMPED 
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
WHERE C.CODCLI = {codcli}
AND C.DATA BETWEEN TO_DATE('{data_inicio}', 'DD/MM/YYYY') 
               AND TO_DATE('{data_fim}', 'DD/MM/YYYY')
AND C.CODFILIAL IN ('1', '98') 
AND C.POSICAO = 'F' 
AND C.DTCANCEL IS NULL 
AND C.CONDVENDA NOT IN (4, 8, 10, 13, 20, 98, 99)
AND NVL(I.BONIFIC, 'N') = 'N'
GROUP BY I.CODPROD, P.DESCRICAO, P.PESOBRUTO
ORDER BY FATURAMENTO_TOTAL DESC
```

#### Campos Retornados
- **CODPROD**: Código do produto
- **DESCRICAO**: Descrição do produto
- **PESOBRUTO**: Peso bruto
- **QUANTIDADE_TOTAL**: Quantidade total vendida
- **FATURAMENTO_TOTAL**: Faturamento total do produto
- **CUSTO_TOTAL**: Custo total do produto
- **PESO_TOTAL**: Peso total vendido
- **PRECO_MEDIO**: Preço médio de venda
- **PEDIDOS**: Quantidade de pedidos que contêm o produto

---

## 🎯 **CONFIGURAÇÕES ESPECÍFICAS**

### Cliente Foco
```python
CLIENTE_ANALISE = 2206
```

### Filiais
```python
FILIAIS_VALIDAS = ['1', '98']
```

### Condições de Venda Excluídas
```python
CONDICOES_EXCLUIDAS = [4, 8, 10, 13, 20, 98, 99]
```

### Período de Análise
```python
# Últimos 6 meses
PERIODO_ANALISE = 6  # meses
```

---

## 🔧 **FUNÇÕES AUXILIARES**

### Cálculo de Período
```python
def calcular_periodo_analise():
    """Calcula período dos últimos 6 meses"""
    hoje = date.today()
    data_fim = hoje
    data_inicio = hoje - relativedelta(months=6)
    
    return data_inicio, data_fim
```

### Conexão com Banco
```python
def get_db_connection():
    """Estabelece conexão com o banco Oracle"""
    dsn = f"{DB_CONFIG['host']}:{DB_CONFIG['port']}/{DB_CONFIG['service']}"
    try:
        conn = oracledb.connect(
            user=DB_CONFIG['user'], 
            password=DB_CONFIG['password'], 
            dsn=dsn
        )
        return conn
    except Exception as e:
        print(f"Erro ao conectar ao banco Oracle: {e}")
        return None
```

---

## 📊 **ESTRUTURA DE DADOS**

### DataFrame de Vendas Detalhadas
```python
df_vendas = {
    'DATA': datetime,              # Data da venda
    'CODPROD': int,               # Código do produto
    'DESCRICAO': str,             # Descrição do produto
    'PESOBRUTO': float,           # Peso bruto
    'QT': int,                    # Quantidade
    'PVENDA': float,              # Preço de venda
    'VLCUSTOFIN': float,          # Custo financeiro
    'CONDVENDA': int,             # Condição de venda
    'BONIFIC': str,               # Bonificação
    'VALOR_TOTAL': float,         # Valor total
    'CUSTO_TOTAL': float,         # Custo total
    'PESO_TOTAL': float           # Peso total
}
```

### DataFrame de Resumo Mensal
```python
df_resumo_mensal = {
    'ANO': int,                   # Ano
    'MES': int,                   # Mês
    'PEDIDOS': int,               # Quantidade de pedidos
    'ITENS': int,                 # Quantidade de itens
    'QUANTIDADE_TOTAL': int,      # Quantidade total
    'FATURAMENTO_TOTAL': float,   # Faturamento total
    'CUSTO_TOTAL': float,         # Custo total
    'PESO_TOTAL': float           # Peso total
}
```

### DataFrame de Análise de Produtos
```python
df_produtos = {
    'CODPROD': int,               # Código do produto
    'DESCRICAO': str,             # Descrição
    'PESOBRUTO': float,           # Peso bruto
    'QUANTIDADE_TOTAL': int,      # Quantidade total
    'FATURAMENTO_TOTAL': float,   # Faturamento total
    'CUSTO_TOTAL': float,         # Custo total
    'PESO_TOTAL': float,          # Peso total
    'PRECO_MEDIO': float,         # Preço médio
    'PEDIDOS': int                 # Quantidade de pedidos
}
```

---

## 🚀 **CASOS DE USO**

### 1. **Análise de Performance do Cliente**
- Identificar produtos mais vendidos
- Analisar tendências mensais
- Calcular margem de contribuição

### 2. **Relatório de Gestão**
- Faturamento por período
- Análise de custos
- Performance de produtos

### 3. **Análise Comercial**
- Identificar oportunidades de venda
- Analisar sazonalidade
- Planejar estratégias de vendas

---

## ⚠️ **CONSIDERAÇÕES IMPORTANTES**

### Filtros Críticos
- **CODCLI específico**: Sempre filtrar pelo cliente 2206
- **POSICAO = 'F'**: Apenas pedidos finalizados
- **DTCANCEL IS NULL**: Excluir pedidos cancelados
- **CONDVENDA válidas**: Excluir condições especiais
- **BONIFIC = 'N'**: Excluir bonificações

### Performance
- **Índices**: Verificar índices em CODCLI, DATA, CODFILIAL
- **Joins**: Otimizar com índices nas chaves estrangeiras
- **Filtros**: Aplicar filtros mais restritivos primeiro

### Manutenção
- **Verificar cliente**: Confirmar se código 2206 ainda é válido
- **Atualizar filtros**: Condições de venda podem mudar
- **Monitorar performance**: Queries podem ser otimizadas

---

## 📝 **EXEMPLO COMPLETO DE IMPLEMENTAÇÃO**

```python
class RelatorioCliente2206:
    def __init__(self):
        self.codcli = 2206
        self.data_inicio = None
        self.data_fim = None
    
    def executar_relatorio(self):
        """Executa relatório completo do cliente 2206"""
        try:
            # 1. Informações básicas do cliente
            cliente_info = get_cliente_info(self.codcli)
            if not cliente_info:
                print("Cliente não encontrado")
                return False
            
            # 2. Calcular período de análise
            self.data_inicio, self.data_fim = calcular_periodo_analise()
            
            # 3. Buscar vendas detalhadas
            vendas = get_vendas_ultimos_6_meses(self.codcli)
            
            # 4. Gerar resumo mensal
            resumo_mensal = gerar_resumo_mensal(vendas)
            
            # 5. Análise de produtos
            analise_produtos = gerar_analise_produtos(vendas)
            
            # 6. Gerar relatório PDF
            gerar_relatorio_pdf(cliente_info, vendas, resumo_mensal, analise_produtos)
            
            return True
        except Exception as e:
            print(f"Erro ao executar relatório: {e}")
            return False
```

---

## 🔍 **QUERIES ADICIONAIS**

### Query para Análise de Condições de Venda
```sql
-- Verificar condições de venda utilizadas
SELECT DISTINCT C.CONDVENDA, COUNT(*) AS QUANTIDADE
FROM PCPEDC C 
WHERE C.CODCLI = 2206
AND C.DATA BETWEEN TO_DATE('01/01/2025', 'DD/MM/YYYY') 
               AND TO_DATE('31/12/2025', 'DD/MM/YYYY')
AND C.POSICAO = 'F'
GROUP BY C.CONDVENDA
ORDER BY QUANTIDADE DESC;
```

### Query para Análise de Filiais
```sql
-- Verificar distribuição por filial
SELECT C.CODFILIAL, COUNT(*) AS PEDIDOS, SUM(I.QT * I.PVENDA) AS FATURAMENTO
FROM PCPEDC C 
JOIN PCPEDI I ON C.NUMPED = I.NUMPED
WHERE C.CODCLI = 2206
AND C.DATA BETWEEN TO_DATE('01/01/2025', 'DD/MM/YYYY') 
               AND TO_DATE('31/12/2025', 'DD/MM/YYYY')
AND C.POSICAO = 'F'
GROUP BY C.CODFILIAL
ORDER BY FATURAMENTO DESC;
```

---

**📋 Arquivo**: `DASHBOARD_DICON/relatorio_cliente_2206.py`  
**📊 Funcionalidade**: Relatório detalhado de cliente específico com análise de vendas  
**📈 Dados**: Vendas, produtos, faturamento, custos, análise temporal  
**🔧 Tecnologia**: Oracle, Pandas, Matplotlib, ReportLab
