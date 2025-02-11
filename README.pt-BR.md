<div align="center">
   <img height="30" width="40" src="https://github.com/hipolitorodrigues/assets-for-github/blob/985021e61af3982fd9f28be446b106b958f24696/images/01/img-readme-ico.svg">
   <a href="./README.md">
      <img height="30" width="120" src="https://github.com/hipolitorodrigues/assets-for-github/blob/985021e61af3982fd9f28be446b106b958f24696/images/01/img-readme-en.svg">
   </a>
   <a href="./README.pt-BR.md">
      <img height="30" width="60" src="https://github.com/hipolitorodrigues/assets-for-github/blob/985021e61af3982fd9f28be446b106b958f24696/images/01/img-readme-pt-br.svg">
   </a>
</div>

# Guia Progressivo de Fundamentos do SQLite com Python

Este guia foi desenvolvido para desenvolvedores que têm Python instalado e desejam usar o **SQLite** através do Python. O objetivo é se familiarizar com como interagir com SQLite usando o módulo `sqlite3` integrado do Python.

---

## 1. Introdução ao SQLite em Python

### 1.1 O que é SQLite?
- **SQLite** é um banco de dados relacional leve e autocontido.
- Não requer um processo de servidor separado.
- Armazena dados em um único arquivo (arquivo `.db`).
- Totalmente compatível com ACID e suporte a transações.

### 1.2 Por que Usar SQLite com Python?
- **Sem dependências externas**: SQLite vem integrado ao Python.
- **Leve e fácil de usar**.
- **Perfeito para aplicações pequenas e médias**.

### 1.3 Instalando SQLite
SQLite vem pré-instalado com Python (versão 3.7+). Para verificar:
```sh
python -c "import sqlite3; print(sqlite3.version)"
```

---

## 2. Operações com BANCO DE DADOS

### 2.1 Criando e Conectando a um Banco de Dados
```python
import sqlite3

# Conectar a (ou criar) um banco de dados
connection = sqlite3.connect('meu_banco.db')

# Fechar a conexão
connection.close()
```

### 2.2 Usando um Banco de Dados em Memória
```python
connection = sqlite3.connect(':memory:')
```

### 2.3 SHOW DATABASES
SQLite não possui um comando SQL específico para mostrar bancos de dados, pois é um banco de dados baseado em arquivo. Em vez disso, você pode verificar o caminho do arquivo da sua conexão atual ou listar seus arquivos `.db` manualmente.

### 2.4 DROP DATABASE
SQLite não suporta o comando `DROP DATABASE`. Para "deletar" um banco de dados, simplesmente exclua o arquivo `.db` do seu sistema de arquivos:
```python
import os
os.remove('meu_banco.db')
```

---

## 3. Operações com TABELAS

### 3.1 Criando uma Tabela
```python
import sqlite3

connection = sqlite3.connect('meu_banco.db')
cursor = connection.cursor()

cursor.execute('''
    CREATE TABLE IF NOT EXISTS Clientes (
        ID INTEGER PRIMARY KEY AUTOINCREMENT,
        Nome TEXT NOT NULL,
        Email TEXT UNIQUE NOT NULL,
        DataRegistro TEXT DEFAULT CURRENT_TIMESTAMP
    )
''')

connection.commit()
connection.close()
```

### 3.2 SHOW TABLES
SQLite não suporta `SHOW TABLES` diretamente. No entanto, você pode consultar a tabela `sqlite_master` para obter uma lista de tabelas:
```python
cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
tables = cursor.fetchall()
for table in tables:
    print(table[0])
```

### 3.3 DROP TABLE
```python
cursor.execute("DROP TABLE IF EXISTS Clientes")
```

---

## 4. Operações com COLUNAS

### 4.1 ADD COLUMN
```python
cursor.execute("ALTER TABLE Clientes ADD COLUMN Idade INTEGER;")
```

### 4.2 SHOW COLUMNS
SQLite não tem um comando específico para mostrar colunas. Você pode consultar a função `PRAGMA table_info()` para recuperar detalhes das colunas:
```python
cursor.execute("PRAGMA table_info(Clientes);")
columns = cursor.fetchall()
for column in columns:
    print(column)
```

### 4.3 DROP COLUMN
SQLite não suporta diretamente a remoção de colunas. Para remover uma coluna, você precisa:
1. Criar uma nova tabela sem a coluna indesejada.
2. Copiar os dados da tabela antiga.
3. Excluir a tabela antiga.

Exemplo:
```python
# Passo 1: Criar uma nova tabela sem a coluna 'Idade'
cursor.execute('''
    CREATE TABLE IF NOT EXISTS NovosClientes (
        ID INTEGER PRIMARY KEY AUTOINCREMENT,
        Nome TEXT NOT NULL,
        Email TEXT UNIQUE NOT NULL,
        DataRegistro TEXT DEFAULT CURRENT_TIMESTAMP
    )
''')

# Passo 2: Copiar dados da tabela antiga
cursor.execute('''
    INSERT INTO NovosClientes (ID, Nome, Email, DataRegistro)
    SELECT ID, Nome, Email, DataRegistro FROM Clientes
''')

# Passo 3: Excluir a tabela antiga
cursor.execute("DROP TABLE IF EXISTS Clientes")

# Passo 4: Renomear a nova tabela
cursor.execute("ALTER TABLE NovosClientes RENAME TO Clientes")
```

---

## 5. Operações com DADOS

### 5.1 Inserindo Dados (Usando Prepared Statements)
```python
connection = sqlite3.connect('meu_banco.db')
cursor = connection.cursor()

cursor.execute("INSERT INTO Clientes (Nome, Email) VALUES (?, ?)",
               ("Alice", "alice@exemplo.com"))

connection.commit()
connection.close()
```

### 5.2 Consultando Dados
```python
connection = sqlite3.connect('meu_banco.db')
cursor = connection.cursor()

cursor.execute("SELECT * FROM Clientes")
clientes = cursor.fetchall()
for cliente in clientes:
    print(cliente)

connection.close()
```

### 5.3 Atualizando Dados
```python
connection = sqlite3.connect('meu_banco.db')
cursor = connection.cursor()

cursor.execute("UPDATE Clientes SET Email = ? WHERE ID = ?", 
               ("alice.novo@exemplo.com", 1))

connection.commit()
connection.close()
```

### 5.4 Deletando Dados
```python
connection = sqlite3.connect('meu_banco.db')
cursor = connection.cursor()

cursor.execute("DELETE FROM Clientes WHERE ID = ?", (1,))

connection.commit()
connection.close()
```

---

## 6. Usando Transações

### 6.1 Habilitando Chaves Estrangeiras
```python
connection = sqlite3.connect('meu_banco.db')
cursor = connection.cursor()
cursor.execute("PRAGMA foreign_keys = ON;")
```

### 6.2 Usando Transações para Múltiplas Consultas
```python
try:
    connection = sqlite3.connect('meu_banco.db')
    cursor = connection.cursor()
    
    cursor.execute("BEGIN TRANSACTION;")
    cursor.execute("INSERT INTO Clientes (Nome, Email) VALUES (?, ?)",
                   ("João Silva", "joao@exemplo.com"))
    cursor.execute("INSERT INTO Clientes (Nome, Email) VALUES (?, ?)",
                   ("Maria Silva", "maria@exemplo.com"))
    
    connection.commit()
except sqlite3.Error as e:
    print("Ocorreu um erro:", e)
    connection.rollback()
finally:
    connection.close()
```

---

## 7. Trabalhando com JSON no SQLite
SQLite fornece suporte nativo a JSON. Você pode usar a função `json_extract()` para trabalhar com dados JSON:
```python
cursor.execute("SELECT json_extract('{\"nome\": \"Alice\"}', '$.nome');")
```

---

## 8. Exportando e Importando Dados

### 8.1 Exportando para CSV
```python
import csv

connection = sqlite3.connect('meu_banco.db')
cursor = connection.cursor()

cursor.execute("SELECT * FROM Clientes")
rows = cursor.fetchall()

with open('clientes.csv', 'w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow([desc[0] for desc in cursor.description])  # Nomes das colunas
    writer.writerows(rows)

connection.close()
```

### 8.2 Importando do CSV
```python
with open('clientes.csv', 'r') as file:
    reader = csv.reader(file)
    next(reader)  # Pular linha de cabeçalho
    
    connection = sqlite3.connect('meu_banco.db')
    cursor = connection.cursor()
    
    for row in reader:
        cursor.execute("INSERT INTO Clientes (ID, Nome, Email, DataRegistro) VALUES (?, ?, ?, ?)", row)
    
    connection.commit()
    connection.close()
```

---

## 9. Melhores Práticas para Usar SQLite com Python
- **Use gerenciadores de contexto (`with sqlite3.connect() as conn`)** para manipular conexões com segurança.
- **Habilite restrições de chave estrangeira (`PRAGMA foreign_keys = ON`).**
- **Use marcadores `?` para prevenir injeção de SQL.**
- **Use transações para operações em lote.**
- **Indexe colunas frequentemente consultadas para melhor desempenho.**

---

## 10. Conclusão
Este guia introduz os fundamentos do uso do **SQLite com Python** seguindo as melhores práticas. Continue explorando tópicos avançados como **triggers, busca em texto completo e extensões SQLite** para aprofundar seu conhecimento! 🚀
