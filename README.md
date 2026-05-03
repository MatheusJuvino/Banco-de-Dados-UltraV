[README.md](https://github.com/user-attachments/files/27322608/README.md)
# Banco-de-Dados-UltraV
Banco de dados em MySQL de repositório para projeto de um site
# 🗄️ UltraV — Banco de Dados

Schema relacional MySQL para um sistema de ótica, cobrindo clientes, receitas médicas, estoque de armações e lentes, carrinho de compras e vendas.

---

## 📋 Requisitos

- MySQL 5.7+ ou MariaDB 10.3+
- Permissão para `CREATE DATABASE`

---

## 🚀 Como executar

```bash
mysql -u seu_usuario -p < ultrav_schema.sql
```

Ou cole o conteúdo diretamente no seu cliente SQL (MySQL Workbench, DBeaver, etc.).

---

## 🗂️ Estrutura do banco

O banco `UltraV` é composto por **9 tabelas** organizadas em três grupos:

### 👤 Cadastro de clientes

| Tabela     | Descrição                                      |
|------------|------------------------------------------------|
| `Cliente`  | Dados pessoais: nome, CPF, telefone e e-mail   |
| `Endereco` | Endereços vinculados a um cliente (1:N)        |
| `Receita`  | Receitas oftalmológicas por cliente (OD e OE)  |

### 📦 Catálogo de produtos

| Tabela     | Descrição                                             |
|------------|-------------------------------------------------------|
| `Armacao`  | Armações disponíveis: modelo, marca, cor, preço e estoque |
| `Lente`    | Lentes: tipo (simples, multifocal), material e preço  |

### 🛒 Comercial

| Tabela          | Descrição                                                  |
|-----------------|------------------------------------------------------------|
| `Carrinho`      | Carrinho de compras de um cliente (ativo ou finalizado)    |
| `ItemCarrinho`  | Itens (armação + lente) dentro de um carrinho              |
| `Venda`         | Registro de venda com receita, endereço e tipo de entrega  |
| `ItemVenda`     | Itens vendidos com preço unitário no momento da venda      |

---

## 🔗 Diagrama de relacionamentos

```
Cliente ──< Endereco
        ──< Receita
        ──< Carrinho ──< ItemCarrinho >── Armacao
        |                            >── Lente
        └──< Venda >── Receita
                  >── Endereco
                  └──< ItemVenda >── Armacao
                                 >── Lente
```

---

## 🔑 Chaves estrangeiras

| Constraint          | Tabela origem  | Referencia              |
|---------------------|----------------|-------------------------|
| `fk_end_cliente`    | Endereco       | Cliente(id_cliente)     |
| `fk_rec_cliente`    | Receita        | Cliente(id_cliente)     |
| `fk_car_cliente`    | Carrinho       | Cliente(id_cliente)     |
| `fk_ic_carrinho`    | ItemCarrinho   | Carrinho(id_carrinho)   |
| `fk_ic_armacao`     | ItemCarrinho   | Armacao(id_armacao)     |
| `fk_ic_lente`       | ItemCarrinho   | Lente(id_lente)         |
| `fk_ven_cliente`    | Venda          | Cliente(id_cliente)     |
| `fk_ven_receita`    | Venda          | Receita(id_receita)     |
| `fk_ven_endereco`   | Venda          | Endereco(id_endereco)   |
| `fk_iv_venda`       | ItemVenda      | Venda(id_venda)         |
| `fk_iv_armacao`     | ItemVenda      | Armacao(id_armacao)     |
| `fk_iv_lente`       | ItemVenda      | Lente(id_lente)         |

---

## 📐 Detalhes das tabelas

### Cliente
```sql
id_cliente  INT PRIMARY KEY AUTO_INCREMENT
nome        VARCHAR(100) NOT NULL
cpf         VARCHAR(14)  UNIQUE
telefone    VARCHAR(20)
email       VARCHAR(100)
```

### Endereco
```sql
id_endereco INT PRIMARY KEY AUTO_INCREMENT
id_cliente  INT NOT NULL  -- FK → Cliente
rua         VARCHAR(150)
numero      VARCHAR(10)
bairro      VARCHAR(100)
cidade      VARCHAR(100)
estado      VARCHAR(50)
cep         VARCHAR(10)
complemento VARCHAR(100)
```

### Receita
```sql
id_receita         INT PRIMARY KEY AUTO_INCREMENT
id_cliente         INT NOT NULL  -- FK → Cliente
data_receita       DATE
grau_esferico_od   DECIMAL(4,2)
grau_cilindrico_od DECIMAL(4,2)
eixo_od            INT
grau_esferico_oe   DECIMAL(4,2)
grau_cilindrico_oe DECIMAL(4,2)
eixo_oe            INT
```

### Armacao
```sql
id_armacao INT PRIMARY KEY AUTO_INCREMENT
modelo     VARCHAR(100)
marca      VARCHAR(100)
cor        VARCHAR(50)
preco      DECIMAL(10,2)
estoque    INT
```

### Lente
```sql
id_lente INT PRIMARY KEY AUTO_INCREMENT
tipo     VARCHAR(100)  -- ex: simples, multifocal
material VARCHAR(100)
preco    DECIMAL(10,2)
```

### Carrinho
```sql
id_carrinho  INT PRIMARY KEY AUTO_INCREMENT
id_cliente   INT NOT NULL  -- FK → Cliente
data_criacao DATETIME
status       VARCHAR(20)   -- ativo | finalizado
```

### ItemCarrinho
```sql
id_item     INT PRIMARY KEY AUTO_INCREMENT
id_carrinho INT NOT NULL  -- FK → Carrinho
id_armacao  INT           -- FK → Armacao
id_lente    INT           -- FK → Lente
quantidade  INT
```

### Venda
```sql
id_venda     INT PRIMARY KEY AUTO_INCREMENT
id_cliente   INT NOT NULL  -- FK → Cliente
id_receita   INT           -- FK → Receita
id_endereco  INT           -- FK → Endereco
data_venda   DATETIME
valor_total  DECIMAL(10,2)
tipo_entrega VARCHAR(20)   -- entrega | retirada
```

### ItemVenda
```sql
id_item        INT PRIMARY KEY AUTO_INCREMENT
id_venda       INT NOT NULL  -- FK → Venda
id_armacao     INT           -- FK → Armacao
id_lente       INT           -- FK → Lente
quantidade     INT
preco_unitario DECIMAL(10,2)
```

---

## 💡 Observações de design

- **`ItemVenda` armazena `preco_unitario`** — isso é intencional. O preço de uma lente ou armação pode mudar com o tempo; registrar o valor no momento da venda garante integridade histórica.
- **`id_armacao` e `id_lente` são opcionais** em `ItemCarrinho` e `ItemVenda`, permitindo vender somente armação, somente lente, ou os dois juntos no mesmo item.
- **`Carrinho` tem status `ativo/finalizado`** — ao finalizar uma venda, o carrinho deve ser atualizado para `finalizado` pela aplicação.
- O banco usa **`utf8mb4`** como charset, suportando caracteres especiais e emojis corretamente.

---

## 📁 Arquivos

```
.
└── ultrav_schema.sql   # Script completo de criação do banco
└── README.md           # Este arquivo
```
