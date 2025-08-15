# Cadastro de Clientes — Arquitetura e Decisões de Design

## Visão Geral
O sistema **Cadastro de Clientes** foi projetado para atender novo requisito, implementando um CRUD completo de **Clientes** e **Logradouros**, com:
- Armazenamento de **logotipo** no banco de dados (`VARBINARY(MAX)`)
- Autenticação e autorização via **JWT**
- Arquitetura escalável e de alta performance
- Separação de responsabilidades usando **DDD**, **Clean Architecture** e **Clean Code**

A solução é composta por:
- **Frontend MVC**: ASP.NET MVC (Razor, Controllers, Views)
- **API REST**: .NET 8, seguindo CQRS híbrido (EF Core para leitura e Stored Procedures + Dapper para escrita)
- **Banco de Dados**: SQL Server 2016+  
- **Cache opcional**: Redis para leitura de alta demanda

---

## C4 Model
Este projeto foi documentado usando o **C4 Model**, para garantir clareza em diferentes níveis de detalhe.

- **Nível 1 — Context**  
  Mostra como o sistema se conecta com usuários e sistemas externos.
- **Nível 2 — Containers**  
  Identifica os "containers" (aplicações, serviços, bancos de dados) e suas interações.
- **Nível 3 — Components**  
  Detalha a arquitetura interna de um container.
- **Nível 4 — Code** *(opcional)*  
  Mostra classes, métodos e relacionamentos em UML para orientar desenvolvedores.

---

## Diagramas

### **Nível 1 — Context**
![Context Diagram](docs/cadastro-clientes-Context.drawio.svg)

### **Nível 2 — Containers**
![Containers Diagram](docs/cadastro-clientes-Containers.drawio.svg)

### **Nível 3 — Components (API)**
![Components API Diagram](docs/cadastro-clientes-Componentes.drawio.svg)

### **Nível 3 — Frontend MVC**
![Frontend MVC Diagram](docs/cadastro-clientes-Frontend-MVC.drawio.svg)

### **Sequência — Criar Cliente**
![Sequence Diagram](docs/cadastro-clientes-Sequencia.drawio.svg)

### **Nível 4 — Code**

#### API
![Code - API](docs/cadastro-clientes-Code-API.drawio.svg)

---

## Decisões de Design

| Requisito | Solução |
|-----------|---------|
| **CRUD de Clientes e Logradouros** | API REST (.NET 8) com endpoints para criar, atualizar, listar e remover. Modelo de domínio com agregado `Cliente` e entidade `Logradouro`. |
| **E-mail único** | Validação no domínio e constraint única no banco (`IsDeleted=0`). |
| **Vários logradouros** | Relacionamento 1:N (`Cliente` → `Logradouro`). |
| **Logotipo no banco** | Tabela `Logotipo` (`VARBINARY(MAX)`, metadados e hash SHA-256). |
| **API aberta com segurança** | JWT com OIDC, roles e policies (`Admin`, `Reader`). |
| **Alta performance** | CQRS híbrido: EF Core para leitura (AsNoTracking, projeção DTO) e Dapper + Stored Procedures para escrita. Índices otimizados e paginação em todas as consultas. |
| **Concorrência** | `rowversion` no SQL Server, validado no update (`If-Match` ou campo DTO). |
| **Soft delete** | Campo `IsDeleted` no `Cliente`. |

---

## Arquitetura (Clean Architecture + DDD)

- **Presentation Layer**:  
  - *MVC*: Controllers, Views.  
  - *API*: Controllers que recebem HTTP e delegam para casos de uso (Handlers).
- **Application Layer**:  
  - Handlers CQRS, DTOs, validações, orquestração de persistência.
- **Domain Layer**:  
  - Entidades, Value Objects, invariantes e regras de negócio.
- **Infrastructure Layer**:  
  - EF Core (read), Dapper + SPs (write), Auth, Cache, Logging.

---

## Como essa arquitetura atende aos requisitos

- **Escalabilidade**: API stateless + cache distribuído (Redis) permite balanceamento horizontal.
- **Segurança**: JWT, roles e policies controlam acesso.
- **Performance**: SPs otimizadas para escrita, EF Core projetado para consultas rápidas.
- **Manutenibilidade**: DDD e Clean Architecture mantêm o domínio desacoplado de frameworks.
- **Confiabilidade**: Controle de concorrência com `rowversion` e transações no banco.

---

## Como Executar (POC)

1. **Pré-requisitos**  
   - .NET 8 SDK  
   - SQL Server 2016+  
   - Docker (opcional para Redis/local DB)

2. **Configurar Banco de Dados**  
   - Executar migrations (`dotnet ef database update`)  
   - Executar scripts SQL (`/deploy/scripts.sql`)

3. **Executar API**  
   ```bash
   cd src/Presentation.Api
   dotnet run
