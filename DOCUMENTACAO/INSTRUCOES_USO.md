# 📖 INSTRUÇÕES DE USO - DOCUMENTAÇÃO WINTHOR

## 🎯 **COMO NAVEGAR NA DOCUMENTAÇÃO**

### **1. Comece pelo Índice Geral**
- 📚 **[INDICE_GERAL.md](INDICE_GERAL.md)** - Visão geral de todas as queries
- 🔍 Busca rápida por funcionalidade
- 📊 Organização por tipo de sistema

### **2. Escolha o Sistema de Interesse**
- 🎯 **Campanhas**: BRF ou Bem Brasil
- 📊 **Relatórios**: Cliente 2206 ou Dashboard DICON
- 🔄 **Atualizadores**: VILOG
- 🗄️ **Estrutura**: Tabelas e relacionamentos

### **3. Leia a Documentação Específica**
- 📋 Visão geral do sistema
- 🔍 Queries principais com SQL completo
- 🎯 Configurações específicas
- 📝 Exemplos de implementação

---

## 🚀 **CASOS DE USO PRÁTICOS**

### **Cenário 1: "Preciso criar um relatório de campanha BRF"**
1. **Vá para**: [BRF_CAMPANHA.md](campanhas/BRF_CAMPANHA.md)
2. **Copie a query**: `get_dados_vendas_brf()`
3. **Adapte os parâmetros**: Mês, ano, vendedores
4. **Implemente**: Use o exemplo de código fornecido

### **Cenário 2: "Quero analisar o estoque atual"**
1. **Vá para**: [ATUALIZADOR_VILOG.md](atualizadores/ATUALIZADOR_VILOG.md)
2. **Copie a query principal**: Consulta de estoque e produtos
3. **Adapte os filtros**: Filial, departamento, seção
4. **Implemente**: Use as funções de exemplo

### **Cenário 3: "Preciso entender a estrutura do banco"**
1. **Vá para**: [TABELAS_PRINCIPAIS.md](banco/TABELAS_PRINCIPAIS.md)
2. **Estude as tabelas**: PCPEDC, PCPEDI, PCPRODUT
3. **Entenda os relacionamentos**: Diagrama de chaves
4. **Use os índices recomendados**: Para performance

---

## 🔧 **IMPLEMENTAÇÃO RÁPIDA**

### **Template Básico para Nova Query**
```python
import oracledb
import pandas as pd

# Configurações
DB_CONFIG = {
    'host': '10.0.0.10',
    'port': 1521,
    'service': 'WINT',
    'user': 'dicon',
    'password': 'wdicon01'
}

def minha_query():
    """Descrição da funcionalidade"""
    try:
        # Conectar ao Oracle
        oracledb.init_oracle_client(lib_dir=r"C:\oracle\instantclient_23_9")
        dsn = f"{DB_CONFIG['host']}:{DB_CONFIG['port']}/{DB_CONFIG['service']}"
        connection = oracledb.connect(
            user=DB_CONFIG['user'],
            password=DB_CONFIG['password'],
            dsn=dsn
        )
        
        # Sua query aqui
        sql_query = """
        SELECT 
            -- seus campos aqui
        FROM 
            -- suas tabelas aqui
        WHERE 
            -- seus filtros aqui
        """
        
        # Executar query
        df = pd.read_sql_query(sql_query, connection)
        return df
        
    except Exception as e:
        print(f"Erro: {e}")
        return None
    finally:
        if connection:
            connection.close()
```

---

## 📊 **FILTROS COMUNS E CONFIGURAÇÕES**

### **Filtros Sempre Aplicados**
```sql
-- Para vendas
C.POSICAO = 'F'                    -- Apenas pedidos finalizados
C.DTCANCEL IS NULL                 -- Excluir cancelados
NVL(I.BONIFIC, 'N') = 'N'         -- Excluir bonificações

-- Para estoque
E.CODFILIAL <> '98'                -- Excluir depósito

-- Para campanhas
P.CODEPTO IN (102, 103)            -- Bem Brasil ou BRF
```

### **Configurações de Conexão**
```python
# Sempre usar
oracledb.init_oracle_client(lib_dir=r"C:\oracle\instantclient_23_9")

# Configuração de sessão
cursor.execute("ALTER SESSION SET NLS_NUMERIC_CHARACTERS = '.,'")
```

---

## 🔍 **TROUBLESHOOTING COMUM**

### **Erro: "ORA-12541: TNS:no listener"**
- ✅ Verificar se o IP 10.0.0.10 está acessível
- ✅ Confirmar se a porta 1521 está aberta
- ✅ Verificar se o serviço WINT está ativo

### **Erro: "ORA-12154: TNS:could not resolve the connect identifier"**
- ✅ Verificar se o Oracle Client está instalado
- ✅ Confirmar o caminho: `C:\oracle\instantclient_23_9`
- ✅ Verificar variáveis de ambiente

### **Erro: "ORA-00942: table or view does not exist"**
- ✅ Verificar se o usuário 'dicon' tem permissões
- ✅ Confirmar se as tabelas existem no schema
- ✅ Verificar se está conectando no banco correto

### **Query muito lenta**
- ✅ Verificar se existem índices nas colunas filtradas
- ✅ Usar filtros mais restritivos primeiro
- ✅ Considerar usar hints do Oracle

---

## 📝 **BOAS PRÁTICAS**

### **1. Sempre Fechar Conexões**
```python
try:
    connection = oracledb.connect(...)
    # sua lógica aqui
finally:
    if connection:
        connection.close()
```

### **2. Usar Parâmetros Preparados**
```python
# ❌ Ruim - SQL Injection
sql = f"SELECT * FROM PCPEDC WHERE CODCLI = {codcli}"

# ✅ Bom - Parâmetros preparados
sql = "SELECT * FROM PCPEDC WHERE CODCLI = :codcli"
df = pd.read_sql_query(sql, connection, params={'codcli': codcli})
```

### **3. Tratar Erros Adequadamente**
```python
try:
    df = pd.read_sql_query(sql, connection)
    return df
except Exception as e:
    print(f"Erro ao executar query: {e}")
    return pd.DataFrame()  # Retornar DataFrame vazio
```

### **4. Validar Dados Antes de Usar**
```python
if df is None or df.empty:
    print("Nenhum dado encontrado")
    return None

# Verificar colunas obrigatórias
colunas_obrigatorias = ['CAMPO1', 'CAMPO2']
for coluna in colunas_obrigatorias:
    if coluna not in df.columns:
        print(f"Coluna obrigatória não encontrada: {coluna}")
        return None
```

---

## 🔄 **ATUALIZAÇÃO DA DOCUMENTAÇÃO**

### **Quando Atualizar**
- ✅ Nova funcionalidade implementada
- ✅ Query modificada
- ✅ Configuração alterada
- ✅ Bug corrigido

### **Como Atualizar**
1. **Modifique o arquivo** específico
2. **Atualize o índice geral** se necessário
3. **Teste a implementação**
4. **Documente as mudanças**

### **Padrão de Atualização**
```markdown
**📋 Última Atualização**: [DATA]  
**🔧 Versão**: [VERSÃO]  
**📝 Mudanças**: [DESCRIÇÃO DAS MUDANÇAS]
```

---

## 📞 **SUPORTE E CONTATO**

### **Para Dúvidas Técnicas**
- 📚 Consulte primeiro esta documentação
- 🔍 Use o índice geral para busca
- 📝 Verifique os exemplos de implementação

### **Para Sugestões**
- ✏️ Adicione comentários nos arquivos
- 🔄 Proponha melhorias na estrutura
- 📊 Sugira novas queries para documentar

### **Para Problemas Críticos**
- ⚠️ Verifique a seção de troubleshooting
- 🔧 Confirme as configurações básicas
- 📞 Entre em contato com a equipe técnica

---

## 🎯 **CHECKLIST DE IMPLEMENTAÇÃO**

### **Antes de Começar**
- [ ] Identifiquei o sistema correto na documentação
- [ ] Entendi o contexto da query
- [ ] Verifiquei as configurações necessárias
- [ ] Confirmei os filtros aplicados

### **Durante a Implementação**
- [ ] Usei o template básico fornecido
- [ ] Apliquei os filtros corretos
- [ ] Tratei erros adequadamente
- [ ] Fechei conexões corretamente

### **Após a Implementação**
- [ ] Testei a query com dados reais
- [ ] Validei os resultados
- [ ] Documentei mudanças se necessário
- [ ] Atualizei a documentação se necessário

---

**📋 Arquivo**: INSTRUCOES_USO.md  
**🎯 Objetivo**: Guia prático para uso da documentação  
**📚 Relacionado**: Todos os arquivos de documentação  
**🔧 Última Atualização**: Janeiro 2025
