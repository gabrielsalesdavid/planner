# RocketSeat NLW Journey - Planner de Viagens

## 📌 Visão Geral

**RocketSeat NLW Journey** é uma aplicação web moderna de planejamento de viagens desenvolvida durante o **Bootcamp Next Level Week (NLW)** da RocketSeat. A aplicação permite que usuários criem viagens, adicionem participantes, atividades e links úteis de forma simples e intuitiva.

O projeto demonstra boas práticas de desenvolvimento com **Java 17** e **Spring Boot 3.3.1**, implementando padrões arquitetônicos robustos e uma API REST bem estruturada.

---

## 🎯 Objetivo

Fornecer uma plataforma para:
- Criar e gerenciar viagens com datas e destinos
- Convidar participantes para viagens
- Registrar atividades durante a viagem
- Compartilhar links úteis (hotéis, restaurantes, ingressos, etc.)
- Confirmar participação na viagem

---

## 🛠️ Stack Tecnológico

### Backend
- **Java 17** - Linguagem de programação (LTS)
- **Spring Boot 3.3.1** - Framework web e microserviços
- **Spring Data JPA** - ORM (Object-Relational Mapping)
- **Hibernate** - Mapeamento de dados
- **Flyway** - Versionamento de banco de dados
- **Lombok** - Redução de boilerplate
- **Maven 3.x** - Gerenciamento de dependências

### Banco de Dados
- **H2 Database** - Desenvolvimento e testes
- **PostgreSQL** - Produção (configurável)

### Ferramentas
- **Spring Boot DevTools** - Recarregamento automático
- **JUnit 5** - Framework de testes

---

## 📂 Estrutura do Projeto

```
RocketSeatNLWJourney/
├── src/
│   ├── main/
│   │   ├── java/com/rocketseat/planner/
│   │   │   ├── PlannerApplication.java          # Classe principal
│   │   │   ├── activity/                        # Módulo de Atividades
│   │   │   │   ├── Activity.java               # Entidade
│   │   │   │   ├── ActivityData.java           # DTO de dados
│   │   │   │   ├── ActivityRepository.java     # Repositório
│   │   │   │   ├── ActivityService.java        # Lógica de negócio
│   │   │   │   ├── ActivityResponse.java       # DTO de resposta
│   │   │   │   ├── ActivityRequestPayload.java # DTO de entrada
│   │   │   │
│   │   │   ├── link/                           # Módulo de Links
│   │   │   │   ├── Link.java                  # Entidade
│   │   │   │   ├── LinkData.java              # DTO de dados
│   │   │   │   ├── LinkRepository.java        # Repositório
│   │   │   │   ├── LinkService.java           # Lógica de negócio
│   │   │   │   ├── LinkResponse.java          # DTO de resposta
│   │   │   │   ├── LinkRequestPayload.java    # DTO de entrada
│   │   │   │
│   │   │   ├── participant/                    # Módulo de Participantes
│   │   │   │   ├── Participant.java           # Entidade
│   │   │   │   ├── ParticipantController.java # Controller REST
│   │   │   │   ├── ParticipantData.java       # DTO de dados
│   │   │   │   ├── ParticipantRepository.java # Repositório
│   │   │   │   ├── ParticipantService.java    # Lógica de negócio
│   │   │   │   ├── ParticipantRequestPayload.java # DTO de entrada
│   │   │   │   └── ParticipantCreateResponse.java # DTO de resposta
│   │   │   │
│   │   │   └── trip/                          # Módulo de Viagens
│   │   │       ├── Trip.java                  # Entidade
│   │   │       ├── TripController.java        # Controller REST
│   │   │       ├── TripRepository.java        # Repositório
│   │   │       ├── TripRequestPayload.java    # DTO de entrada
│   │   │       └── TripCreateResponse.java    # DTO de resposta
│   │   │
│   │   └── resources/
│   │       ├── application.properties         # Configurações
│   │       └── db/migration/                  # Scripts SQL Flyway
│   │           ├── V1__create-table-trips.sql
│   │           ├── V2__create-table-participants.sql
│   │           ├── V3__create-table-activities.sql
│   │           └── V4__create-table-links.sql
│   │
│   └── test/
│       └── java/com/rocketseat/planner/
│           └── PlannerApplicationTests.java
│
├── Conceitos/                                 # Documentação de conceitos
│   └── Conceitos_Arquitetura_NLW.md
├── Fundamentos/                              # Documentação de fundamentos
│   └── Fundamentos_RocketSeat_NLW.md
├── Docs/                                     # Documentação adicional
├── pom.xml                                   # Dependências Maven
├── mvnw                                      # Maven wrapper (Linux/Mac)
├── mvnw.cmd                                  # Maven wrapper (Windows)
└── README.md                                 # Este arquivo
```

---

## 🚀 Como Executar

### Pré-requisitos
- **Java 17+** instalado
- **Maven 3.6+** instalado
- **Git** para clonar o repositório

### Passo a Passo

1. **Clonar o repositório**
```bash
git clone https://github.com/gabrielsalesdavid/RocketSeat-NLW-Journey.git
cd RocketSeatNLWJourney
```

2. **Compilar o projeto**
```bash
./mvnw clean compile
```

3. **Executar a aplicação**
```bash
./mvnw spring-boot:run
```

A aplicação estará disponível em: `http://localhost:8080`

### Alternativa com IDE
1. Abra o projeto em sua IDE (IntelliJ IDEA, Eclipse, VS Code)
2. Clique com botão direito no `PlannerApplication.java`
3. Selecione "Run"

---

## 📋 Endpoints da API

### Viagens (Trips)

#### Criar Viagem
```
POST /trips
Content-Type: application/json

{
  "destination": "Paris",
  "startsAt": "2024-02-15T10:00:00",
  "endsAt": "2024-02-22T18:00:00"
}

Resposta: 201 Created
{
  "tripId": "uuid-da-viagem"
}
```

#### Obter Detalhes da Viagem
```
GET /trips/{tripId}

Resposta: 200 OK
{
  "id": "uuid-da-viagem",
  "destination": "Paris",
  "startsAt": "2024-02-15T10:00:00",
  "endsAt": "2024-02-22T18:00:00",
  "isConfirmed": false,
  "createdAt": "2024-01-17T14:30:00"
}
```

#### Confirmar Viagem
```
PATCH /trips/{tripId}/confirm

Resposta: 204 No Content
```

### Participantes (Participants)

#### Convidar Participante
```
POST /trips/{tripId}/invites
Content-Type: application/json

{
  "email": "joao@example.com",
  "name": "João Silva"
}

Resposta: 201 Created
{
  "participantId": "uuid-participante"
}
```

#### Listar Participantes
```
GET /trips/{tripId}/participants

Resposta: 200 OK
{
  "participants": [
    {
      "id": "uuid-participante",
      "name": "João Silva",
      "email": "joao@example.com",
      "isConfirmed": false,
      "createdAt": "2024-01-17T14:30:00"
    }
  ]
}
```

#### Confirmar Presença do Participante
```
PATCH /participants/{participantId}/confirm

Resposta: 204 No Content
```

### Atividades (Activities)

#### Registrar Atividade
```
POST /trips/{tripId}/activities
Content-Type: application/json

{
  "title": "Visita à Torre Eiffel",
  "description": "Passeio turístico",
  "occursAt": "2024-02-17T14:00:00"
}

Resposta: 201 Created
{
  "activityId": "uuid-atividade"
}
```

#### Listar Atividades
```
GET /trips/{tripId}/activities

Resposta: 200 OK
{
  "activities": [
    {
      "id": "uuid-atividade",
      "title": "Visita à Torre Eiffel",
      "description": "Passeio turístico",
      "occursAt": "2024-02-17T14:00:00",
      "createdAt": "2024-01-17T14:30:00"
    }
  ]
}
```

### Links (Recursos)

#### Adicionar Link
```
POST /trips/{tripId}/links
Content-Type: application/json

{
  "title": "Hotel des Invalides",
  "url": "https://example.com/hotel"
}

Resposta: 201 Created
{
  "linkId": "uuid-link"
}
```

#### Listar Links
```
GET /trips/{tripId}/links

Resposta: 200 OK
{
  "links": [
    {
      "id": "uuid-link",
      "title": "Hotel des Invalides",
      "url": "https://example.com/hotel",
      "createdAt": "2024-01-17T14:30:00"
    }
  ]
}
```

---

## 🏗️ Arquitetura

### Padrão em Camadas

```
┌──────────────────────────────┐
│   Presentation Layer         │
│   (Controllers)              │
├──────────────────────────────┤
│   Business Logic Layer       │
│   (Services)                 │
├──────────────────────────────┤
│   Persistence Layer          │
│   (Repositories + JPA)       │
├──────────────────────────────┤
│   Data Access Layer          │
│   (Database)                 │
└──────────────────────────────┘
```

### Padrões de Design Implementados

1. **Repository Pattern** - Abstração do acesso a dados
2. **Service Layer** - Lógica de negócio centralizada
3. **DTO Pattern** - Transferência de dados entre camadas
4. **Controller Pattern** - Manipulação de requisições HTTP
5. **Dependency Injection** - Inversão de controle com Spring

---

## 🗄️ Modelo de Dados

### Entidades Principais

#### Trip (Viagem)
- `id`: UUID (Chave Primária)
- `destination`: String (Destino)
- `startsAt`: LocalDateTime (Data de início)
- `endsAt`: LocalDateTime (Data de término)
- `description`: String (Descrição)
- `isConfirmed`: Boolean (Confirmada?)
- `createdAt`: LocalDateTime (Data de criação)

#### Participant (Participante)
- `id`: UUID (Chave Primária)
- `tripId`: UUID (Chave Estrangeira)
- `name`: String (Nome)
- `email`: String (Email)
- `isConfirmed`: Boolean (Confirmado?)
- `createdAt`: LocalDateTime (Data de criação)

#### Activity (Atividade)
- `id`: UUID (Chave Primária)
- `tripId`: UUID (Chave Estrangeira)
- `title`: String (Título)
- `description`: String (Descrição)
- `occursAt`: LocalDateTime (Data da atividade)
- `createdAt`: LocalDateTime (Data de criação)

#### Link (Recurso)
- `id`: UUID (Chave Primária)
- `tripId`: UUID (Chave Estrangeira)
- `title`: String (Título)
- `url`: String (URL)
- `createdAt`: LocalDateTime (Data de criação)

### Relacionamentos
```
Trip (1) ──── (N) Participant
Trip (1) ──── (N) Activity
Trip (1) ──── (N) Link
```

---

## 📚 Documentação Completa

O projeto inclui duas documentações detalhadas:

### 1. **Fundamentos** (`Fundamentos/Fundamentos_RocketSeat_NLW.md`)
Cobre os conceitos técnicos básicos:
- Arquitetura MVC e em camadas
- Tecnologias utilizadas (Java 17, Spring Boot, JPA)
- Estrutura de dados
- Persistência com JPA
- REST API
- Lombok
- Flyway para migrações
- Princípios SOLID

### 2. **Conceitos** (`Conceitos/Conceitos_Arquitetura_NLW.md`)
Apresenta conceitos avançados:
- Arquitetura em camadas detalhada
- Padrões de design (Repository, DTO, Service, Controller)
- REST API conceitual
- ORM e EntityManager
- Validações
- Injeção de dependência
- Segurança (OWASP)
- Performance
- Testes
- Escalabilidade
- Logging

---

## 🧪 Testes

### Executar Testes
```bash
./mvnw test
```

### Cobertura de Testes
```bash
./mvnw test jacoco:report
```

---

## 🔧 Configurações

### `application.properties`

#### Banco de Dados
```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
```

#### Hibernate/JPA
```properties
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
```

#### Flyway
```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
```

### Trocar para PostgreSQL (Produção)
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/planner
spring.datasource.username=postgres
spring.datasource.password=sua_senha
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQL10Dialect
```

---

## 📝 Migrações de Banco de Dados

O projeto usa **Flyway** para versionamento automático:

```
V1__create-table-trips.sql         - Cria tabela trips
V2__create-table-participants.sql  - Cria tabela participants
V3__create-table-activities.sql    - Cria tabela activities
V4__create-table-links.sql         - Cria tabela links
```

As migrações são executadas automaticamente na inicialização.

---

## 🔐 Segurança

Implementações recomendadas:
- ✅ Validação de entrada em todas as camadas
- ⚠️ Autenticação (JWT/OAuth2) - Futura implementação
- ⚠️ Autorização (RBAC) - Futura implementação
- ✅ HTTPS em produção
- ⚠️ Rate limiting - Futura implementação
- ✅ UUIDs para chaves primárias (não sequencial)

---

## 📊 Diagrama de Classes

```
┌─────────────────────┐
│       Trip          │
├─────────────────────┤
│ - id: UUID          │
│ - destination: Str  │
│ - startsAt: LocalDT │
│ - endsAt: LocalDT   │
└─────────────────────┘
        │ 1
        │
        ├──────(N)──────┐
        │               │
    ┌──────────┐   ┌──────────┐
    │Participant│   │Activity  │
    └──────────┘   └──────────┘
        │               │
        └──────(N)──────┘
        
┌─────────────────────┐
│       Link          │
├─────────────────────┤
│ - id: UUID          │
│ - tripId: UUID (FK) │
│ - title: String     │
│ - url: String       │
└─────────────────────┘
```

---

## 🚀 Próximos Passos

- [ ] Autenticação com JWT
- [ ] Autorização baseada em papéis
- [ ] Validação de email
- [ ] Notificações por email
- [ ] Frontend em React/Vue
- [ ] Deploy em Azure/AWS
- [ ] Docker support
- [ ] CI/CD pipeline
- [ ] Testes de integração
- [ ] Documentação OpenAPI/Swagger

---

## 📞 Suporte e Contribuições

Para reportar bugs ou sugerir melhorias, abra uma issue no repositório.

### Contribuindo
1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

---

## 📄 Licença

Este projeto é de código aberto e está disponível sob a licença MIT.

---

## 👨‍💼 Autor

**Gabriel Sales David**
- GitHub: [@gabrielsalesdavid](https://github.com/gabrielsalesdavid)
- Estudante do Bootcamp NLW - RocketSeat

---

## 🎓 Referências

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [Flyway Documentation](https://flywaydb.org/documentation)
- [REST API Best Practices](https://restfulapi.net/)
- [Java 17 Features](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)

---

**Última atualização:** 17 de janeiro de 2026
