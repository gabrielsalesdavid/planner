# Fundamentos - RocketSeat NLW Journey

## Visão Geral
Este documento descreve os fundamentos técnicos utilizados no projeto **RocketSeat NLW Journey**, uma aplicação web de planejamento de viagens desenvolvida com Spring Boot 3.3.1 e Java 17.

---

## 1. Arquitetura e Padrões

### 1.1 Padrão MVC (Model-View-Controller)
A aplicação segue o padrão arquitetural MVC, dividindo-se em:

- **Model**: Entidades JPA representando as tabelas do banco de dados (Trip, Activity, Participant, Link)
- **View**: Respostas em formato JSON via REST API
- **Controller**: Classes controladoras que gerenciam as requisições HTTP

### 1.2 Padrão de Camadas
A aplicação é organizada em camadas distintas:

```
┌─────────────────────────────────────┐
│     Camada de Apresentação          │
│     (Controllers)                   │
├─────────────────────────────────────┤
│     Camada de Negócio               │
│     (Services)                      │
├─────────────────────────────────────┤
│     Camada de Persistência          │
│     (Repositories + Entities)       │
├─────────────────────────────────────┤
│     Camada de Dados                 │
│     (Banco de Dados - H2/PostgreSQL)│
└─────────────────────────────────────┘
```

---

## 2. Tecnologias Fundamentais

### 2.1 Java 17
- **Versão LTS (Long Term Support)**
- Suporta records, sealed classes e pattern matching
- Melhorias de performance e segurança
- Compatibilidade total com Spring Boot 3.x

### 2.2 Spring Boot 3.3.1
Framework que simplifica a criação de aplicações Java autossuficientes:

- **spring-boot-starter-web**: Criação de APIs REST
- **spring-boot-starter-data-jpa**: ORM (Object-Relational Mapping) com JPA/Hibernate
- **spring-boot-devtools**: Recarregamento automático durante desenvolvimento

### 2.3 Spring Data JPA
Abstração para operações de banco de dados:

- Reduz código boilerplate
- Oferece repositories genéricos (CRUD)
- Suporta queries derivadas do método
- Integração automática com Hibernate

### 2.4 Flyway
Versionamento e migração de banco de dados:

- Scripts SQL controlados por versão (V1__, V2__, etc.)
- Aplicação automática de migrações na inicialização
- Rastreamento de histórico de mudanças no schema

---

## 3. Estrutura de Dados Fundamental

### 3.1 Entidades Principais

#### Trip (Viagem)
```
╔═══════════════════════════════════╗
║ Trip (Tabela: trips)              ║
╠═══════════════════════════════════╣
║ - id: UUID (Primary Key)          ║
║ - destination: String              ║
║ - startsAt: LocalDateTime          ║
║ - endsAt: LocalDateTime            ║
║ - description: String              ║
║ - isConfirmed: Boolean             ║
║ - createdAt: LocalDateTime         ║
╚═══════════════════════════════════╝
```

#### Activity (Atividade)
```
╔═══════════════════════════════════╗
║ Activity (Tabela: activities)     ║
╠═══════════════════════════════════╣
║ - id: UUID (Primary Key)          ║
║ - tripId: UUID (Foreign Key)      ║
║ - title: String                    ║
║ - description: String              ║
║ - occursAt: LocalDateTime          ║
║ - createdAt: LocalDateTime         ║
╚═══════════════════════════════════╝
```

#### Participant (Participante)
```
╔═══════════════════════════════════╗
║ Participant (Tabela: participants)║
╠═══════════════════════════════════╣
║ - id: UUID (Primary Key)          ║
║ - tripId: UUID (Foreign Key)      ║
║ - name: String                     ║
║ - email: String                    ║
║ - isConfirmed: Boolean             ║
║ - createdAt: LocalDateTime         ║
╚═══════════════════════════════════╝
```

#### Link (Link/Recurso)
```
╔═══════════════════════════════════╗
║ Link (Tabela: links)              ║
╠═══════════════════════════════════╣
║ - id: UUID (Primary Key)          ║
║ - tripId: UUID (Foreign Key)      ║
║ - title: String                    ║
║ - url: String                      ║
║ - createdAt: LocalDateTime         ║
╚═══════════════════════════════════╝
```

### 3.2 Relacionamentos

```
┌──────────────┐           ┌──────────────┐
│    Trips     │           │  Activities  │
├──────────────┤      1:N  ├──────────────┤
│ id (PK)      │◀──────────│ tripId (FK)  │
│ destination  │           │ title        │
└──────────────┘           └──────────────┘

┌──────────────┐           ┌──────────────┐
│    Trips     │           │ Participants │
├──────────────┤      1:N  ├──────────────┤
│ id (PK)      │◀──────────│ tripId (FK)  │
│ destination  │           │ email        │
└──────────────┘           └──────────────┘

┌──────────────┐           ┌──────────────┐
│    Trips     │           │    Links     │
├──────────────┤      1:N  ├──────────────┤
│ id (PK)      │◀──────────│ tripId (FK)  │
│ destination  │           │ url          │
└──────────────┘           └──────────────┘
```

---

## 4. Padrões de Persistência

### 4.1 Anotações JPA Fundamentais

| Anotação | Propósito |
|----------|-----------|
| `@Entity` | Marca a classe como entidade JPA |
| `@Table` | Define o nome da tabela no banco |
| `@Id` | Define chave primária |
| `@GeneratedValue` | Gera valor automático da chave primária |
| `@Column` | Configura características da coluna |
| `@OneToMany` | Define relação 1:N |
| `@ManyToOne` | Define relação N:1 |

### 4.2 Repositories
Interface que estende `JpaRepository<Entity, ID>`:

```java
public interface TripRepository extends JpaRepository<Trip, UUID> {
    // Oferece automaticamente métodos CRUD
    // save(), findById(), findAll(), delete(), etc.
}
```

---

## 5. Comunicação REST

### 5.1 Protocolo HTTP
Requisições/Respostas usando métodos HTTP padrão:

| Método | Operação | Exemplo |
|--------|----------|---------|
| GET | Recuperar | `GET /trips/{id}` |
| POST | Criar | `POST /trips` |
| PUT | Atualizar | `PUT /trips/{id}` |
| DELETE | Deletar | `DELETE /trips/{id}` |

### 5.2 Padrão de Resposta
Respostas em JSON com estrutura padronizada:

```json
{
  "id": "uuid-da-viagem",
  "destination": "Rio de Janeiro",
  "startsAt": "2024-01-20T10:00:00",
  "endsAt": "2024-01-25T18:00:00"
}
```

### 5.3 Payload de Requisição
Dados enviados pelo cliente para criar/atualizar recursos:

```json
{
  "destination": "Salvador",
  "startsAt": "2024-02-01T08:00:00",
  "endsAt": "2024-02-07T20:00:00"
}
```

---

## 6. Lombok

Biblioteca que reduz código boilerplate:

| Anotação | Efeito |
|----------|--------|
| `@Getter` | Gera métodos getter |
| `@Setter` | Gera métodos setter |
| `@NoArgsConstructor` | Construtor sem argumentos |
| `@AllArgsConstructor` | Construtor com todos os argumentos |
| `@Data` | Combina @Getter, @Setter e @ToString |

---

## 7. Configuração da Aplicação

### 7.1 application.properties
Arquivo de configuração centralizado:

```properties
# Banco de dados
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

# Hibernate/JPA
spring.jpa.hibernate.ddl-auto=validate

# Flyway
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
```

---

## 8. Ciclo de Vida da Aplicação

```
1. Inicialização Spring Boot
   ↓
2. Carregamento de properties
   ↓
3. Configuração de Data Source
   ↓
4. Execução de migrações Flyway
   ↓
5. Inicialização de Repositories
   ↓
6. Carregamento de Controllers
   ↓
7. Aplicação pronta para requisições
   ↓
8. Listener HTTP na porta configurada (padrão: 8080)
```

---

## 9. Fluxo de Uma Requisição

```
Cliente (Frontend/Postman)
    ↓
HTTP Request (GET /trips/123)
    ↓
Controller (TripController)
    ↓
Service (TripService)
    ↓
Repository (TripRepository)
    ↓
JPA/Hibernate
    ↓
Banco de Dados
    ↓
EntityManager
    ↓
Repository
    ↓
Service
    ↓
Response DTO
    ↓
JSON Response
    ↓
Cliente
```

---

## 10. UUID vs ID Numérico

### 10.1 UUID (Universally Unique Identifier)

**Vantagens:**
- Globalm e único
- Não sequencial (melhor para segurança)
- Gerado no cliente ou servidor
- Reduz conflitos em ambientes distribuídos

**Implementação:**
```java
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private UUID id;
```

---

## 11. Versionamento de Banco de Dados com Flyway

### 11.1 Estrutura de Migrações

```
src/main/resources/db/migration/
├── V1__create-table-trips.sql
├── V2__create-table-participants.sql
├── V3__create-table-activities.sql
└── V4__create-table-links.sql
```

### 11.2 Convenção de Nomenclatura

- **V** = Version
- **1** = Número sequencial
- **__ (double underscore)** = Separador obrigatório
- **create-table-trips.sql** = Descrição legível

---

## 12. Princípios SOLID Aplicados

| Princípio | Aplicação |
|-----------|-----------|
| **S** (Single Responsibility) | Cada classe tem uma responsabilidade única (Entities, Services, Controllers) |
| **O** (Open/Closed) | Controllers abertos para extensão via herança, fechados para modificação |
| **L** (Liskov Substitution) | Repositories implementam contrato de JpaRepository |
| **I** (Interface Segregation) | Interfaces específicas por domínio (TripRepository, ActivityRepository) |
| **D** (Dependency Inversion) | Injeção de dependência via @Autowired e construtores |

---

## 13. LocalDateTime e Manipulação de Datas

### 13.1 Classe LocalDateTime
- Imutável e thread-safe
- Não inclui informação de timezone
- Ideal para aplicações simples e servidores em um mesmo timezone

### 13.2 Formatação
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss");
LocalDateTime dateTime = LocalDateTime.parse("2024-01-20T10:00:00", formatter);
```

---

## Conclusão

Os fundamentos descritos estabelecem uma base sólida para:
- Desenvolvimento escalável de APIs REST
- Persistência de dados confiável e versionada
- Separação clara de responsabilidades
- Manutenibilidade e extensibilidade do código

Esses conceitos formam a espinha dorsal da aplicação RocketSeat NLW Journey.
