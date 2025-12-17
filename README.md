# gRPC - Guia Completo

> Este guia foi elaborado por **Enéas Almeida** com o principal objetivo de facilitar os repasses de informações à equipe.

```
  ____  ____   ____   ____
 / ___|  _ \ / ___| / ___|
| |  _| |_) | |    | |
| |_| |  _ <| |___ | |___
 \____|_| \_\\____| \____|
```

## 📋 Índice

- [O que é gRPC?](#o-que-é-grpc)
- [Características Principais](#características-principais)
- [Onde Utilizar](#onde-é-ideal-para-utilizar)
- [Conceitos Fundamentais](#conceitos-fundamentais)
- [Protocol Buffers](#protocol-buffers)
- [HTTP/2](#http2)
- [Tipos de Comunicação](#formatos-de-tráfego-entre-comunicação-grpc)
- [REST vs gRPC](#rest-vs-grpc)
- [Instalação e Configuração](#pré-requisitos)
- [Comandos Úteis](#comandos)

## O que é gRPC?

**gRPC** (gRPC Remote Procedure Call) é um framework moderno de comunicação entre sistemas desenvolvido pelo Google que permite a comunicação eficiente e rápida entre serviços distribuídos.

### Características Principais

-   **Desenvolvedor:** Google
-   **Mantenedor:** CNCF (Cloud Native Computing Foundation) - mesma organização que mantém Kubernetes e OpenTelemetry
-   **Lançamento:** Fevereiro de 2015
-   **Licença:** Código aberto
-   **Protocolo:** HTTP/2
-   **Serialização:** Protocol Buffers (protobuf)
-   **Performance:** Alta velocidade com baixa latência
-   **Segurança:** Suporte nativo a TLS/SSL
-   **Independência:** Agnóstico de linguagem de programação

## Links importantes

-   [grpc.io](https://grpc.io/) - Site oficial do gRPC.
-   [protobuf.dev](https://protobuf.dev/) - Manual Protocol Buffers.

## Onde é ideal para utilizar?

-   Microsserviços;
-   Aplicações em tempo real;
-   Sistemas distribuídos;
-   Aplicações IOT;
-   Streaming de dados.

## Linguagens com suporte oficial

-   Go
-   Java
-   C

Através do gRPC-C é possível utilizar python, nodejs, kotlin e etc.

## Conceitos Fundamentais

### O que significa na prática o Remote Procedure Call (RPC)?

RPC permite que um programa execute uma função/procedimento em outro espaço de endereçamento (geralmente em outra máquina) como se fosse uma chamada local.

```
┌─────────────────────────────────────────────────────────────────┐
│                     ARQUITETURA gRPC                            │
└─────────────────────────────────────────────────────────────────┘

    Cliente                                         Servidor
    ┌──────┐                                        ┌──────┐
    │      │                                        │      │
    │ App  │                                        │ App  │
    │      │                                        │      │
    └──┬───┘                                        └───┬──┘
       │                                                │
       │  ┌──────────────────────────────────┐         │
       │  │    1. Chamada de Método         │         │
       ├──┼─────────────────────────────────►│         │
       │  │    getAccount(id: "123")        │         │
       │  └──────────────────────────────────┘         │
       │                                                │
    ┌──┴───────┐                              ┌────────┴───┐
    │  gRPC    │   ◄───── HTTP/2 ─────►      │   gRPC     │
    │  Stub    │                              │   Server   │
    └──┬───────┘                              └────────┬───┘
       │                                                │
       │  ┌──────────────────────────────────┐         │
       │  │    2. Serialização Protobuf     │         │
       │  │    ┌────────────────┐           │         │
       │  │    │ Binary Data    │           │         │
       │  │    │ [01010101...]  │           │         │
       │  │    └────────────────┘           │         │
       │  └──────────────────────────────────┘         │
       │                                                │
       │  ┌──────────────────────────────────┐         │
       │  │    3. Transporte HTTP/2          │         │
       ├──┼─────────────────────────────────►│         │
       │  │    Headers + Binary Payload      │         │
       │  └──────────────────────────────────┘         │
       │                                                │
       │                                             ┌──┴───┐
       │                                             │ Exec │
       │                                             │ Func │
       │                                             └──┬───┘
       │                                                │
       │  ┌──────────────────────────────────┐         │
       │  │    4. Resposta (Protobuf)        │         │
       │  │◄─────────────────────────────────┼─────────┤
       │  │    Account{id, name, email}      │         │
       │  └──────────────────────────────────┘         │
       │                                                │
    ┌──┴───────┐                                    ┌───┴──┐
    │ Processa │                                    │      │
    │ Resposta │                                    │      │
    └──────────┘                                    └──────┘


Fluxo Detalhado:
─────────────────

1. Cliente chama método como se fosse local
2. gRPC Stub serializa parâmetros (Protobuf)
3. Dados binários são enviados via HTTP/2
4. Servidor deserializa e executa a função
5. Resultado é serializado e retornado
6. Cliente recebe e deserializa a resposta
```

**Evolução Histórica:**
- **Passado:** XML-RPC, SOAP (XML) - verboso e lento
- **Presente:** gRPC (Protobuf) - compacto e rápido
- **Vantagem:** Contratos fortemente tipados (.proto files)

## Protocol Buffers

Protocol Buffers (Protobuf) é uma linguagem de definição de interface (IDL) criada pelo Google para serialização estruturada de dados.

### Características do Protocol Buffers

```
┌────────────────────────────────────────────────────────────────┐
│                    PROTOCOL BUFFERS WORKFLOW                    │
└────────────────────────────────────────────────────────────────┘

1. DEFINIÇÃO DO CONTRATO (.proto)
   ┌─────────────────────────────────────┐
   │ syntax = "proto3";                  │
   │                                     │
   │ message Account {                   │
   │   string id = 1;                    │
   │   string name = 2;                  │
   │   string email = 3;                 │
   │ }                                   │
   └─────────────────────────────────────┘
                  │
                  │ protoc compiler
                  ▼
2. GERAÇÃO DE CÓDIGO
   ┌─────────────────────────────────────┐
   │  Go        │  Java    │  Python     │
   │  ────────  │  ──────  │  ─────────  │
   │  account.  │  Account │  account_   │
   │  pb.go     │  .java   │  pb2.py     │
   └─────────────────────────────────────┘
                  │
                  │
                  ▼
3. SERIALIZAÇÃO (Objeto → Binário)
   ┌─────────────────────────────────────┐
   │ Account Object                      │
   │ ┌─────────────────────────────────┐ │
   │ │ id: "12345"                     │ │
   │ │ name: "João Silva"              │ │
   │ │ email: "joao@email.com"         │ │
   │ └─────────────────────────────────┘ │
   └─────────────────────────────────────┘
                  │
                  │ Marshal/Encode
                  ▼
   ┌─────────────────────────────────────┐
   │ Binary Format (Compact)             │
   │ [0x0a 0x05 0x31 0x32 0x33 0x34...]  │
   │ Tamanho: ~45 bytes                  │
   └─────────────────────────────────────┘
                  │
                  │ Network Transfer
                  ▼
4. DESERIALIZAÇÃO (Binário → Objeto)
   ┌─────────────────────────────────────┐
   │ Unmarshal/Decode                    │
   │                                     │
   │ Account Object (Reconstruído)       │
   └─────────────────────────────────────┘
```

### Vantagens do Protocol Buffers

-   **Formato Binário:** Dados compactos e eficientes
-   **Contratos Tipados:** Validação em tempo de compilação
-   **Serialização Rápida:** Alto desempenho (CPU eficiente)
-   **Baixo Consumo de Rede:** Arquivos 3-10x menores que JSON
-   **Retrocompatibilidade:** Evolução de schema sem quebrar compatibilidade
-   **Multi-linguagem:** Geração automática de código
-   **Independente:** Pode ser usado sem gRPC

### Protocol Buffers vs JSON

```
┌────────────────────────────────────────────────────────────────┐
│              COMPARAÇÃO: PROTOBUF vs JSON                      │
└────────────────────────────────────────────────────────────────┘

EXEMPLO: Objeto Account
──────────────────────────────────────────────────────────────────

JSON (Texto)                      │  Protobuf (Binário)
──────────────────────────────────┼──────────────────────────────
{                                 │  [Binary Data]
  "id": "12345",                  │  0x0a 0x05 0x31 0x32 0x33
  "name": "João Silva",           │  0x34 0x35 0x12 0x0b 0x4a
  "email": "joao@email.com"       │  0xc3 0xa3 0x6f 0x20 0x53
}                                 │  ... [compressed]
                                  │
Tamanho: ~98 bytes                │  Tamanho: ~45 bytes
──────────────────────────────────┼──────────────────────────────
✗ Texto legível                   │  ✓ Binário otimizado
✗ Parsing mais lento              │  ✓ Parsing 3-10x mais rápido
✗ Mais bytes na rede              │  ✓ Menos uso de banda
✗ Sem validação de tipo           │  ✓ Validação forte de tipos
✓ Human-readable                  │  ✗ Não legível (requer decode)
✓ Debugging mais fácil            │  ✗ Requer ferramentas especiais


PERFORMANCE BENCHMARK
──────────────────────────────────────────────────────────────────
Métrica              │ JSON       │ Protobuf    │ Ganho
─────────────────────┼────────────┼─────────────┼─────────
Serialização         │ 1000 ns    │ 300 ns      │ 3.3x
Deserialização       │ 1200 ns    │ 400 ns      │ 3.0x
Tamanho (10KB JSON)  │ 10,000 B   │ 3,000 B     │ 3.3x
CPU (Serialize 1M)   │ 100%       │ 30%         │ 3.3x
Uso de Memória       │ Alto       │ Baixo       │ 2-3x
```

### Estrutura de um Arquivo .proto

```proto
syntax = "proto3";  // Versão do Protocol Buffers

package pb;  // Namespace do pacote

option go_package = "./pb";  // Caminho de geração para Go

// Definição de mensagem (estrutura de dados)
message Account {
  string id = 1;      // Campo 1: identificador único
  string name = 2;    // Campo 2: nome da conta
  string email = 3;   // Campo 3: email da conta
}

// Definição de serviço (APIs disponíveis)
service AccountService {
  rpc CreateAccount (Account) returns (Account);
  rpc GetAccount (AccountRequest) returns (Account);
  rpc ListAccounts (Empty) returns (stream Account);
}

message AccountRequest {
  string id = 1;
}

message Empty {}
```

**Padrão:** Arquivo `.proto` (protofile)
**Versão Recomendada:** `proto3` (para gRPC)
**Compilador:** `protoc` (Protocol Buffer Compiler)

## HTTP/2

HTTP/2 é a base de transporte do gRPC, oferecendo recursos avançados para comunicação eficiente.

### Características do HTTP/2

```
┌────────────────────────────────────────────────────────────────┐
│                HTTP/1.1 vs HTTP/2 COMPARISON                   │
└────────────────────────────────────────────────────────────────┘

HTTP/1.1 (Texto)                  HTTP/2 (Binário)
─────────────────────────────────────────────────────────────────

MÚLTIPLAS REQUISIÇÕES:
──────────────────────────────────────────────────────────────────
Cliente          Servidor        Cliente          Servidor
   │                │               │                │
   ├──── Req 1 ────►│               ├──┬─ Req 1 ────►│
   │                │               │  ├─ Req 2 ────►│
   │◄─── Res 1 ─────┤               │  └─ Req 3 ────►│
   │                │               │                │
   ├──── Req 2 ────►│               │◄─┬─ Res 1 ─────┤
   │                │               │  ├─ Res 3 ─────┤
   │◄─── Res 2 ─────┤               │  └─ Res 2 ─────┤
   │                │               │                │
   ├──── Req 3 ────►│             MULTIPLEXING:
   │                │             Uma única conexão TCP!
   │◄─── Res 3 ─────┤
   │                │
3 conexões TCP                    1 conexão TCP


ESTRUTURA DE FRAMES:
──────────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────────┐
│                      HTTP/2 FRAME                           │
├───────────┬─────────────────────────────────────────────────┤
│  Header   │  Length (24) │ Type (8) │ Flags (8) │ Stream  │
│  (9 bytes)│              │          │           │  ID(31) │
├───────────┼─────────────────────────────────────────────────┤
│  Payload  │                                                 │
│  (N bytes)│              Frame Data                         │
│           │                                                 │
└───────────┴─────────────────────────────────────────────────┘

Frame Types:
• HEADERS    - Metadados da requisição/resposta
• DATA       - Payload da mensagem
• SETTINGS   - Configurações da conexão
• PING       - Keep-alive
• GOAWAY     - Encerramento da conexão


MULTIPLEXING EM AÇÃO:
──────────────────────────────────────────────────────────────────
    Conexão TCP Única
    ═════════════════════════════════════════════

    Stream 1  ────►  [HEADERS] [DATA] [DATA]
    Stream 3  ────►         [HEADERS] [DATA]
    Stream 5  ────►  [HEADERS] [DATA]
    Stream 7  ────►              [HEADERS] [DATA]

    ⬇ Mesma Conexão TCP ⬇

    Stream 1  ◄────  [HEADERS] [DATA]
    Stream 3  ◄────         [HEADERS] [DATA]
    Stream 5  ◄────              [HEADERS] [DATA]
    Stream 7  ◄────  [HEADERS] [DATA] [DATA]


SERVER PUSH:
──────────────────────────────────────────────────────────────────
Cliente                          Servidor
   │                                │
   ├──── Request: index.html ──────►│
   │                                │
   │◄──── Response: index.html ─────┤
   │◄──── Push: style.css ──────────┤  (Antecipado!)
   │◄──── Push: script.js ──────────┤  (Antecipado!)
   │                                │
   (Cliente recebe recursos antes de solicitar)


COMPRESSÃO DE HEADERS (HPACK):
──────────────────────────────────────────────────────────────────
Request 1:
┌────────────────────────────────────────┐
│ :method: GET                           │
│ :path: /api/accounts/123               │
│ :authority: api.example.com            │
│ user-agent: grpc-go/1.50.0             │
│ content-type: application/grpc+proto   │
└────────────────────────────────────────┘
Tamanho: ~200 bytes

Request 2 (mesma conexão):
┌────────────────────────────────────────┐
│ [2] [3]                    ← Referências
│ :path: /api/accounts/456   ← Só o diff
└────────────────────────────────────────┘
Tamanho: ~15 bytes (93% menor!)
```

### Benefícios do HTTP/2 para gRPC

-   **Origem:** Criado pela Google como projeto SPDY
-   **Lançamento:** Maio de 2015 (RFC 7540)
-   **Formato Binário:** Parsing mais eficiente que texto
-   **Multiplexing:** Múltiplas requisições simultâneas em uma conexão TCP
-   **Server Push:** Servidor pode enviar recursos proativamente
-   **Header Compression (HPACK):** Redução de overhead
-   **Priorização de Streams:** Controle de precedência de requisições
-   **Flow Control:** Gerenciamento de backpressure
-   **Baixa Latência:** Reduz roundtrips
-   **Economia de Recursos:** Menos conexões TCP = menos overhead

### HTTP/2 vs HTTP/1.1

| Característica | HTTP/1.1 | HTTP/2 |
|---|---|---|
| **Formato** | Texto | Binário |
| **Conexões** | Múltiplas (6-8 por host) | Única por host |
| **Multiplexing** | Não | Sim |
| **Header Compression** | Não | Sim (HPACK) |
| **Server Push** | Não | Sim |
| **Priorização** | Não | Sim |
| **Performance** | Moderada | Alta |
| **Latência** | Alta (head-of-line blocking) | Baixa |

## Formatos de tráfego entre comunicação gRPC

gRPC suporta 4 padrões de comunicação distintos, cada um otimizado para diferentes casos de uso.

### 1. Unary RPC (Requisição-Resposta Simples)

O padrão mais comum, similar ao REST tradicional: uma requisição, uma resposta.

```
┌──────────────────────────────────────────────────────────────┐
│                      UNARY RPC FLOW                          │
└──────────────────────────────────────────────────────────────┘

Cliente                                    Servidor
────────                                   ─────────

  [App]                                      [App]
    │                                          │
    │  1. Chamada do método                   │
    ├──────────────────────────────────────►  │
    │  GetAccount(id: "123")                  │
    │                                          │
    │                                       ┌──┴──┐
    │                                       │Query│
    │                                       │ DB  │
    │                                       └──┬──┘
    │                                          │
    │  2. Resposta única                      │
    │  ◄──────────────────────────────────────┤
    │  Account{id, name, email}               │
    │                                          │
    ▼                                          ▼
 [Processa]                                 [Done]


EXEMPLO DE CÓDIGO (.proto):
────────────────────────────────────────────────────────────────
service AccountService {
  rpc GetAccount(AccountRequest) returns (Account);
}

USO TÍPICO:
• APIs CRUD básicas (Create, Read, Update, Delete)
• Validações simples
• Operações síncronas
• Substituição direta de REST/HTTP APIs


FLUXO TEMPORAL:
────────────────────────────────────────────────────────────────
t=0ms    Cliente envia requisição
t=50ms   Servidor recebe e processa
t=100ms  Servidor envia resposta
t=150ms  Cliente recebe resposta

Total: ~150ms (roundtrip completo)
```

### 2. Server Streaming RPC (Servidor Envia Múltiplas Respostas)

Cliente envia uma requisição e recebe um stream de múltiplas respostas do servidor.

```
┌──────────────────────────────────────────────────────────────┐
│                  SERVER STREAMING RPC FLOW                   │
└──────────────────────────────────────────────────────────────┘

Cliente                                    Servidor
────────                                   ─────────

  [App]                                      [App]
    │                                          │
    │  1. Requisição única                    │
    ├──────────────────────────────────────►  │
    │  ListAccounts(filter)                   │
    │                                          │
    │                                       ┌──┴──┐
    │  2. Stream de respostas               │Query│
    │  ◄──────────────────────────────────┐ │ DB  │
    │  Account #1                         │ └──┬──┘
    ├─► [Processa]                        │    │
    │                                     │    │
    │  ◄──────────────────────────────────┤    │
    │  Account #2                         │    │
    ├─► [Processa]                        │    │
    │                                     │    │
    │  ◄──────────────────────────────────┤    │
    │  Account #3                         │    │
    ├─► [Processa]                        │    │
    │                                     │    │
    │  ◄──────────────────────────────────┤    │
    │  Account #N                         │    │
    ├─► [Processa]                        │    │
    │                                     │    │
    │  ◄──────────────────────────────────┘    │
    │  [END OF STREAM]                         │
    │                                          │
    ▼                                          ▼
 [Complete]                                 [Done]


EXEMPLO DE CÓDIGO (.proto):
────────────────────────────────────────────────────────────────
service AccountService {
  rpc ListAccounts(ListRequest) returns (stream Account);
}

USO TÍPICO:
• Listagem de grandes volumes de dados
• Relatórios e exportações
• Logs em tempo real
• Notificações push
• Dashboards com dados ao vivo
• Download de arquivos em chunks

VANTAGENS:
• Cliente processa dados incrementalmente (menos memória)
• Servidor pode enviar dados conforme processa (streaming)
• Feedback mais rápido (primeira resposta chega antes)
• Ideal para datasets grandes


FLUXO TEMPORAL:
────────────────────────────────────────────────────────────────
t=0ms     Cliente envia requisição
t=50ms    Servidor envia Account #1   ──► Cliente processa
t=100ms   Servidor envia Account #2   ──► Cliente processa
t=150ms   Servidor envia Account #3   ──► Cliente processa
t=200ms   Servidor envia Account #N   ──► Cliente processa
t=250ms   Stream finalizado

Total: Cliente começa a processar em ~50ms!
```

### 3. Client Streaming RPC (Cliente Envia Múltiplas Requisições)

Cliente envia um stream de requisições e recebe uma única resposta do servidor.

```
┌──────────────────────────────────────────────────────────────┐
│                  CLIENT STREAMING RPC FLOW                   │
└──────────────────────────────────────────────────────────────┘

Cliente                                    Servidor
────────                                   ─────────

  [App]                                      [App]
    │                                          │
    │  1. Abre stream                         │
    ├──────────────────────────────────────►  │
    │                                          │
    │  2. Envia dados #1                    ┌──┴──┐
    ├──────────────────────────────────────►│Buffer│
    │  Account{...}                         └──┬──┘
    │                                          │
    │  3. Envia dados #2                      │
    ├──────────────────────────────────────►  │
    │  Account{...}                           │
    │                                          │
    │  4. Envia dados #3                      │
    ├──────────────────────────────────────►  │
    │  Account{...}                           │
    │                                          │
    │  5. Envia dados #N                      │
    ├──────────────────────────────────────►  │
    │  Account{...}                           │
    │                                          │
    │  6. Fecha stream (EOF)                  │
    ├──────────────────────────────────────►  │
    │                                          │
    │                                       ┌──┴──┐
    │                                       │Batch│
    │                                       │Save │
    │                                       └──┬──┘
    │                                          │
    │  7. Resposta final                      │
    │  ◄──────────────────────────────────────┤
    │  Summary{total: N, success: M}          │
    │                                          │
    ▼                                          ▼
 [Complete]                                 [Done]


EXEMPLO DE CÓDIGO (.proto):
────────────────────────────────────────────────────────────────
service AccountService {
  rpc CreateAccounts(stream Account) returns (Summary);
}

USO TÍPICO:
• Upload de arquivos grandes em chunks
• Importação em lote (batch insert)
• Telemetria e métricas (envio contínuo)
• Agregações de dados
• Backup incremental

VANTAGENS:
• Cliente envia dados conforme disponíveis
• Servidor processa em lote (mais eficiente)
• Reduz overhead de múltiplas conexões
• Ideal para uploads e batch operations


FLUXO TEMPORAL:
────────────────────────────────────────────────────────────────
t=0ms     Cliente abre stream
t=10ms    Cliente envia Account #1
t=20ms    Cliente envia Account #2
t=30ms    Cliente envia Account #3
t=40ms    Cliente envia Account #N
t=50ms    Cliente fecha stream
t=100ms   Servidor processa todos os dados
t=150ms   Servidor envia resposta final

Total: ~150ms (mas dados enviados incrementalmente)
```

### 4. Bidirectional Streaming RPC (Comunicação Bidirecional)

Tanto cliente quanto servidor enviam streams de dados de forma independente e assíncrona.

```
┌──────────────────────────────────────────────────────────────┐
│              BIDIRECTIONAL STREAMING RPC FLOW                │
└──────────────────────────────────────────────────────────────┘

Cliente                                    Servidor
────────                                   ─────────

  [App]                                      [App]
    │                                          │
    │  1. Abre stream bidirecional            │
    ├═════════════════════════════════════════►│
    │                                          │
    │  2. Cliente envia msg #1                │
    ├──────────────────────────────────────►  │
    │  CreateAccount{name: "João"}            │
    │                                          │
    │                                       [Processa]
    │                                          │
    │  3. Servidor responde msg #1            │
    │  ◄──────────────────────────────────────┤
    │  Account{id: "123", name: "João"}       │
    ├─► [Processa]                            │
    │                                          │
    │  4. Cliente envia msg #2                │
    ├──────────────────────────────────────►  │
    │  CreateAccount{name: "Maria"}           │
    │                                          │
    │  5. Servidor responde msg #2            │
    │  ◄──────────────────────────────────────┤
    │  Account{id: "124", name: "Maria"}      │
    ├─► [Processa]                            │
    │                                          │
    │  6. Cliente envia msg #3                │
    ├──────────────────────────────────────►  │
    │  CreateAccount{name: "Pedro"}           │
    │                                          │
    │  7. Servidor responde msg #3            │
    │  ◄──────────────────────────────────────┤
    │  Account{id: "125", name: "Pedro"}      │
    ├─► [Processa]                            │
    │                                          │
    │  ◄──────────────────────────────────────┤
    │  [Servidor pode enviar a qualquer momento]
    │                                          │
    │  8. Ambos fecham stream                 │
    ├═════════════════════════════════════════►│
    │                                          │
    ▼                                          ▼
 [Complete]                                 [Done]


CARACTERÍSTICAS IMPORTANTES:
────────────────────────────────────────────────────────────────
• Os streams são INDEPENDENTES:
  - Cliente pode enviar sem esperar resposta
  - Servidor pode responder fora de ordem
  - Ambos podem enviar/receber simultaneamente

• Ordem não é garantida (por padrão)
• Full-duplex: comunicação simultânea nos dois sentidos
• Streams podem ser fechados independentemente


EXEMPLO DE CÓDIGO (.proto):
────────────────────────────────────────────────────────────────
service AccountService {
  rpc CreateAccountsStream(stream Account) returns (stream Account);
}

USO TÍPICO:
• Chat em tempo real
• Jogos multiplayer
• Trading de alta frequência
• Sincronização de dados
• Colaboração em tempo real (Google Docs style)
• IoT com feedback bidirecional
• Video/Audio streaming com controles

VANTAGENS:
• Latência ultra-baixa
• Comunicação full-duplex
• Não bloqueia (totalmente assíncrono)
• Ideal para aplicações interativas em tempo real


FLUXO TEMPORAL (Exemplo):
────────────────────────────────────────────────────────────────
t=0ms     Stream aberto
t=10ms    Cliente → Servidor (msg #1)
t=20ms    Servidor → Cliente (response #1)
t=25ms    Cliente → Servidor (msg #2)
t=30ms    Cliente processa response #1
t=35ms    Servidor → Cliente (response #2)
t=40ms    Cliente → Servidor (msg #3)
t=45ms    Cliente processa response #2
t=50ms    Servidor → Cliente (response #3)
...
(Comunicação contínua e assíncrona)
```

### Comparação dos 4 Padrões

```
┌────────────────────────────────────────────────────────────────┐
│           COMPARAÇÃO DOS PADRÕES DE COMUNICAÇÃO                │
└────────────────────────────────────────────────────────────────┘

Padrão              │ Cliente → │ Servidor → │ Caso de Uso
                    │  Servidor │  Cliente   │
────────────────────┼───────────┼────────────┼────────────────────
1. Unary            │  1 msg    │  1 msg     │ CRUD, APIs simples
                    │           │            │
2. Server Streaming │  1 msg    │  N msgs    │ Listagens, logs,
                    │           │            │ notificações
                    │           │            │
3. Client Streaming │  N msgs   │  1 msg     │ Upload, batch,
                    │           │            │ métricas
                    │           │            │
4. Bidirectional    │  N msgs   │  N msgs    │ Chat, jogos,
    Streaming       │           │            │ real-time sync
────────────────────┴───────────┴────────────┴────────────────────


ESCOLHA DO PADRÃO:
──────────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────────┐
│  Precisa enviar/receber múltiplas mensagens?                │
│                                                             │
│  Não           Sim, mas só uma direção        Sim, ambas   │
│   │                    │                          │        │
│   ▼                    ▼                          ▼        │
│ UNARY     Quem envia múltiplas?        BIDIRECTIONAL      │
│           │                   │           STREAMING        │
│           ▼                   ▼                            │
│        Cliente           Servidor                          │
│           │                   │                            │
│           ▼                   ▼                            │
│     CLIENT              SERVER                             │
│     STREAMING           STREAMING                          │
└─────────────────────────────────────────────────────────────┘
```

## REST vs gRPC

Comparação detalhada entre os dois paradigmas de comunicação mais populares para APIs.

```
┌────────────────────────────────────────────────────────────────┐
│                    REST vs gRPC COMPARISON                     │
└────────────────────────────────────────────────────────────────┘

ARQUITETURA DE COMUNICAÇÃO:
────────────────────────────────────────────────────────────────

REST API                           gRPC API
────────────────────────────────   ────────────────────────────────

Cliente                            Cliente
  │                                  │
  │ HTTP/1.1                         │ HTTP/2
  │ JSON (Texto)                     │ Protobuf (Binário)
  │                                  │
  ├─► POST /api/accounts             ├─► CreateAccount()
  │   {                              │   Account{...}
  │     "name": "João",              │
  │     "email": "j@mail.com"        │
  │   }                              │
  │                                  │
  │◄─ 201 Created                    │◄─ Account{id, name, email}
  │   {                              │
  │     "id": "123",                 │
  │     "name": "João",              │
  │     "email": "j@mail.com"        │
  │   }                              │
  │                                  │
Servidor                           Servidor


CARACTERÍSTICAS TÉCNICAS:
────────────────────────────────────────────────────────────────

┌─────────────────────┬──────────────────┬──────────────────┐
│   Característica    │      REST        │      gRPC        │
├─────────────────────┼──────────────────┼──────────────────┤
│ Formato de Dados    │ JSON (Texto)     │ Protobuf (Bin.)  │
│ Protocolo           │ HTTP/1.1         │ HTTP/2           │
│ Contrato            │ OpenAPI/Swagger  │ .proto (forte)   │
│ Geração de Código   │ Manual/Opcional  │ Automática       │
│ Streaming           │ ✗ Não suportado  │ ✓ Bidirecional   │
│ Browser Support     │ ✓ Nativo         │ ✗ Requer proxy   │
│ Multiplexing        │ ✗ Não            │ ✓ Sim            │
│ Latência            │ Alta             │ Baixa            │
│ Payload Size        │ Grande           │ Pequeno (60%)    │
│ Verbos/Métodos      │ GET/POST/PUT/DEL │ Customizáveis    │
│ Tipo de Comunicação │ Unidirecional    │ Bidirecional     │
│ Debugging           │ Fácil (cURL)     │ Requer tools     │
│ Caching             │ ✓ HTTP Cache     │ ✗ Complexo       │
│ Load Balancing      │ ✓ Padrão         │ ✓ Com config     │
└─────────────────────┴──────────────────┴──────────────────┘


EXEMPLO PRÁTICO - CRIAR UMA CONTA:
────────────────────────────────────────────────────────────────

REST:
─────
Request:
  POST /api/v1/accounts HTTP/1.1
  Host: api.example.com
  Content-Type: application/json
  Authorization: Bearer token123

  {
    "name": "João Silva",
    "email": "joao@example.com"
  }

Response:
  HTTP/1.1 201 Created
  Content-Type: application/json

  {
    "id": "acc_123",
    "name": "João Silva",
    "email": "joao@example.com",
    "created_at": "2024-01-15T10:30:00Z"
  }

  Tamanho Total: ~250 bytes


gRPC:
─────
Request:
  POST /AccountService/CreateAccount HTTP/2
  content-type: application/grpc+proto

  [Binary Protobuf Data]
  0x0a 0x0b 0x4a 0xc3 0xa3 0x6f 0x20 0x53...

Response:
  HTTP/2 200 OK
  content-type: application/grpc+proto

  [Binary Protobuf Data]
  0x0a 0x07 0x61 0x63 0x63 0x5f 0x31 0x32...

  Tamanho Total: ~80 bytes (68% menor!)


QUANDO USAR CADA UM:
────────────────────────────────────────────────────────────────

USE REST QUANDO:                   USE gRPC QUANDO:
───────────────────                ────────────────

✓ APIs públicas/externas           ✓ Microserviços internos
✓ Navegadores como clientes        ✓ Comunicação servidor-servidor
✓ Simplicidade é prioridade        ✓ Performance crítica
✓ Caching HTTP importante          ✓ Streaming necessário
✓ Ferramentas familiares           ✓ Baixa latência essencial
✓ Documentação OpenAPI             ✓ Contratos fortemente tipados
✓ Compatibilidade ampla            ✓ Polyglot (múltiplas linguagens)
✓ Debug fácil (cURL, Postman)      ✓ Real-time communication


PERFORMANCE COMPARISON:
────────────────────────────────────────────────────────────────

Métrica                  REST          gRPC         Melhoria
──────────────────────   ────────      ──────       ────────
Latência (1 req)         50ms          20ms         2.5x
Throughput               1000 req/s    7000 req/s   7x
Payload (10KB JSON)      10,000 bytes  3,500 bytes  65% ↓
Serialização             1000ns        300ns        3.3x
CPU (10K reqs)           100%          40%          60% ↓
Memória                  250MB         100MB        60% ↓
Conexões simultâneas     100           500          5x


EXEMPLO DE ARQUITETURA:
────────────────────────────────────────────────────────────────

ARQUITETURA HÍBRIDA (Recomendado):
───────────────────────────────────────────────────────────────

                    ┌──────────────┐
                    │   Browser    │
                    │   Mobile     │
                    └──────┬───────┘
                           │
                      REST/JSON
                      HTTP/1.1
                           │
                    ┌──────▼───────┐
                    │  API Gateway │
                    │  (gRPC-Web)  │
                    └──────┬───────┘
                           │
                      gRPC/Proto
                      HTTP/2
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐      ┌────▼────┐      ┌────▼────┐
    │ Service │◄────►│ Service │◄────►│ Service │
    │    A    │ gRPC │    B    │ gRPC │    C    │
    └─────────┘      └─────────┘      └─────────┘

    Backend: gRPC (performance)
    Frontend: REST (compatibilidade)


TRANSIÇÃO DE REST PARA gRPC:
────────────────────────────────────────────────────────────────

REST Endpoint              →  gRPC Method
───────────────────────────   ──────────────────────────────

GET    /accounts/{id}      →  GetAccount(id)
POST   /accounts            →  CreateAccount(account)
PUT    /accounts/{id}       →  UpdateAccount(account)
DELETE /accounts/{id}       →  DeleteAccount(id)
GET    /accounts            →  ListAccounts() (stream)
```

### Resumo Executivo

| **REST** | **gRPC** |
|----------|----------|
| Maduro e amplamente adotado | Mais recente, crescendo rapidamente |
| Ideal para APIs públicas | Ideal para microserviços |
| Suporte universal (browsers) | Requer cliente específico |
| Baseado em recursos (CRUD) | Baseado em ações (RPC) |
| Debugging simples | Requer ferramentas especiais |
| Payload maior (JSON) | Payload menor (Protobuf) |
| Uma req/res por vez | Streaming bidirecional |
| HTTP/1.1 padrão | HTTP/2 obrigatório |

**Conclusão:** Use REST para APIs públicas e interfaces com navegadores. Use gRPC para comunicação entre serviços backend onde performance é crítica.

## Pré requisitos

### ProtoC

-   [Manual de instalação](https://grpc.io/docs/protoc-installation/)

```bash
# Instalação
apt install -y protobuf-compiler
# Versão
protoc --version
```

### Pacotes Go

-   [Manual de instalação](https://grpc.io/docs/languages/go/quickstart/)

```bash
# Generator para go
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
# Generate grpc para go
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```

### Sqlite3

```bash
# Instalação
sudo apt install sqlite3
# Versão
sqlite3 --version
```

### Evans

Evans é um cliente CLI interativo para testar serviços gRPC, similar ao Postman para REST APIs.

-   [Manual de instalação](https://github.com/ktr0731/evans)

```bash
# Instalação
go install github.com/ktr0731/evans@latest
```

**Características do Evans:**
- Interface REPL interativa para gRPC
- Suporte a reflection (descobre serviços automaticamente)
- Autocomplete para comandos e serviços
- Suporte a streaming (unary, server, client e bidirectional)
- Formato JSON e Protobuf
- Ideal para desenvolvimento e testes

## Recomendações de plugins para VsCode

### vscode-proto3

Plugin essencial para trabalhar com arquivos `.proto` no VSCode.

**Funcionalidades:**
- Syntax highlighting para Protocol Buffers
- Autocompletar para mensagens e serviços
- Validação de sintaxe em tempo real
- Navegação entre definições (Go to Definition)
- Formatação automática de código
- Snippets para estruturas comuns

**Instalação:**
1. Abra VSCode
2. Acesse Extensions (Ctrl+Shift+X)
3. Procure por "vscode-proto3"
4. Clique em "Install"

## Comandos

### Rodar o programa

```bash
go run cmd/grpc_server/main.go
```

### Sqlite3

```bash
# Acessa o banco
sqlite3 db.sqlite
# Cria tabela
sqlite> create table accounts (id string PRIMARY KEY, name string, email string);
# Lista os dados da tabela
sqlite> select * from accounts;
# Para sair
sqlite> .quit
```

<details>
<summary>Mais comandos do Sqlite3</summary>

```bash
# Deleta todos os registros
sqlite> DELETE FROM accounts;
# Dropa a tabela
sqlite> DROP TABLE accounts;
# Insere um registro
sqlite> INSERT INTO accounts (id, name, email) VALUES ('xx0011', 'tiago', 'tiago@gmail.com');
```

</details>

### ProtolC

```bash
# Gera os arquivos e interfaces na pasta /internal/pb
protoc --go_out=. --go-grpc_out=. proto/account.proto
# Baixa os pacotes
go mod tidy
```

### Client Evans

```bash
# 1 - Acessa o client, utilizando reflection
evans -r repl
# 2 - Seleciona o package
> package pb
# 3 - Seleciona o serviço
> service AccountService
# 4 - Executa a chamada ao serviço
> call CreateAccount
```

**Atenção:** Para parar o envio de streams no Evans: ctrl + D

## Arquitetura e Boas Práticas

### Estrutura de Projeto Recomendada

```
projeto-grpc/
├── proto/                          # Arquivos .proto
│   ├── account.proto
│   ├── user.proto
│   └── common.proto
│
├── internal/                       # Código interno
│   ├── pb/                        # Código gerado pelo protoc
│   │   ├── account.pb.go
│   │   ├── account_grpc.pb.go
│   │   └── ...
│   │
│   ├── service/                   # Implementação dos serviços
│   │   ├── account_service.go
│   │   └── user_service.go
│   │
│   ├── repository/                # Camada de dados
│   │   └── account_repository.go
│   │
│   └── middleware/                # Interceptors
│       ├── auth.go
│       ├── logging.go
│       └── metrics.go
│
├── cmd/
│   ├── server/                    # Servidor gRPC
│   │   └── main.go
│   │
│   └── client/                    # Cliente gRPC
│       └── main.go
│
├── db/                            # Banco de dados
│   └── db.sqlite
│
├── go.mod
└── README.md
```

### Fluxo de Desenvolvimento gRPC

```
┌────────────────────────────────────────────────────────────────┐
│              CICLO DE DESENVOLVIMENTO gRPC                     │
└────────────────────────────────────────────────────────────────┘

1. DEFINIR CONTRATO
   ┌─────────────────────────────────┐
   │  Escrever arquivo .proto        │
   │  - Definir messages             │
   │  - Definir services             │
   │  - Definir RPCs                 │
   └────────────┬────────────────────┘
                │
                ▼
2. GERAR CÓDIGO
   ┌─────────────────────────────────┐
   │  protoc --go_out=. \            │
   │    --go-grpc_out=. \            │
   │    proto/*.proto                │
   └────────────┬────────────────────┘
                │
                ▼
3. IMPLEMENTAR SERVIDOR
   ┌─────────────────────────────────┐
   │  - Implementar interface        │
   │  - Adicionar lógica de negócio  │
   │  - Tratar erros                 │
   │  - Adicionar middleware         │
   └────────────┬────────────────────┘
                │
                ▼
4. IMPLEMENTAR CLIENTE
   ┌─────────────────────────────────┐
   │  - Criar conexão                │
   │  - Chamar métodos               │
   │  - Tratar respostas             │
   └────────────┬────────────────────┘
                │
                ▼
5. TESTAR
   ┌─────────────────────────────────┐
   │  - Unit tests                   │
   │  - Integration tests            │
   │  - Evans (manual testing)       │
   └────────────┬────────────────────┘
                │
                ▼
6. DEPLOY
   ┌─────────────────────────────────┐
   │  - Docker/Kubernetes            │
   │  - Service Mesh (Istio)         │
   │  - Load Balancing               │
   │  - Monitoring                   │
   └─────────────────────────────────┘
```

### Interceptors (Middleware) Pattern

```
┌────────────────────────────────────────────────────────────────┐
│                   gRPC INTERCEPTORS CHAIN                      │
└────────────────────────────────────────────────────────────────┘

Cliente                                              Servidor
────────                                             ─────────

  Request
    │
    │     ┌──────────────────────────────────────────┐
    ├────►│  1. Client Interceptor (Auth)           │
    │     │     - Adiciona token                    │
    │     └──────────────────────────────────────────┘
    │
    │     ┌──────────────────────────────────────────┐
    ├────►│  2. Client Interceptor (Logging)        │
    │     │     - Log da requisição                 │
    │     └──────────────────────────────────────────┘
    │
    ├──────────────────  REDE  ─────────────────────────►
    │
    │                     ┌────────────────────────────────┐
    │                     │  3. Server Interceptor (Auth)  │
    │                     │     - Valida token             │
    │                     └────────────────────────────────┘
    │                                      │
    │                     ┌────────────────▼───────────────┐
    │                     │  4. Server Interceptor (Logs)  │
    │                     │     - Log da requisição        │
    │                     └────────────────────────────────┘
    │                                      │
    │                     ┌────────────────▼───────────────┐
    │                     │  5. Server Interceptor (Metrics)│
    │                     │     - Coleta métricas          │
    │                     └────────────────────────────────┘
    │                                      │
    │                     ┌────────────────▼───────────────┐
    │                     │  6. Handler (Business Logic)   │
    │                     │     - Processa requisição      │
    │                     └────────────────┬───────────────┘
    │                                      │
    │                                   Response
    │                                      │
    │◄─────────────────  REDE  ──────────────────────────┤
    │
    │     ┌──────────────────────────────────────────┐
    │◄────┤  7. Client Interceptor (Response)        │
    │     │     - Processa resposta                  │
    │     └──────────────────────────────────────────┘
    │
  Response


TIPOS DE INTERCEPTORS:
──────────────────────────────────────────────────────────────────

Unary Interceptor:          Stream Interceptor:
- Uma req/res por vez       - Streams de dados
- Simples de implementar    - Mais complexo
- Uso: auth, logs, metrics  - Uso: conexões persistentes
```

### Tratamento de Erros em gRPC

```
┌────────────────────────────────────────────────────────────────┐
│                   gRPC STATUS CODES                            │
└────────────────────────────────────────────────────────────────┘

Código              │ HTTP    │ Uso
────────────────────┼─────────┼─────────────────────────────────
OK                  │ 200     │ Sucesso
CANCELLED           │ 499     │ Cliente cancelou
UNKNOWN             │ 500     │ Erro desconhecido
INVALID_ARGUMENT    │ 400     │ Parâmetros inválidos
DEADLINE_EXCEEDED   │ 504     │ Timeout
NOT_FOUND           │ 404     │ Recurso não encontrado
ALREADY_EXISTS      │ 409     │ Recurso já existe
PERMISSION_DENIED   │ 403     │ Sem permissão
UNAUTHENTICATED     │ 401     │ Não autenticado
RESOURCE_EXHAUSTED  │ 429     │ Rate limit / Quota
FAILED_PRECONDITION │ 400     │ Pré-condição falhou
ABORTED             │ 409     │ Conflito de concorrência
OUT_OF_RANGE        │ 400     │ Fora do range válido
UNIMPLEMENTED       │ 501     │ Não implementado
INTERNAL            │ 500     │ Erro interno
UNAVAILABLE         │ 503     │ Serviço indisponível
DATA_LOSS           │ 500     │ Perda de dados


EXEMPLO DE TRATAMENTO:
──────────────────────────────────────────────────────────────────

Servidor (Go):
─────────────
if account == nil {
    return nil, status.Error(
        codes.NotFound,
        "account not found",
    )
}

if err := validate(req); err != nil {
    return nil, status.Error(
        codes.InvalidArgument,
        fmt.Sprintf("invalid input: %v", err),
    )
}


Cliente (Go):
────────────
resp, err := client.GetAccount(ctx, req)
if err != nil {
    st, ok := status.FromError(err)
    if ok {
        switch st.Code() {
        case codes.NotFound:
            // Trata não encontrado
        case codes.InvalidArgument:
            // Trata argumento inválido
        default:
            // Trata outros erros
        }
    }
}
```

### Performance e Otimização

```
┌────────────────────────────────────────────────────────────────┐
│              BOAS PRÁTICAS DE PERFORMANCE                      │
└────────────────────────────────────────────────────────────────┘

1. CONNECTION POOLING
   ┌────────────────────────────────────────────────┐
   │  Reutilize conexões gRPC                       │
   │  - Uma conexão por target                      │
   │  - HTTP/2 multiplexing automático              │
   │  - Evite criar/fechar conexões repetidamente   │
   └────────────────────────────────────────────────┘

2. STREAMING PARA GRANDES VOLUMES
   ┌────────────────────────────────────────────────┐
   │  Use streaming ao invés de unary               │
   │  - Server streaming: listagens grandes         │
   │  - Client streaming: uploads                   │
   │  - Bidirecional: real-time                     │
   └────────────────────────────────────────────────┘

3. COMPRESSÃO
   ┌────────────────────────────────────────────────┐
   │  Habilite compressão para payloads grandes     │
   │  - gzip (padrão)                               │
   │  - Trade-off: CPU vs Rede                      │
   └────────────────────────────────────────────────┘

4. TIMEOUTS E DEADLINES
   ┌────────────────────────────────────────────────┐
   │  Sempre defina timeouts                        │
   │  - Context com deadline                        │
   │  - Previne requisições travadas                │
   │  - Libera recursos rapidamente                 │
   └────────────────────────────────────────────────┘

5. LOAD BALANCING
   ┌────────────────────────────────────────────────┐
   │  Client-side:                                  │
   │  - Round-robin                                 │
   │  - Pick-first                                  │
   │                                                │
   │  Server-side:                                  │
   │  - Nginx                                       │
   │  - Envoy                                       │
   │  - Istio                                       │
   └────────────────────────────────────────────────┘

6. MONITORING
   ┌────────────────────────────────────────────────┐
   │  Métricas importantes:                         │
   │  - Latência (p50, p95, p99)                    │
   │  - Taxa de erro                                │
   │  - Throughput (req/s)                          │
   │  - Conexões ativas                             │
   │                                                │
   │  Ferramentas:                                  │
   │  - Prometheus + Grafana                        │
   │  - OpenTelemetry                               │
   │  - Jaeger (tracing)                            │
   └────────────────────────────────────────────────┘
```

### Segurança em gRPC

```
┌────────────────────────────────────────────────────────────────┐
│                   SEGURANÇA gRPC                               │
└────────────────────────────────────────────────────────────────┘

1. TLS/SSL (Transport Security)
   ┌────────────────────────────────────────────────┐
   │  Cliente                      Servidor         │
   │    │                              │            │
   │    ├──── TLS Handshake ──────────►│            │
   │    │◄─── Certificado ─────────────┤            │
   │    │                              │            │
   │    ├──── Dados Criptografados ───►│            │
   │    │◄─── Dados Criptografados ────┤            │
   │                                                │
   │  ✓ Confidencialidade                          │
   │  ✓ Integridade                                │
   │  ✓ Autenticação do servidor                   │
   │  ✓ (Opcional) Autenticação mútua (mTLS)       │
   └────────────────────────────────────────────────┘

2. AUTENTICAÇÃO
   ┌────────────────────────────────────────────────┐
   │  Métodos:                                      │
   │  • JWT (JSON Web Tokens)                       │
   │  • OAuth 2.0                                   │
   │  • API Keys                                    │
   │  • mTLS (Mutual TLS)                           │
   │                                                │
   │  Implementação:                                │
   │  • Metadata/Headers                            │
   │  • Interceptors para validação                 │
   └────────────────────────────────────────────────┘

3. AUTORIZAÇÃO
   ┌────────────────────────────────────────────────┐
   │  Request                                       │
   │    │                                           │
   │    ├──► Auth Interceptor                       │
   │    │    ├─ Valida Token                        │
   │    │    ├─ Extrai Claims                       │
   │    │    └─ Verifica Permissões                 │
   │    │                                           │
   │    ├──► Handler (se autorizado)                │
   │    │                                           │
   │    └──► Error (se não autorizado)              │
   └────────────────────────────────────────────────┘

4. RATE LIMITING
   ┌────────────────────────────────────────────────┐
   │  Protege contra:                               │
   │  • DoS/DDoS                                    │
   │  • Abuse de API                                │
   │  • Custos excessivos                           │
   │                                                │
   │  Estratégias:                                  │
   │  • Token Bucket                                │
   │  • Leaky Bucket                                │
   │  • Fixed Window                                │
   │  • Sliding Window                              │
   └────────────────────────────────────────────────┘
```

### Microserviços com gRPC

```
┌────────────────────────────────────────────────────────────────┐
│              ARQUITETURA DE MICROSERVIÇOS                      │
└────────────────────────────────────────────────────────────────┘

                    ┌──────────────────────┐
                    │    API Gateway       │
                    │   (gRPC-Web/REST)    │
                    └──────────┬───────────┘
                               │
                   ┌───────────┼───────────┐
                   │           │           │
            ┌──────▼─────┐ ┌──▼─────┐ ┌──▼─────┐
            │  Service A │ │Service │ │Service │
            │  (Accounts)│ │   B    │ │   C    │
            └──────┬─────┘ └────────┘ └────────┘
                   │
          ┌────────┼────────┐
          │        │        │
     ┌────▼───┐ ┌─▼────┐ ┌─▼────┐
     │Database│ │Cache │ │Queue │
     └────────┘ └──────┘ └──────┘


COMUNICAÇÃO ENTRE SERVIÇOS:
────────────────────────────────────────────────────────────────

Síncrona (gRPC):              Assíncrona (Message Queue):
─────────────────             ─────────────────────────────

Service A ──gRPC──► Service B  Service A ──► Queue ──► Service B
    │                   │          │                        │
    └───── Response ────┘          └─── Fire & Forget ──────┘

Vantagens:                    Vantagens:
• Resposta imediata           • Desacoplamento
• Simples                     • Escalabilidade
• Contratos tipados           • Resilência

Desvantagens:                 Desvantagens:
• Acoplamento                 • Complexidade
• Cascata de falhas           • Eventual consistency


PADRÕES DE DESIGN:
────────────────────────────────────────────────────────────────

1. SERVICE MESH (Istio, Linkerd)
   • Observabilidade
   • Traffic management
   • Security (mTLS)
   • Resiliência (retry, timeout)

2. CIRCUIT BREAKER
   • Previne cascata de falhas
   • Fail fast
   • Fallback strategies

3. SAGA PATTERN
   • Transações distribuídas
   • Compensação de erros
   • Consistência eventual
```

## Recursos Adicionais

### Documentação Oficial
- **gRPC:** https://grpc.io/
- **Protocol Buffers:** https://protobuf.dev/
- **Go gRPC:** https://grpc.io/docs/languages/go/

### Ferramentas Úteis
- **Evans:** Cliente CLI para gRPC
- **grpcurl:** cURL para gRPC
- **Postman:** Suporte a gRPC (versão desktop)
- **Bloomrpc:** GUI client para gRPC
- **ghz:** Ferramenta de benchmark para gRPC

### Monitoramento e Observabilidade
- **Prometheus:** Métricas
- **Grafana:** Visualização
- **Jaeger:** Distributed tracing
- **OpenTelemetry:** Observabilidade completa

### Service Mesh
- **Istio:** Service mesh completo
- **Linkerd:** Service mesh leve
- **Envoy:** Proxy para gRPC

<hr />

<div>
  <sub>Made with 💙 by <a href="https://github.com/venzel">Enéas Almeida</a></sub>
</div>
