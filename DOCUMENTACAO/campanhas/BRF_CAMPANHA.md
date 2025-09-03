# 🎯 SISTEMA DE CAMPANHA BRF - QUERIES ORACLE

## 📋 Visão Geral
Sistema que gera relatórios PDF para análise de campanha BRF com seleção de mês específico. Meta: 170 toneladas.

## 🏗️ Estrutura do Sistema
- **Período**: Julho, Agosto e Setembro de 2025
- **Meta**: 170.000 kg (170 toneladas)
- **Departamento**: 103 (BRF)
- **Vendedores**: 20 vendedores específicos
- **Produtos**: 33 produtos selecionados

## 🔍 QUERIES PRINCIPAIS

### 1. 📊 **DADOS DE VENDAS BRF POR MÊS**

#### Descrição
Busca dados de vendas BRF para um mês específico, agrupando por vendedor e calculando quantidade em peso vendido.

#### Contexto
- Análise mensal de performance
- Ranking de vendedores
- Controle de metas

#### SQL Completo
```sql
SELECT
    U.NOME AS "NOME DO VENDEDOR",
    SUM(I.QT * P.PESOLIQ) AS "QUANTIDADE EM PESO VENDIDO"
FROM
    PCPEDC C
JOIN
    PCPEDI I ON C.NUMPED = I.NUMPED
JOIN
    PCPRODUT P ON I.CODPROD = P.CODPROD
JOIN
    PCUSUARI U ON C.CODUSUR = U.CODUSUR
WHERE
    C.POSICAO = 'F'
    AND EXTRACT(YEAR FROM C.DATA) = {ano}
    AND EXTRACT(MONTH FROM C.DATA) = {mes}
    AND P.CODEPTO = 103 -- DEPARTAMENTO BRF
    AND U.CODUSUR IN (33, 66, 6, 41, 63, 15, 31, 111, 86, 130, 108, 9, 40, 91, 118, 23, 1, 2, 12, 51) -- VENDEDORES SELECIONADOS
    AND P.CODPROD IN (1003, 1215, 1315, 1316, 1317, 1318, 1319, 1320, 1321, 1339, 1408, 1880, 1914, 1984, 2188, 2189, 2190, 2336, 2337, 2391, 2449, 2450, 2451, 2452, 2453, 2454, 2455, 2465, 2557, 320, 356, 76, 80) -- PRODUTOS SELECIONADOS
GROUP BY
    U.NOME
ORDER BY
    "QUANTIDADE EM PESO VENDIDO" DESC
```

#### Campos Retornados
- **NOME DO VENDEDOR**: Nome completo do vendedor
- **QUANTIDADE EM PESO VENDIDO**: Soma total em kg (QT × PESOLIQ)

#### Filtros Aplicados
- **POSICAO = 'F'**: Apenas pedidos finalizados
- **DEPARTAMENTO = 103**: Apenas produtos BRF
- **Vendedores específicos**: 20 vendedores pré-selecionados
- **Produtos específicos**: 33 produtos da campanha BRF
- **Período**: Mês e ano específicos

#### Exemplo de Implementação
```python
def get_dados_vendas_brf(self, mes=None, ano=None):
    if mes is None:
        mes = self.mes_selecionado
    if ano is None:
        ano = self.ano_selecionado
        
    sql_query = f"""
    SELECT
        U.NOME AS "NOME DO VENDEDOR",
        SUM(I.QT * P.PESOLIQ) AS "QUANTIDADE EM PESO VENDIDO"
    FROM
        PCPEDC C
    JOIN
        PCPEDI I ON C.NUMPED = I.NUMPED
    JOIN
        PCPRODUT P ON I.CODPROD = P.CODPROD
    JOIN
        PCUSUARI U ON C.CODUSUR = U.CODUSUR
    WHERE
        C.POSICAO = 'F'
        AND EXTRACT(YEAR FROM C.DATA) = {ano}
        AND EXTRACT(MONTH FROM C.DATA) = {mes}
        AND P.CODEPTO = 103
        AND U.CODUSUR IN (33, 66, 6, 41, 63, 15, 31, 111, 86, 130, 108, 9, 40, 91, 118, 23, 1, 2, 12, 51)
        AND P.CODPROD IN (1003, 1215, 1315, 1316, 1317, 1318, 1319, 1320, 1321, 1339, 1408, 1880, 1914, 1984, 2188, 2189, 2190, 2336, 2337, 2391, 2449, 2450, 2451, 2452, 2453, 2454, 2455, 2465, 2557, 320, 356, 76, 80)
    GROUP BY
        U.NOME
    ORDER BY
        "QUANTIDADE EM PESO VENDIDO" DESC
    """
    
    return pd.read_sql_query(sql_query, self.connection)
```

---

### 2. 📈 **DADOS DOS TRÊS MESES (CONSOLIDADO)**

#### Descrição
Busca dados consolidados dos três meses (Julho, Agosto, Setembro) para cálculo de totais e análise geral da campanha.

#### Contexto
- Análise consolidada da campanha
- Cálculo de totais
- Comparação de performance geral

#### SQL Completo
```sql
SELECT
    U.NOME AS "NOME DO VENDEDOR",
    SUM(I.QT * P.PESOLIQ) AS "QUANTIDADE EM PESO VENDIDO"
FROM
    PCPEDC C
JOIN
    PCPEDI I ON C.NUMPED = I.NUMPED
JOIN
    PCPRODUT P ON I.CODPROD = P.CODPROD
JOIN
    PCUSUARI U ON C.CODUSUR = U.CODUSUR
WHERE
    C.POSICAO = 'F'
    AND EXTRACT(YEAR FROM C.DATA) = 2025
    AND EXTRACT(MONTH FROM C.DATA) IN (7, 8, 9) -- Julho, Agosto, Setembro
    AND P.CODEPTO = 103 -- DEPARTAMENTO BRF
    AND U.CODUSUR IN (33, 66, 6, 41, 63, 15, 31, 111, 86, 130, 108, 9, 40, 91, 118, 23, 1, 2, 12, 51) -- VENDEDORES SELECIONADOS
    AND P.CODPROD IN (1003, 1215, 1315, 1316, 1317, 1318, 1319, 1320, 1321, 1339, 1408, 1880, 1914, 1984, 2188, 2189, 2190, 2336, 2337, 2391, 2449, 2450, 2451, 2452, 2453, 2454, 2455, 2465, 2557, 320, 356, 76, 80) -- PRODUTOS SELECIONADOS
GROUP BY
    U.NOME
ORDER BY
    "QUANTIDADE EM PESO VENDIDO" DESC
```

#### Diferenças da Query Mensal
- **Período fixo**: Sempre os três meses (7, 8, 9)
- **Ano fixo**: Sempre 2025
- **Consolidação**: Soma de todos os três meses

---

## 🎯 **CONFIGURAÇÕES ESPECÍFICAS**

### Vendedores Selecionados
```python
VENDEDORES_BRF = [
    33, 66, 6, 41, 63, 15, 31, 111, 86, 130,  # Primeira linha
    108, 9, 40, 91, 118, 23, 1, 2, 12, 51     # Segunda linha
]
```

### Produtos BRF (Departamento 103)
```python
PRODUTOS_BRF = [
    1003, 1215, 1315, 1316, 1317, 1318, 1319, 1320, 1321, 1339,  # Primeira linha
    1408, 1880, 1914, 1984, 2188, 2189, 2190, 2336, 2337, 2391,  # Segunda linha
    2449, 2450, 2451, 2452, 2453, 2454, 2455, 2465, 2557, 320,   # Terceira linha
    356, 76, 80                                                      # Quarta linha
]
```

### Meta da Campanha
```python
META_BRF = 170000  # 170 toneladas em kg
```

---

## 🔧 **FUNÇÕES AUXILIARES**

### Formatação de Números
```python
def formatar_numero_brasil(self, valor):
    """Formata número para padrão brasileiro (1.234,56)"""
    if pd.isna(valor) or valor == 0:
        return "0,00"
    return f"{valor:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")

def formatar_peso_kg(self, valor):
    """Formata peso em kg com separadores brasileiros"""
    if pd.isna(valor) or valor == 0:
        return "0,00 kg"
    
    valor_float = float(valor)
    
    if valor_float.is_integer():
        return f"{valor_float:,.0f}".replace(",", ".") + " kg"
    else:
        return f"{valor_float:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".") + " kg"
```

### Nomes dos Meses
```python
def obter_nome_mes(self, mes):
    """Converte número do mês para nome"""
    nomes_meses = {
        1: 'Janeiro', 2: 'Fevereiro', 3: 'Março', 4: 'Abril',
        5: 'Maio', 6: 'Junho', 7: 'Julho', 8: 'Agosto',
        9: 'Setembro', 10: 'Outubro', 11: 'Novembro', 12: 'Dezembro'
    }
    return nomes_meses.get(mes, f'Mês {mes}')
```

---

## 📊 **ESTRUTURA DE DADOS**

### DataFrame de Vendas
```python
# Estrutura retornada pelas queries
df_vendas = {
    'NOME DO VENDEDOR': str,           # Nome completo do vendedor
    'QUANTIDADE EM PESO VENDIDO': float # Total em kg
}
```

### Cálculos Realizados
1. **Peso Total**: `QT × PESOLIQ` (Quantidade × Peso Líquido)
2. **Ranking**: Ordenação por peso vendido (decrescente)
3. **Filtros**: Apenas pedidos finalizados e produtos BRF

---

## 🚀 **CASOS DE USO**

### 1. **Relatório Mensal Individual**
- Selecionar mês específico
- Gerar ranking de vendedores
- Calcular performance individual

### 2. **Relatório Consolidado**
- Análise dos três meses
- Total geral da campanha
- Comparação com meta (170 toneladas)

### 3. **Análise de Performance**
- Identificar top performers
- Detectar vendedores com baixa performance
- Ajustar estratégias de campanha

---

## ⚠️ **CONSIDERAÇÕES IMPORTANTES**

### Filtros Críticos
- **POSICAO = 'F'**: Sempre verificar se o pedido está finalizado
- **DEPARTAMENTO = 103**: Garantir que são produtos BRF
- **Vendedores específicos**: Lista fixa de 20 vendedores
- **Produtos específicos**: Lista fixa de 33 produtos

### Performance
- **Índices**: Verificar se existem índices em DATA, CODEPTO, CODPROD
- **Joins**: Otimizar com índices nas chaves estrangeiras
- **Filtros**: Aplicar filtros mais restritivos primeiro

### Manutenção
- **Atualizar listas**: Vendedores e produtos podem mudar
- **Verificar metas**: Meta de 170 toneladas pode ser ajustada
- **Monitorar performance**: Queries podem ser otimizadas

---

## 📝 **EXEMPLO COMPLETO DE IMPLEMENTAÇÃO**

```python
class SistemaCampanhaBRF:
    def __init__(self):
        self.connection = None
        self.mes_selecionado = None
        self.ano_selecionado = None
        self.periodo_analise = None
        self.calcular_tres_meses = False
    
    def conectar_oracle(self):
        """Conecta ao banco Oracle"""
        try:
            oracledb.init_oracle_client(lib_dir=r"C:\oracle\instantclient_23_9")
            dsn = f"{DB_HOST}:{DB_PORT}/{DB_SERVICE}"
            self.connection = oracledb.connect(
                user=DB_USER,
                password=DB_PASSWORD,
                dsn=dsn
            )
            print("✅ Conectado ao Oracle com sucesso!")
            return True
        except Exception as e:
            print(f"❌ Erro ao conectar ao Oracle: {e}")
            return False
    
    def executar_campanha(self):
        """Executa análise completa da campanha"""
        if not self.conectar_oracle():
            return False
        
        try:
            # Buscar dados mensais
            dados_mensais = self.get_dados_vendas_brf()
            
            # Buscar dados consolidados
            dados_consolidados = self.get_dados_tres_meses()
            
            # Gerar relatório
            self.gerar_relatorio_pdf(dados_mensais, dados_consolidados)
            
            return True
        except Exception as e:
            print(f"❌ Erro ao executar campanha: {e}")
            return False
        finally:
            if self.connection:
                self.connection.close()
```

---

**📋 Arquivo**: `BRF_CAMPANHA/sistema_campanha_brf.py`  
**🎯 Funcionalidade**: Sistema de campanha BRF com análise mensal e consolidada  
**📊 Dados**: Vendas por vendedor, produtos BRF, período específico  
**🔧 Tecnologia**: Oracle, Pandas, ReportLab, Matplotlib
