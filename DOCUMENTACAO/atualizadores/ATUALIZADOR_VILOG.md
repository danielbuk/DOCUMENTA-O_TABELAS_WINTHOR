# 🔄 ATUALIZADOR VILOG - QUERIES ORACLE

## 📋 Visão Geral
Sistema que sincroniza dados de estoque do banco Oracle Winthor com planilha Google Sheets VILOG, atualizando informações de produtos, estoque disponível e custos.

## 🏗️ Estrutura do Sistema
- **Origem**: Banco Oracle Winthor
- **Destino**: Google Sheets VILOG
- **Frequência**: Atualização automática
- **Dados**: Estoque, produtos, custos, departamentos
- **Filtros**: Exclui filial 98 (depósito)

## 🔍 QUERY PRINCIPAL

### 1. 📊 **CONSULTA DE ESTOQUE E PRODUTOS**

#### Descrição
Busca dados completos de estoque e produtos para sincronização com a planilha VILOG, incluindo estoque disponível, custos e informações de classificação.

#### Contexto
- Sincronização de estoque
- Atualização de custos
- Manutenção de catálogo de produtos
- Controle de inventário

#### SQL Completo
```sql
SELECT
  P.CODPROD AS "Código do Produto",
  P.DESCRICAO AS "Nome do Produto",
  (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 0) AS "Disponível (Caixas)",
  P.PESOBRUTO AS "Peso",
  P.EMBALAGEM AS "Embalagem",
  P.CUSTOREP AS "Custo de Reposição",
  D.DESCRICAO AS "Departamento",
  S.DESCRICAO AS "Seção"
FROM
  PCPRODUT P
  JOIN PCEST E ON P.CODPROD = E.CODPROD
  JOIN PCDEPTO D ON P.CODEPTO = D.CODEPTO
  JOIN PCSECAO S ON P.CODSEC = S.CODSEC
WHERE
  E.CODFILIAL <> '98'
```

#### Campos Retornados
- **Código do Produto**: Código único do produto
- **Nome do Produto**: Descrição completa do produto
- **Disponível (Caixas)**: Estoque disponível em caixas
- **Peso**: Peso bruto do produto
- **Embalagem**: Tipo de embalagem
- **Custo de Reposição**: Custo para reposição
- **Departamento**: Nome do departamento
- **Seção**: Nome da seção

#### Cálculos Realizados
```sql
-- Estoque disponível em caixas
(E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 0)

-- Onde:
-- QTESTGER: Estoque geral
-- QTRESERV: Estoque reservado
-- QTBLOQUEADA: Estoque bloqueado
-- QTUNITCX: Quantidade de unidades por caixa
-- NULLIF: Evita divisão por zero
```

#### Filtros Aplicados
- **CODFILIAL <> '98'**: Exclui filial 98 (depósito)
- **Joins obrigatórios**: Produtos, estoque, departamento e seção

---

## 🎯 **CONFIGURAÇÕES ESPECÍFICAS**

### Configurações do Banco
```python
DB_CONFIG = {
    'host': '10.0.0.10',
    'port': 1521,
    'service': 'WINT',
    'user': 'dicon',
    'password': 'wdicon01'
}
```

### Configurações do Google Sheets
```python
GOOGLE_CONFIG = {
    'credentials_file': 'tensile-proxy-468412-t2-dd576fcb8c22.json',
    'spreadsheet_name': 'VILOG',
    'worksheet_name': 'PRODUTOS'
}
```

### Configurações do Oracle Client
```python
ORACLE_CLIENT_PATH = r"C:\oracle\instantclient_23_9"
```

---

## 🔧 **FUNÇÕES PRINCIPAIS**

### 1. **Busca de Dados Oracle**
```python
def fetch_oracle_data():
    """Busca os dados do banco de dados Oracle e retorna um DataFrame."""
    connection = None
    try:
        print("Iniciando conexão com o banco de dados Oracle...")
        
        # Configuração para oracledb
        oracledb.init_oracle_client(lib_dir=ORACLE_CLIENT_PATH)
        
        # String de conexão
        dsn = f"{DB_CONFIG['host']}:{DB_CONFIG['port']}/{DB_CONFIG['service']}"
        connection = oracledb.connect(
            user=DB_CONFIG['user'], 
            password=DB_CONFIG['password'], 
            dsn=dsn
        )
        
        cursor = connection.cursor()
        cursor.execute("ALTER SESSION SET NLS_NUMERIC_CHARACTERS = '.,'")

        print("Conexão bem-sucedida. Executando a consulta SQL...")
        df = pd.read_sql(SQL_QUERY, connection)
        print(f"Consulta SQL retornou {len(df)} linhas de dados.")

        # Arredonda colunas numéricas
        for col in df.select_dtypes(include='number').columns:
            if 'Código do Produto' not in col:
                df[col] = df[col].round(3)

        return df
    except Exception as e:
        print(f"ERRO: Erro ao buscar dados do Oracle: {e}")
        return None
    finally:
        if connection:
            connection.close()
            print("Conexão com o Oracle fechada.")
```

### 2. **Atualização do Google Sheets**
```python
def update_google_sheet(df):
    """Atualiza a planilha do Google com os dados do DataFrame."""
    try:
        print("Iniciando conexão com a API do Google Sheets...")
        gc = gspread.service_account(filename=GOOGLE_CREDENTIALS_FILE)

        print(f"Abrindo a planilha '{SPREADSHEET_NAME}'...")
        spreadsheet = gc.open(SPREADSHEET_NAME)

        worksheet = spreadsheet.worksheet(WORKSHEET_NAME)
        print(f"Aba '{WORKSHEET_NAME}' encontrada. Apagando dados antigos...")
        
        # Limpar dados existentes
        worksheet.clear()
        
        # Inserir novos dados
        set_with_dataframe(worksheet, df)
        
        print("✅ Planilha atualizada com sucesso!")
        return True
    except Exception as e:
        print(f"❌ Erro ao atualizar planilha: {e}")
        return False
```

---

## 📊 **ESTRUTURA DE DADOS**

### DataFrame de Estoque
```python
df_estoque = {
    'Código do Produto': int,           # Código único
    'Nome do Produto': str,              # Descrição
    'Disponível (Caixas)': float,       # Estoque em caixas
    'Peso': float,                       # Peso bruto
    'Embalagem': str,                    # Tipo de embalagem
    'Custo de Reposição': float,         # Custo
    'Departamento': str,                 # Nome do departamento
    'Seção': str                         # Nome da seção
}
```

### Estrutura da Planilha VILOG
```
| Código do Produto | Nome do Produto | Disponível (Caixas) | Peso | Embalagem | Custo de Reposição | Departamento | Seção |
|-------------------|-----------------|---------------------|------|-----------|-------------------|--------------|-------|
| 1001             | Produto A       | 15.5                | 0.5  | CX        | 2.50              | Laticínios   | Leite |
| 1002             | Produto B       | 8.0                 | 1.0  | CX        | 3.75              | Laticínios   | Queijo |
```

---

## 🚀 **CASOS DE USO**

### 1. **Sincronização Diária de Estoque**
- Atualizar estoque disponível
- Manter custos atualizados
- Sincronizar catálogo de produtos

### 2. **Controle de Inventário**
- Monitorar estoque disponível
- Identificar produtos com baixo estoque
- Controlar custos de reposição

### 3. **Relatórios de Gestão**
- Análise de estoque por departamento
- Controle de custos
- Planejamento de compras

---

## ⚠️ **CONSIDERAÇÕES IMPORTANTES**

### Filtros Críticos
- **CODFILIAL <> '98'**: Sempre excluir filial depósito
- **Joins obrigatórios**: Garantir que todos os produtos tenham estoque
- **Divisão por zero**: Usar NULLIF para evitar erros

### Performance
- **Índices**: Verificar índices em CODPROD, CODFILIAL
- **Joins**: Otimizar com índices nas chaves estrangeiras
- **Volume**: Considerar volume de produtos para otimização

### Manutenção
- **Verificar filiais**: Confirmar se filial 98 ainda é depósito
- **Atualizar estrutura**: Estrutura da planilha pode mudar
- **Monitorar performance**: Queries podem ser otimizadas

---

## 📝 **EXEMPLO COMPLETO DE IMPLEMENTAÇÃO**

```python
class AtualizadorVILOG:
    def __init__(self):
        self.connection = None
        self.df_dados = None
    
    def executar_atualizacao(self):
        """Executa atualização completa do VILOG"""
        try:
            # 1. Buscar dados do Oracle
            print("🔄 Iniciando atualização do VILOG...")
            self.df_dados = fetch_oracle_data()
            
            if self.df_dados is None or self.df_dados.empty:
                print("❌ Nenhum dado encontrado para atualização")
                return False
            
            # 2. Validar dados
            if not self.validar_dados():
                print("❌ Dados inválidos encontrados")
                return False
            
            # 3. Atualizar Google Sheets
            if update_google_sheet(self.df_dados):
                print("✅ Atualização concluída com sucesso!")
                return True
            else:
                print("❌ Erro ao atualizar planilha")
                return False
                
        except Exception as e:
            print(f"❌ Erro durante atualização: {e}")
            return False
    
    def validar_dados(self):
        """Valida dados antes da atualização"""
        if self.df_dados is None:
            return False
        
        # Verificar se há dados
        if len(self.df_dados) == 0:
            return False
        
        # Verificar colunas obrigatórias
        colunas_obrigatorias = [
            'Código do Produto', 'Nome do Produto', 'Disponível (Caixas)',
            'Peso', 'Embalagem', 'Custo de Reposição', 'Departamento', 'Seção'
        ]
        
        for coluna in colunas_obrigatorias:
            if coluna not in self.df_dados.columns:
                print(f"❌ Coluna obrigatória não encontrada: {coluna}")
                return False
        
        return True
```

---

## 🔍 **QUERIES ADICIONAIS**

### Query para Análise de Estoque por Departamento
```sql
-- Estoque disponível por departamento
SELECT 
    D.DESCRICAO AS DEPARTAMENTO,
    COUNT(P.CODPROD) AS PRODUTOS,
    SUM((E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 0)) AS CAIXAS_DISPONIVEIS,
    SUM(P.CUSTOREP * (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA)) AS VALOR_ESTOQUE
FROM PCPRODUT P
JOIN PCEST E ON P.CODPROD = E.CODPROD
JOIN PCDEPTO D ON P.CODEPTO = D.CODEPTO
WHERE E.CODFILIAL <> '98'
GROUP BY D.DESCRICAO
ORDER BY VALOR_ESTOQUE DESC;
```

### Query para Produtos com Baixo Estoque
```sql
-- Produtos com estoque baixo (menos de 5 caixas)
SELECT 
    P.CODPROD,
    P.DESCRICAO,
    (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 0) AS CAIXAS_DISPONIVEIS,
    P.CUSTOREP,
    D.DESCRICAO AS DEPARTAMENTO
FROM PCPRODUT P
JOIN PCEST E ON P.CODPROD = E.CODPROD
JOIN PCDEPTO D ON P.CODEPTO = D.CODEPTO
WHERE E.CODFILIAL <> '98'
AND (E.QTESTGER - E.QTRESERV - E.QTBLOQUEADA) / NULLIF(P.QTUNITCX, 0) < 5
ORDER BY CAIXAS_DISPONIVEIS;
```

---

## 🔧 **CONFIGURAÇÕES DE AMBIENTE**

### Variáveis de Ambiente
```bash
# Oracle
ORACLE_HOME=C:\oracle\instantclient_23_9
PATH=%ORACLE_HOME%;%PATH%

# Python
PYTHONPATH=.
```

### Dependências Python
```python
# requirements.txt
oracledb==1.4.1
pandas==2.0.3
gspread==5.12.0
gspread-dataframe==3.3.1
```

---

**📋 Arquivo**: `ATUALIZADOR-VILOG/ATUALIZADOR-VILOG.py`  
**🔄 Funcionalidade**: Sincronização de estoque Oracle com Google Sheets VILOG  
**📊 Dados**: Estoque, produtos, custos, departamentos  
**🔧 Tecnologia**: Oracle, Google Sheets API, Pandas
