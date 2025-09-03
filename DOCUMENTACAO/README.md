# 📚 DOCUMENTAÇÃO COMPLETA DO SISTEMA WINTHOR DICON

## 🎯 Visão Geral
Esta documentação contém todas as queries Oracle utilizadas no sistema Winthor da DICON, organizadas por funcionalidade e módulo. Cada query está documentada com sua finalidade, estrutura e contexto de uso.

## 🏗️ Estrutura do Sistema
- **Sistema Principal**: Winthor Oracle (IP: 10.0.0.10, Porta: 1521, Service: WINT)
- **Usuário**: dicon
- **Banco**: Sistema ERP completo para gestão de distribuição
- **Oracle Client**: Instant Client 23.9

## 📁 Módulos Documentados

### 1. 🎯 **SISTEMAS DE CAMPANHA**
- [Campanha BRF](campanhas/BRF_CAMPANHA.md)
- [Campanha Bem Brasil](campanhas/BEM_BRASIL_CAMPANHA.md)

### 2. 📊 **DASHBOARDS E RELATÓRIOS**
- [Dashboard DICON](dashboards/DASHBOARD_DICON.md)
- [Relatório Cliente 2206](dashboards/RELATORIO_CLIENTE_2206.md)
- [Sistema de Separação](sistemas/SISTEMA_SEPARACAO.md)

### 3. 🔄 **ATUALIZADORES E AUTOMAÇÕES**
- [Atualizador VILOG](atualizadores/ATUALIZADOR_VILOG.md)
- [Catálogo Winthor](automacoes/CATALOGO_WINTHOR.md)

### 4. 📈 **ANÁLISES E RELATÓRIOS**
- [Análise de Compras](analises/ANALISE_COMPRAS.md)
- [Produtos sem Venda](analises/PRODUTOS_SEM_VENDA.md)
- [Falta de Produtos](analises/FALTA_PRODUTOS.md)

### 5. 🗄️ **ESTRUTURA DO BANCO**
- [Tabelas Principais](banco/TABELAS_PRINCIPAIS.md)
- [Relacionamentos](banco/RELACIONAMENTOS.md)
- [Campos Importantes](banco/CAMPOS_IMPORTANTES.md)

## 🔍 Como Usar Esta Documentação

### Para Desenvolvedores
1. **Busque por funcionalidade**: Use o índice para encontrar queries relacionadas
2. **Analise a estrutura**: Cada query tem explicação detalhada dos campos
3. **Copie e adapte**: Use as queries como base para novas funcionalidades

### Para Analistas
1. **Entenda o contexto**: Cada query explica o que está sendo analisado
2. **Identifique filtros**: Veja quais condições são aplicadas
3. **Personalize**: Adapte as queries para suas necessidades específicas

## 🚀 Queries Mais Utilizadas

### Top 5 Queries Principais
1. **Vendas por Período**: Análise de vendas por mês/ano
2. **Estoque Atual**: Consulta de estoque disponível
3. **Clientes por Vendedor**: Distribuição de clientes
4. **Produtos por Departamento**: Categorização de produtos
5. **Análise de Campanhas**: Performance de campanhas específicas

## 📊 Padrões de Nomenclatura

### Tabelas Principais
- **PCPEDC**: Pedidos de venda (cabeçalho)
- **PCPEDI**: Itens dos pedidos
- **PCPRODUT**: Produtos/cadastro
- **PCEST**: Estoque
- **PCCLIENT**: Clientes
- **PCUSUARI**: Usuários/vendedores
- **PCDEPTO**: Departamentos
- **PCSECAO**: Seções

### Campos Comuns
- **CODPROD**: Código do produto
- **CODCLI**: Código do cliente
- **CODFILIAL**: Código da filial
- **POSICAO**: Status do pedido
- **DATA**: Data da operação
- **QT**: Quantidade
- **PVENDA**: Preço de venda

## 🔧 Configurações Importantes

### Conexão Oracle
```python
DB_HOST = '10.0.0.10'
DB_PORT = 1521
DB_SERVICE = 'WINT'
DB_USER = 'dicon'
DB_PASSWORD = 'wdicon01'
```

### Oracle Client
```python
oracledb.init_oracle_client(lib_dir=r"C:\oracle\instantclient_23_9")
```

## 📝 Convenções de Documentação

### Estrutura de Cada Query
1. **Descrição**: O que a query faz
2. **Contexto**: Quando usar
3. **SQL**: Código completo
4. **Campos**: Explicação de cada campo
5. **Filtros**: Condições aplicadas
6. **Exemplo de Uso**: Como implementar

### Símbolos Utilizados
- 🎯 **Campanhas**: Queries relacionadas a campanhas promocionais
- 📊 **Relatórios**: Queries para geração de relatórios
- 🔄 **Atualizações**: Queries para sincronização de dados
- 📈 **Análises**: Queries para análise de dados
- 🗄️ **Estrutura**: Queries sobre estrutura do banco

## 🤝 Contribuição
Para manter esta documentação atualizada:
1. **Adicione novas queries** quando criar funcionalidades
2. **Atualize queries existentes** quando modificar
3. **Documente mudanças** na estrutura do banco
4. **Teste as queries** antes de documentar

---

**Última Atualização**: Janeiro 2025  
**Versão**: 1.0  
**Responsável**: Sistema de Automações DICON
