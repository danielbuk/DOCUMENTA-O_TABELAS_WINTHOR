# 🇧🇷 SISTEMA DE CAMPANHA BEM BRASIL - QUERIES ORACLE

## 📋 Visão Geral
Sistema que gera relatórios PDF para análise de campanha Bem Brasil com análise de positivação de CNPJ, crescimento em kg e análise de produtos específicos (Carinha e Anel de Cebola).

## 🏗️ Estrutura do Sistema
- **Departamento**: 102 (Bem Brasil)
- **Vendedores**: 20 vendedores específicos
- **Análises**: Positivação CNPJ, crescimento em kg, produtos específicos
- **Período**: Flexível com seleção de mês e período de análise

## 🔍 QUERIES PRINCIPAIS

### 1. 📊 **POSITIVAÇÃO DE CNPJ**

#### Descrição
Analisa a positivação de CNPJ comparando período de referência com período atual, calculando crescimento percentual e elegibilidade.

#### Contexto
- Análise de desenvolvimento de clientes
- Identificação de vendedores com crescimento
- Controle de performance por CNPJ

#### SQL Completo
```sql
SELECT
    U.CODUSUR AS CODIGO_RCA, 
    U.NOME AS NOME_VENDEDOR,
    COUNT(DISTINCT CASE WHEN EXTRACT(MONTH FROM C.DATA) = {mes_ref} AND EXTRACT(YEAR FROM C.DATA) = {ano_ref} THEN C.CODCLI END) AS REFERENCIA,
    COUNT(DISTINCT CASE WHEN C.DATA BETWEEN TO_DATE('{data_inicio}', 'DD/MM/YYYY') 
                                       AND TO_DATE('{data_fim}', 'DD/MM/YYYY') THEN C.CODCLI END) AS ATUAL
FROM PCPEDC C
JOIN PCPEDI I ON C.NUMPED = I.NUMPED
JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
WHERE C.DATA BETWEEN TO_DATE('01/{mes_ref:02d}/{ano_ref}', 'DD/MM/YYYY') 
                  AND TO_DATE('{data_fim}', 'DD/MM/YYYY')
AND P.CODEPTO = 102
AND U.CODUSUR IN (33, 66, 6, 41, 63, 15, 31, 111, 86, 130, 108, 9, 40, 91, 118, 23, 1, 2, 12, 51)
AND C.CODFILIAL IN ('1', '98') 
AND C.POSICAO = 'F' 
AND C.DTCANCEL IS NULL
AND NVL(I.BONIFIC, 'N') = 'N'
AND C.CONDVENDA IN (1, 2, 3, 7, 9, 14, 15, 17, 18, 19, 98)
GROUP BY U.CODUSUR, U.NOME
ORDER BY U.NOME
```

#### Campos Retornados
- **CODIGO_RCA**: Código do vendedor
- **NOME_VENDEDOR**: Nome completo do vendedor
- **REFERENCIA**: Quantidade de CNPJs no período de referência
- **ATUAL**: Quantidade de CNPJs no período atual
- **CRESCIMENTO_%**: Percentual de crescimento (calculado)
- **ELEGIVEL**: Se atingiu meta de 5% de crescimento

#### Filtros Aplicados
- **DEPARTAMENTO = 102**: Apenas produtos Bem Brasil
- **POSICAO = 'F'**: Apenas pedidos finalizados
- **DTCANCEL IS NULL**: Pedidos não cancelados
- **BONIFIC = 'N'**: Sem bonificação
- **CONDVENDA específicas**: Condições de venda válidas
- **CODFILIAL IN ('1', '98')**: Filiais específicas

#### Cálculos Realizados
```python
# Crescimento percentual
df['CRESCIMENTO_%'] = np.where(
    df['REFERENCIA'] > 0,
    ((df['ATUAL'] - df['REFERENCIA']) / df['REFERENCIA']) * 100,
    np.where(df['ATUAL'] > 0, 100.0, 0.0)
)

# Elegibilidade (meta: 5% de crescimento)
df['ELEGIVEL'] = np.where(df['CRESCIMENTO_%'] >= 5, 'Sim', 'Não')
```

---

### 2. 🥔 **CARINHA E ANEL DE CEBOLA**

#### Descrição
Analisa a performance de produtos específicos (Carinha e Anel de Cebola) nos últimos 3 meses, excluindo o mês fechado.

#### Contexto
- Análise de produtos específicos da campanha
- Performance mensal comparativa
- Identificação de tendências

#### SQL Completo
```sql
SELECT
    U.CODUSUR AS CODIGO_RCA,
    U.NOME AS NOME_VENDEDOR,
    {case_statements}
FROM PCPEDC C
JOIN PCPEDI I ON C.NUMPED = I.NUMPED
JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
WHERE C.DATA BETWEEN TO_DATE('01/{mes_inicio:02d}/{ano_inicio}', 'DD/MM/YYYY') 
                  AND TO_DATE('31/{mes_fim:02d}/{ano_fim}', 'DD/MM/YYYY')
AND P.CODEPTO = 102
AND P.CODPROD IN (1003, 1215, 1315, 1316, 1317, 1318, 1319, 1320, 1321, 1339, 1408, 1880, 1914, 1984, 2188, 2189, 2190, 2336, 2337, 2391, 2449, 2450, 2451, 2452, 2453, 2454, 2455, 2465, 2557, 320, 356, 76, 80)
AND U.CODUSUR IN (33, 66, 6, 41, 63, 15, 31, 111, 86, 130, 108, 9, 40, 91, 118, 23, 1, 2, 12, 51)
AND C.CODFILIAL IN ('1', '98') 
AND C.POSICAO = 'F' 
AND C.DTCANCEL IS NULL
AND NVL(I.BONIFIC, 'N') = 'N'
AND C.CONDVENDA IN (1, 2, 3, 7, 9, 14, 15, 17, 18, 19, 98)
GROUP BY U.CODUSUR, U.NOME
ORDER BY U.NOME
```

#### Case Statements Dinâmicos
```python
# Construir case statements dinamicamente
case_statements = []
for i, (mes, ano) in enumerate(meses_ref):
    case_statements.append(
        f"COUNT(DISTINCT CASE WHEN EXTRACT(MONTH FROM C.DATA) = {mes} AND EXTRACT(YEAR FROM C.DATA) = {ano} THEN C.CODCLI END) AS MES_{i+1}"
    )
```

#### Campos Retornados
- **CODIGO_RCA**: Código do vendedor
- **NOME_VENDEDOR**: Nome completo do vendedor
- **MES_1, MES_2, MES_3**: Quantidade de CNPJs por mês

---

### 3. 📈 **CRESCIMENTO EM KG**

#### Descrição
Analisa o crescimento em peso (kg) dos produtos Bem Brasil comparando período de referência com período atual.

#### Contexto
- Análise de volume de vendas
- Comparação de performance por peso
- Identificação de crescimento real

#### SQL Completo
```sql
SELECT
    U.CODUSUR AS CODIGO_RCA,
    U.NOME AS NOME_VENDEDOR,
    SUM(CASE WHEN EXTRACT(MONTH FROM C.DATA) = {mes_ref} AND EXTRACT(YEAR FROM C.DATA) = {ano_ref} 
             THEN (I.QT * P.PESOBRUTO) ELSE 0 END) AS REFERENCIA_KG,
    SUM(CASE WHEN C.DATA BETWEEN TO_DATE('{data_inicio}', 'DD/MM/YYYY') 
                           AND TO_DATE('{data_fim}', 'DD/MM/YYYY') 
             THEN (I.QT * P.PESOBRUTO) ELSE 0 END) AS ATUAL_KG
FROM PCPEDC C
JOIN PCPEDI I ON C.NUMPED = I.NUMPED
JOIN PCUSUARI U ON C.CODUSUR = U.CODUSUR
JOIN PCPRODUT P ON I.CODPROD = P.CODPROD
WHERE C.DATA BETWEEN TO_DATE('01/{mes_ref:02d}/{ano_ref}', 'DD/MM/YYYY') 
                  AND TO_DATE('{data_fim}', 'DD/MM/YYYY')
AND P.CODEPTO = 102
AND U.CODUSUR IN (33, 66, 6, 41, 63, 15, 31, 111, 86, 130, 108, 9, 40, 91, 118, 23, 1, 2, 12, 51)
AND C.CODFILIAL IN ('1', '98') 
AND C.POSICAO = 'F' 
AND C.DTCANCEL IS NULL
AND NVL(I.BONIFIC, 'N') = 'N'
AND C.CONDVENDA IN (1, 2, 3, 7, 9, 14, 15, 17, 18, 19, 98)
GROUP BY U.CODUSUR, U.NOME
ORDER BY U.NOME
```

#### Campos Retornados
- **CODIGO_RCA**: Código do vendedor
- **NOME_VENDEDOR**: Nome completo do vendedor
- **REFERENCIA_KG**: Peso total no período de referência
- **ATUAL_KG**: Peso total no período atual
- **CRESCIMENTO_KG**: Diferença em kg (calculado)
- **CRESCIMENTO_%**: Percentual de crescimento (calculado)

---

## 🎯 **CONFIGURAÇÕES ESPECÍFICAS**

### Vendedores Selecionados
```python
VENDEDORES_BEM_BRASIL = [
    33, 66, 6, 41, 63, 15, 31, 111, 86, 130,  # Primeira linha
    108, 9, 40, 91, 118, 23, 1, 2, 12, 51     # Segunda linha
]
```

### Produtos Bem Brasil (Departamento 102)
```python
PRODUTOS_BEM_BRASIL = [
    1003, 1215, 1315, 1316, 1317, 1318, 1319, 1320, 1321, 1339,  # Primeira linha
    1408, 1880, 1914, 1984, 2188, 2189, 2190, 2336, 2337, 2391,  # Segunda linha
    2449, 2450, 2451, 2452, 2453, 2454, 2455, 2465, 2557, 320,   # Terceira linha
    356, 76, 80                                                      # Quarta linha
]
```

### Condições de Venda Válidas
```python
CONDICOES_VENDA_VALIDAS = [1, 2, 3, 7, 9, 14, 15, 17, 18, 19, 98]
```

---

## 🔧 **FUNÇÕES AUXILIARES**

### Cálculo de Períodos
```python
def calcular_periodo_referencia(self):
    """Calcula período de referência para comparação"""
    if self.mes_selecionado == 7:  # Julho
        mes_ref = 7
        ano_ref = self.ano_selecionado - 1
    else:
        mes_ref = self.mes_selecionado - 1
        ano_ref = self.ano_selecionado
    
    return mes_ref, ano_ref

def calcular_periodo_analise(self):
    """Calcula período de análise atual"""
    if self.mes_fim and self.mes_fim > self.mes_selecionado:
        # Período de dois meses
        data_inicio = f"01/{self.mes_selecionado:02d}/{self.ano_selecionado}"
        data_fim = f"31/{self.mes_fim:02d}/{self.ano_selecionado}"
    else:
        # Apenas um mês
        data_inicio = f"01/{self.mes_selecionado:02d}/{self.ano_selecionado}"
        data_fim = f"31/{self.mes_selecionado:02d}/{self.ano_selecionado}"
    
    return data_inicio, data_fim
```

### Formatação de Dados
```python
def formatar_numero_brasil(self, valor):
    """Formata número para padrão brasileiro (1.234,56)"""
    if pd.isna(valor) or valor == 0:
        return "0,00"
    return f"{valor:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")

def ajustar_media_referencial(self, valor):
    """Ajusta média referencial: valores pequenos viram 1 para análise justa"""
    if pd.isna(valor) or valor == 0:
        return 0
    if valor < 1:
        return 1
    return valor
```

---

## 📊 **ESTRUTURA DE DADOS**

### DataFrame de Positivação CNPJ
```python
df_cnpj = {
    'CODIGO_RCA': int,           # Código do vendedor
    'NOME_VENDEDOR': str,        # Nome completo
    'REFERENCIA': int,           # CNPJs no período ref
    'ATUAL': int,                # CNPJs no período atual
    'CRESCIMENTO_%': float,      # Percentual de crescimento
    'ELEGIVEL': str              # 'Sim' ou 'Não'
}
```

### DataFrame de Produtos Específicos
```python
df_produtos = {
    'CODIGO_RCA': int,           # Código do vendedor
    'NOME_VENDEDOR': str,        # Nome completo
    'MES_1': int,                # CNPJs no mês 1
    'MES_2': int,                # CNPJs no mês 2
    'MES_3': int                 # CNPJs no mês 3
}
```

### DataFrame de Crescimento KG
```python
df_crescimento = {
    'CODIGO_RCA': int,           # Código do vendedor
    'NOME_VENDEDOR': str,        # Nome completo
    'REFERENCIA_KG': float,      # Peso no período ref
    'ATUAL_KG': float,           # Peso no período atual
    'CRESCIMENTO_KG': float,     # Diferença em kg
    'CRESCIMENTO_%': float       # Percentual de crescimento
}
```

---

## 🚀 **CASOS DE USO**

### 1. **Análise de Positivação CNPJ**
- Identificar vendedores com crescimento de clientes
- Controlar meta de 5% de crescimento
- Acompanhar desenvolvimento de carteira

### 2. **Análise de Produtos Específicos**
- Monitorar performance de produtos da campanha
- Identificar tendências mensais
- Ajustar estratégias por produto

### 3. **Análise de Crescimento em KG**
- Medir crescimento real em volume
- Comparar performance por peso
- Identificar vendedores com maior volume

---

## ⚠️ **CONSIDERAÇÕES IMPORTANTES**

### Filtros Críticos
- **DEPARTAMENTO = 102**: Garantir que são produtos Bem Brasil
- **POSICAO = 'F'**: Apenas pedidos finalizados
- **DTCANCEL IS NULL**: Excluir pedidos cancelados
- **BONIFIC = 'N'**: Excluir bonificações
- **CONDVENDA específicas**: Apenas condições válidas

### Performance
- **Índices**: Verificar índices em DATA, CODEPTO, CODPROD, CODFILIAL
- **Joins**: Otimizar com índices nas chaves estrangeiras
- **Filtros**: Aplicar filtros mais restritivos primeiro

### Manutenção
- **Atualizar listas**: Vendedores e produtos podem mudar
- **Verificar condições**: Condições de venda podem ser alteradas
- **Monitorar performance**: Queries podem ser otimizadas

---

## 📝 **EXEMPLO COMPLETO DE IMPLEMENTAÇÃO**

```python
class SistemaCampanhaBemBrasil:
    def __init__(self):
        self.connection = None
        self.mes_selecionado = None
        self.mes_fim = None
        self.ano_selecionado = None
        self.periodo_analise = None
    
    def executar_analise_completa(self):
        """Executa análise completa da campanha Bem Brasil"""
        if not self.conectar_oracle():
            return False
        
        try:
            # 1. Análise de positivação CNPJ
            dados_cnpj = self.get_positivacao_cnpj()
            
            # 2. Análise de produtos específicos
            dados_produtos = self.get_positivacao_carinha_anel()
            
            # 3. Análise de crescimento em KG
            dados_crescimento = self.get_crescimento_kg()
            
            # 4. Gerar relatório consolidado
            self.gerar_relatorio_completo(dados_cnpj, dados_produtos, dados_crescimento)
            
            return True
        except Exception as e:
            print(f"❌ Erro ao executar análise: {e}")
            return False
        finally:
            if self.connection:
                self.connection.close()
    
    def gerar_relatorio_completo(self, df_cnpj, df_produtos, df_crescimento):
        """Gera relatório PDF consolidado"""
        # Implementar geração de relatório
        pass
```

---

## 🔍 **QUERIES ADICIONAIS**

### Query para Análise de Filtros
```sql
-- Verificar produtos disponíveis no departamento
SELECT DISTINCT CODPROD, DESCRICAO, PESOBRUTO
FROM PCPRODUT 
WHERE CODEPTO = 102
ORDER BY CODPROD;

-- Verificar vendedores ativos
SELECT CODUSUR, NOME, ATIVO
FROM PCUSUARI 
WHERE CODUSUR IN (33, 66, 6, 41, 63, 15, 31, 111, 86, 130, 108, 9, 40, 91, 118, 23, 1, 2, 12, 51)
ORDER BY NOME;
```

---

**📋 Arquivo**: `BEM BRASIL CAMPANHA/sistema_campanha_bem_brasil.py`  
**🇧🇷 Funcionalidade**: Sistema de campanha Bem Brasil com análise de CNPJ, produtos e crescimento  
**📊 Dados**: Positivação de clientes, produtos específicos, crescimento em kg  
**🔧 Tecnologia**: Oracle, Pandas, ReportLab, Matplotlib
