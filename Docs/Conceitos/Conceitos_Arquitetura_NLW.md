# Conceitos - Arquitetura e Padrões do RocketSeat NLW Journey

## Visão Geral
Este documento apresenta os conceitos arquitetônicos, padrões de design e abordagens metodológicas adotadas no projeto **RocketSeat NLW Journey**.

---

## 1. Conceitos de Arquitetura

### 1.1 Arquitetura em Camadas (Layered Architecture)

A aplicação é organizada verticalmente em camadas que se comunicam de forma unidirecional:

```
┌─────────────────────────────────────────────────────┐
│         CAMADA DE APRESENTAÇÃO (Presentation Layer) │
│  ▼                                                   │
│  HTTP Requests ─→ Controllers ─→ HTTP Responses    │
│                                                     │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│        CAMADA DE NEGÓCIO (Business Logic Layer)     │
│                                                     │
│  Services ─→ Regras de Negócio ─→ Validações      │
│  Orquestração ─→ Transformações de Dados           │
│                                                     │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│      CAMADA DE PERSISTÊNCIA (Persistence Layer)     │
│                                                     │
│  Repositories ─→ JPA/Hibernate ─→ ORM               │
│  Mapeamento Objeto-Relacional                       │
│                                                     │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│         CAMADA DE DADOS (Data Access Layer)         │
│                                                     │
│  SQL Queries ─→ Banco de Dados ─→ Resultados       │
│  ACID Compliance                                    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Benefícios:**
- Separação de responsabilidades clara
- Testes unitários facilitados
- Manutenção simplificada
- Escalabilidade horizontal

### 1.2 Inversão de Controle (IoC)

O Spring Boot gerencia:

```
Tradicional (Sem IoC):
MyService service = new MyService();
MyController controller = new MyController(service);

Com IoC (Spring):
@Autowired
private MyService service;  // Spring injeta automaticamente
```

---

## 2. Padrões de Design Utilizados

### 2.1 Padrão Repository

**Objetivo:** Abstrair a lógica de acesso aos dados

```java
// Abstração
public interface TripRepository extends JpaRepository<Trip, UUID> {
    List<Trip> findByDestination(String destination);
}

// Implementação gerenciada automaticamente pelo Spring
@Service
public class TripService {
    @Autowired
    private TripRepository tripRepository;
    
    public Trip findTrip(UUID id) {
        return tripRepository.findById(id).orElseThrow();
    }
}
```

**Vantagens:**
- Desacopla lógica de negócio da persistência
- Facilita testes com mocks
- Permite trocar implementação do banco sem afetar services

### 2.2 Padrão DTO (Data Transfer Object)

**Objetivo:** Transferir dados entre camadas sem expor entidades

```
Fluxo de Dados:
┌────────────┐  REQUEST   ┌─────────────────┐
│  Cliente   │───────────→│ PayloadRequest  │ (DTO)
└────────────┘            └─────────────────┘
                                ↓ mapping
                          ┌──────────────┐
                          │  Entity      │
                          │  (Trip)      │
                          └──────────────┘
                                ↓
                          ┌─────────────────┐
                          │  Response DTO   │
                          └─────────────────┘
                                ↓ JSON
┌────────────┐  RESPONSE  ┌─────────────────┐
│  Cliente   │←───────────│  JSON Response  │
└────────────┘            └─────────────────┘
```

**Exemplo no projeto:**
- `TripRequestPayload`: DTO para entrada de dados
- `TripCreateResponse`: DTO para resposta

**Benefícios:**
- Controla quais dados são expostos
- Valida dados de entrada
- Desacopla estrutura do banco da API

### 2.3 Padrão Service (Business Logic Layer)

**Objetivo:** Encapsular a lógica de negócio

```java
@Service
public class TripService {
    
    @Autowired
    private TripRepository tripRepository;
    
    // Lógica de negócio
    public Trip createTrip(TripRequestPayload payload) {
        // Validações
        if (payload.destination.isEmpty()) {
            throw new InvalidDestinationException();
        }
        
        // Transformações
        Trip trip = new Trip(
            UUID.randomUUID(),
            payload.destination,
            payload.startsAt,
            payload.endsAt
        );
        
        // Persistência
        return tripRepository.save(trip);
    }
}
```

**Responsabilidades:**
- Validação de regras de negócio
- Orquestração de múltiplos repositories
- Transformação de dados
- Tratamento de exceções

### 2.4 Padrão Controller (API Endpoints)

**Objetivo:** Gerenciar requisições HTTP

```java
@RestController
@RequestMapping("/trips")
public class TripController {
    
    @Autowired
    private TripService tripService;
    
    @PostMapping
    public ResponseEntity<TripCreateResponse> createTrip(
        @RequestBody TripRequestPayload payload
    ) {
        Trip trip = tripService.createTrip(payload);
        return ResponseEntity.ok(new TripCreateResponse(trip));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<TripResponse> getTrip(@PathVariable UUID id) {
        Trip trip = tripService.findTrip(id);
        return ResponseEntity.ok(new TripResponse(trip));
    }
}
```

**Responsabilidades:**
- Mapear requisições HTTP para operações
- Validar entrada de dados
- Serializar respostas em JSON
- Códigos HTTP apropriados

---

## 3. Conceitos de REST API

### 3.1 REST (Representational State Transfer)

Princípios arquitetônicos para sistemas distribuídos:

| Princípio | Descrição | Exemplo |
|-----------|-----------|---------|
| **Client-Server** | Separação clara entre cliente e servidor | Frontend → Backend |
| **Statelessness** | Servidor não mantém estado da sessão | Cada requisição é independente |
| **Cacheable** | Respostas podem ser cacheadas | GET requests podem ser cacheadas |
| **Uniform Interface** | Interface padrão e consistente | URIs, métodos HTTP, representações |
| **Layered System** | Arquitetura em camadas | API → Service → DB |

### 3.2 Recursos e URI

**Conceito:** Tudo é recurso identificado por uma URI

```
Recurso: Trip (Viagem)

URI: /api/trips/{tripId}
├── GET /api/trips/{tripId}           → Recuperar viagem
├── POST /api/trips                    → Criar viagem
├── PUT /api/trips/{tripId}            → Atualizar viagem
└── DELETE /api/trips/{tripId}         → Deletar viagem

SubRecurso: Activities (Atividades dentro de uma viagem)

URI: /api/trips/{tripId}/activities
├── GET /api/trips/{tripId}/activities     → Listar atividades
├── POST /api/trips/{tripId}/activities    → Adicionar atividade
└── GET /api/trips/{tripId}/activities/{id} → Obter atividade
```

### 3.3 Métodos HTTP Semântica

```
GET    = Seguro, Idempotente    - Recuperar recurso
POST   = Não-idempotente         - Criar novo recurso
PUT    = Idempotente             - Substituir recurso completo
DELETE = Idempotente             - Remover recurso
PATCH  = Não-idempotente         - Atualizar parcial
```

### 3.4 Códigos de Status HTTP

| Código | Categoria | Significado |
|--------|-----------|-------------|
| 200 | 2xx Success | OK - Requisição bem-sucedida |
| 201 | 2xx Success | Created - Recurso criado |
| 204 | 2xx Success | No Content - Sem corpo na resposta |
| 400 | 4xx Client Error | Bad Request - Erro na entrada |
| 401 | 4xx Client Error | Unauthorized - Não autenticado |
| 403 | 4xx Client Error | Forbidden - Sem permissão |
| 404 | 4xx Client Error | Not Found - Recurso não existe |
| 500 | 5xx Server Error | Internal Server Error |

---

## 4. Conceitos de Banco de Dados

### 4.1 Modelo Entidade-Relacionamento (ER)

```
ENTIDADES E RELACIONAMENTOS:

┌──────────────┐
│    Trips     │
├──────────────┤
│ PK: id       │
│ destination  │
│ startsAt     │
│ endsAt       │
└──────────────┘
       ↑ 1
       |
       | N
┌──────────────┐          ┌──────────────┐
│ Activities   │          │ Participants │
├──────────────┤          ├──────────────┤
│ FK: tripId   │          │ FK: tripId   │
│ title        │          │ email        │
│ description  │          │ name         │
└──────────────┘          └──────────────┘
       
       ↑ 1                       ↑ 1
       |                         |
       | N                       | N
┌──────────────┐          
│    Links     │          
├──────────────┤          
│ FK: tripId   │          
│ title        │          
│ url          │          
└──────────────┘          
```

### 4.2 Normalização de Dados

**Objetivo:** Organizar dados para reduzir redundância

**Formas Normais (NF):**

1. **1NF (Primeira Forma Normal)**
   - Cada coluna contém apenas valores atômicos
   - Sem grupos repetidos

2. **2NF (Segunda Forma Normal)**
   - Cumpre 1NF
   - Todos os atributos não-chave dependem da chave primária inteira

3. **3NF (Terceira Forma Normal)**
   - Cumpre 2NF
   - Sem dependências transitivas entre atributos não-chave

**Aplicação no projeto:** Trip está em 3NF
- Trip tem dependência funcional com seus atributos (destination, startsAt)
- Activities têm chave estrangeira para Trip
- Sem redundância de dados

### 4.3 Transações ACID

Propriedades fundamentais:

| Propriedade | Conceito | Exemplo |
|-------------|----------|---------|
| **A**tomicity | Tudo ou nada | Se criar trip falhar, rollback automático |
| **C**onsistency | Estado válido | Chaves estrangeiras mantidas |
| **I**solation | Isolamento | Transação A não afeta B enquanto B executa |
| **D**urability | Persistência | Dados salvos sobrevivem a falhas |

---

## 5. ORM (Object-Relational Mapping)

### 5.1 Conceito

Mapeia automaticamente:

```
Mundo Orientado a Objetos    ↔    Mundo Relacional

Class Trip                        Table trips
├── id: UUID                      ├── id: BINARY(16)
├── destination: String           ├── destination: VARCHAR(255)
├── startsAt: LocalDateTime   ↔   ├── starts_at: TIMESTAMP
└── endsAt: LocalDateTime         └── ends_at: TIMESTAMP

Object instance                   Row in database
trip.destination = "Paris"        INSERT INTO trips (destination)
```

### 5.2 Vantagens do JPA/Hibernate

- **Abstração:** Código agnóstico de banco de dados
- **Lazy Loading:** Carrega dados sob demanda
- **Caching:** Reduz queries ao banco
- **Queries Dinâmicas:** Métodos derivados do repositório

### 5.3 EntityManager (Ciclo de Vida)

```
┌──────────────────────────────────────────┐
│       TRANSIENT (Novo)                   │
│  Entity não gerenciado pelo Hibernate    │
└──────────────────────────────────────────┘
              ↓ repository.save()
┌──────────────────────────────────────────┐
│       MANAGED (Persistido)               │
│  Entity monitorado pelo Hibernate        │
│  Alterações são rastreadas               │
└──────────────────────────────────────────┘
              ↓ sesión.evict()
┌──────────────────────────────────────────┐
│       DETACHED (Desanexado)              │
│  Entity fora da sessão                   │
│  Alterações não são rastreadas           │
└──────────────────────────────────────────┘
              ↓ repositorio.save()
┌──────────────────────────────────────────┐
│       MANAGED (Novamente)                │
└──────────────────────────────────────────┘
```

---

## 6. Conceitos de Migrations com Flyway

### 6.1 Versionamento Automático

```
BASE (Vazio)
    ↓
V1__create-table-trips.sql
    ├── CREATE TABLE trips (...)
    └── Success ✓
    
    ↓
V2__create-table-participants.sql
    ├── CREATE TABLE participants (...)
    └── Success ✓
    
    ↓
V3__create-table-activities.sql
    ├── CREATE TABLE activities (...)
    └── Success ✓
    
    ↓
V4__create-table-links.sql
    ├── CREATE TABLE links (...)
    └── Success ✓
    
    ↓
ATUAL (Pronto para uso)
```

### 6.2 Rastreamento de Migrações

Flyway mantém tabela `flyway_schema_history`:

| version | description | type | script | success |
|---------|-------------|------|--------|---------|
| 1 | create-table-trips | SQL | V1__... | 1 |
| 2 | create-table-participants | SQL | V2__... | 1 |
| 3 | create-table-activities | SQL | V3__... | 1 |
| 4 | create-table-links | SQL | V4__... | 1 |

---

## 7. Conceitos de Validação

### 7.1 Validação em Camadas

```
Camada de Apresentação (Controller)
├── Validação de sintaxe (@Valid)
└── Conversão de tipos

Camada de Negócio (Service)
├── Regras de negócio
├── Consistência de dados
└── Autorização

Camada de Persistência (Repository)
└── Constraints de banco de dados
    ├── NOT NULL
    ├── UNIQUE
    └── Foreign keys
```

### 7.2 Validação de Dados de Entrada

```java
public record TripRequestPayload(
    @NotNull(message = "Destination é obrigatória")
    @NotBlank(message = "Destination não pode ser vazia")
    String destination,
    
    @NotNull(message = "Data de início é obrigatória")
    @Future(message = "Data deve ser no futuro")
    LocalDateTime startsAt,
    
    @NotNull(message = "Data de fim é obrigatória")
    @Future(message = "Data deve ser no futuro")
    LocalDateTime endsAt
) { }
```

---

## 8. Conceitos de Resposta HTTP

### 8.1 Estrutura de Resposta de Sucesso

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "destination": "Paris",
  "startsAt": "2024-02-15T10:00:00",
  "endsAt": "2024-02-22T18:00:00",
  "description": "Viagem para Paris",
  "isConfirmed": false,
  "createdAt": "2024-01-10T15:30:00"
}
```

### 8.2 Estrutura de Resposta de Erro

```json
{
  "timestamp": "2024-01-17T14:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Destination é obrigatória",
  "path": "/trips"
}
```

---

## 9. Conceitos de Injeção de Dependência

### 9.1 Por que é Importante

```
SEM Injeção (Acoplamento Forte):
┌─────────────────────────────────────┐
│ TripService                         │
│ ├── new TripRepository()  ← Criada │
│ ├── new EmailService()   ← Criada  │
│ └── new NotificationService() ←  │
└─────────────────────────────────────┘
❌ Difícil de testar
❌ Difícil de substituir implementações
❌ Acoplamento forte

COM Injeção (Desacoplamento):
┌─────────────────────────────────────┐
│ TripService                         │
│ ├── TripRepository (injetado)      │
│ ├── EmailService (injetado)        │
│ └── NotificationService (injetado) │
└─────────────────────────────────────┘
✓ Fácil de testar com mocks
✓ Fácil trocar implementação
✓ Baixo acoplamento
```

### 9.2 Tipos de Injeção

```java
// 1. Por anotação
@Service
public class TripService {
    @Autowired
    private TripRepository repository;
}

// 2. Por construtor (Recomendado)
@Service
public class TripService {
    private final TripRepository repository;
    
    public TripService(TripRepository repository) {
        this.repository = repository;
    }
}

// 3. Por setter
@Service
public class TripService {
    private TripRepository repository;
    
    @Autowired
    public void setRepository(TripRepository repository) {
        this.repository = repository;
    }
}
```

---

## 10. Conceitos de Segurança

### 10.1 Princípio do Menor Privilégio

```
Implementar no projeto:
├── Validação de entrada rigorosa
├── Sanitização de dados
├── Tratamento seguro de erros
├── Proteção de dados sensíveis
├── Autenticação e autorização
└── HTTPS obrigatório
```

### 10.2 OWASP Top 10 (Consciência)

| Vulnerabilidade | Proteção |
|-----------------|----------|
| Injection | Usar JPA ao invés de SQL puro |
| Broken Authentication | Implementar OAuth2/JWT |
| Sensitive Data Exposure | HTTPS + criptografia |
| XML External Entities | Validar entrada |
| Access Control | Verificar permissões |

---

## 11. Conceitos de Performance

### 11.1 N+1 Query Problem

```
❌ Problema:
for (Trip trip : trips) {
    System.out.println(trip.getParticipants()); // 1 query por trip
}
// 1 query para trips + N queries para participants = N+1 queries

✓ Solução:
@Query("SELECT t FROM Trip t JOIN FETCH t.participants")
List<Trip> findAllWithParticipants();
// 1 query com JOIN = melhor performance
```

### 11.2 Lazy vs Eager Loading

```java
// LAZY (Padrão)
@OneToMany(fetch = FetchType.LAZY)
private List<Activity> activities;
// Activities carregadas sob demanda

// EAGER
@OneToMany(fetch = FetchType.EAGER)
private List<Activity> activities;
// Activities carregadas com a Trip
```

---

## 12. Conceitos de Testes

### 12.1 Pirâmide de Testes

```
        ▲
       /△\
      / △ \     Testes E2E (poucos)
     /  △  \    APIs completas, frontend
    /────────\
   /   △ △   \  Testes de Integração (meio)
  /  △     △  \ Services + Repositories
 /──────────────\
/  △  △  △  △  \ Testes Unitários (muitos)
────────────────── Classes isoladas, mocks
```

### 12.2 Exemplo de Teste Unitário

```java
@ExtendWith(MockitoExtension.class)
class TripServiceTest {
    
    @Mock
    private TripRepository repository;
    
    @InjectMocks
    private TripService service;
    
    @Test
    void shouldCreateTripSuccessfully() {
        // Arrange
        TripRequestPayload payload = new TripRequestPayload(
            "Paris",
            LocalDateTime.now().plusDays(1),
            LocalDateTime.now().plusDays(8)
        );
        
        // Act
        Trip result = service.createTrip(payload);
        
        // Assert
        assertNotNull(result.getId());
        assertEquals("Paris", result.getDestination());
    }
}
```

---

## 13. Conceitos de Escalabilidade

### 13.1 Escalabilidade Vertical vs Horizontal

```
Vertical (Scale-Up):
┌──────────────┐
│  Servidor    │
│  ↑ RAM       │
│  ↑ CPU       │
│  ↑ Disco     │
└──────────────┘
❌ Limite físico
❌ Downtime necessário

Horizontal (Scale-Out):
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Servidor 1  │  │  Servidor 2  │  │  Servidor N  │
└──────────────┘  └──────────────┘  └──────────────┘
        ↓                 ↓                  ↓
        └─────────────────┴──────────────────┘
                   Load Balancer
✓ Infinitamente escalável
✓ Sem downtime
```

### 13.2 Padrões de Escalabilidade

```
Stateless Application:
┌────────────────────────────────────┐
│ Cada requisição é independente     │
│ Pode ser processada por qualquer   │
│ servidor sem perder contexto       │
└────────────────────────────────────┘
✓ Ideal para aplicações distribuídas
```

---

## 14. Conceitos de Logging

### 14.1 Níveis de Log

| Nível | Uso |
|-------|-----|
| TRACE | Informações muito detalhadas |
| DEBUG | Informações para debugging |
| INFO | Eventos normais e importantes |
| WARN | Situações inesperadas |
| ERROR | Erros em operações |
| FATAL | Erro crítico que para app |

### 14.2 Exemplo de Logging

```java
@Service
public class TripService {
    private static final Logger logger = LoggerFactory.getLogger(TripService.class);
    
    public Trip createTrip(TripRequestPayload payload) {
        logger.info("Criando nova trip para destino: {}", payload.destination);
        
        try {
            Trip trip = new Trip(payload);
            tripRepository.save(trip);
            logger.info("Trip criada com sucesso. ID: {}", trip.getId());
            return trip;
        } catch (Exception e) {
            logger.error("Erro ao criar trip", e);
            throw new TripCreationException("Falha ao criar trip", e);
        }
    }
}
```

---

## Conclusão

Os conceitos apresentados formam a base teórica para:

1. **Compreender a arquitetura** da aplicação
2. **Aplicar padrões de design** apropriados
3. **Implementar APIs REST** corretamente
4. **Garantir qualidade de código** através de testes
5. **Escalar a aplicação** conforme necessário
6. **Manter a aplicação** segura e performática

O RocketSeat NLW Journey exemplifica a aplicação prática desses conceitos em uma aplicação web moderna.
