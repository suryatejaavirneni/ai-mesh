# ai-mesh


## Diagram1:

```
flowchart TD
    classDef user fill:#FFEBEE,stroke:#C62828,color:#B71C1C,rounded
    classDef agent fill:#E1F5FE,stroke:#0288D1,color:#01579B,rounded
    classDef specialagent fill:#B3E5FC,stroke:#0288D1,color:#01579B,rounded
    classDef mcp fill:#E1BEE7,stroke:#8E24AA,color:#4A148C,rounded
    classDef danger fill:#FFCDD2,stroke:#C62828,color:#B71C1C,stroke-dasharray: 5 5
    
    User(["User"]):::user
    
    MainAgent["Main<br>Agent"]:::agent
    
    Specialist1["Weather<br>Agent"]:::specialagent
    Specialist2["Restaurant<br>Agent"]:::specialagent
    Specialist3["Hotel<br>Agent"]:::specialagent
    
    MCP1["Weather<br>MCP Server"]:::mcp
    MCP2["Restaurant<br>MCP Server"]:::mcp
    MCP3["Hotel<br>MCP Server"]:::mcp
    
    EmbeddedCreds["⚠️ SECURITY ISSUES:<br>• Embedded API keys<br>• Broad access scopes<br>• No agent identity chain<br>• No context preservation"]:::danger
    
    User --"1. Request for<br>travel plan"--> MainAgent
    
    MainAgent --"2. Creates<br>specialists"--> Specialist1 & Specialist2 & Specialist3
    
    MainAgent --"3. Passes API keys<br>& OAuth tokens"--> EmbeddedCreds
    
    EmbeddedCreds --"4a. Embedded<br>credentials"--> Specialist1
    EmbeddedCreds --"4b. Embedded<br>credentials"--> Specialist2
    EmbeddedCreds --"4c. Embedded<br>credentials"--> Specialist3
    
    Specialist1 --"5a. Direct MCP call<br>with API key"--> MCP1
    Specialist2 --"5b. Direct MCP call<br>with API key"--> MCP2
    Specialist3 --"5c. Direct MCP call<br>with API key"--> MCP3
    
    MCP1 --"6a. Results"--> Specialist1
    MCP2 --"6b. Results"--> Specialist2
    MCP3 --"6c. Results"--> Specialist3
    
    Specialist1 & Specialist2 & Specialist3 --"7. Results"--> MainAgent
    
    MainAgent --"8. Complete<br>response"--> User
    
    Title["1. Current Multi-Agent Flow in A2A/MCP"]
    
    style Title fill:#F5F5F5,stroke:#9E9E9E,color:#212121,stroke-width:2px,font-weight:bold
```

<img width="1146" alt="image" src="https://github.com/user-attachments/assets/ea5478af-3ec1-482a-a7a2-bbfb0942d127" />

## Diagram2: 

```
flowchart TD
    classDef user fill:#FFEBEE,stroke:#C62828,color:#B71C1C,rounded
    classDef agent fill:#E1F5FE,stroke:#0288D1,color:#01579B,rounded
    classDef specialagent fill:#B3E5FC,stroke:#0288D1,color:#01579B,rounded
    classDef mcp fill:#E1BEE7,stroke:#8E24AA,color:#4A148C,rounded
    classDef danger fill:#FFCDD2,stroke:#C62828,color:#B71C1C,stroke-dasharray: 5 5
    classDef oauth fill:#C8E6C9,stroke:#4CAF50,color:#2E7D32,rounded
    
    User(["User"]):::user
    
    MainAgent["Main<br>Agent"]:::agent
    
    Weather["Weather<br>Agent"]:::specialagent
    Restaurant["Restaurant<br>Agent"]:::specialagent
    
    OAuth["OAuth<br>Server"]:::oauth
    
    BroadScope["Overly Broad Scope:<br>'access_all_tools'"]:::danger
    
    WeatherMCP["Weather<br>MCP Server"]:::mcp
    RestaurantMCP["Restaurant<br>MCP Server"]:::mcp
    PaymentMCP["Payment<br>MCP Server"]:::mcp
    
    Hallucination["⚠️ HALLUCINATION:<br>Agent believes it should<br>make a payment"]:::danger
    
    User --"1. Request for<br>travel plan"--> MainAgent
    
    MainAgent --"2. Request authorization<br>with broad scope"--> OAuth
    
    OAuth --"3. Issues token with<br>broad scope"--> BroadScope
    
    MainAgent --"4. Creates<br>specialists"--> Weather & Restaurant
    
    MainAgent --"5. Passes broad<br>scope token"--> Weather & Restaurant
    
    Weather --"6a. Weather<br>MCP call"--> WeatherMCP
    
    Restaurant --"6b. Has access to<br>ALL tools due to<br>broad scope"--> Hallucination
    
    Hallucination --"6c. Unauthorized<br>payment call"--> PaymentMCP
    
    PaymentMCP --"7. Processes<br>payment request"--> Restaurant
    
    Weather & Restaurant --"8. Results (including<br>unintended payment)"--> MainAgent
    
    MainAgent --"9. Response with<br>unauthorized actions"--> User
    
    Title["2. Hallucination Security Risk with Broad OAuth Scopes"]
    
    style Title fill:#F5F5F5,stroke:#9E9E9E,color:#212121,stroke-width:2px,font-weight:bold
```
<img width="1209" alt="image" src="https://github.com/user-attachments/assets/a8182b8d-d0e1-4c6f-a012-d740e0cd9295" />

## Diagram3:

```
flowchart TD
    classDef rar fill:#E8F5E9,stroke:#388E3C,color:#1B5E20,rounded
    classDef spiffe fill:#D1C4E9,stroke:#673AB7,color:#311B92,rounded
    classDef txn fill:#E3F2FD,stroke:#1565C0,color:#0D47A1,rounded
    
    subgraph RARConstruct["Rich Authorization Request (RAR)"]
        RAR["RAR Structure"]:::rar
        
        RARJson["authorization_details: [<br>  {<br>    type: 'restaurant-mcp',<br>    locations: [{country: 'Japan'}],<br>    actions: ['search'],<br>    datatypes: ['restaurants'],<br>    output_constraints: {<br>      no_shellfish: true<br>    }<br>  }<br>]"]
        
        RAR --> RARJson
    end
    
    subgraph SPIFFEConstruct["SPIFFE Workload Identity"]
        SPIFFE["SPIFFE SVID"]:::spiffe
        
        SPIFFEJson["sub: 'spiffe://trust-domain/ns/agent-namespace/sa/food-expert'<br>exp: 1621459200<br>iat: 1621455600"]
        
        SPIFFE --> SPIFFEJson
    end
    
    subgraph TxnConstruct["Transaction Token"]
        TXN["Transaction Token"]:::txn
        
        TXNJson["txn_id: 'unique-transaction-id',<br>rctx: {<br>  user_id: 'user-123',<br>  session_id: 'session-456'<br>},<br>tctx: {<br>  agent_chain: ['travel-assistant', 'food-expert']<br>},<br>authorization_details: [ ... RAR contents ... ]"]
        
        TXN --> TXNJson
    end
    
    RARConstruct -.-> TxnConstruct
    SPIFFEConstruct -.-> TxnConstruct
    
    RARTitle["Fine-Grained Permissions"]
    SPIFFETitle["Secure Workload Identity"]
    TXNTitle["Combined Security Context"]
    
    RARTitle -.-> RARConstruct
    SPIFFETitle -.-> SPIFFEConstruct
    TXNTitle -.-> TxnConstruct
    
    style RARTitle fill:#E8F5E9,stroke:#388E3C,color:#1B5E20,font-weight:bold
    style SPIFFETitle fill:#D1C4E9,stroke:#673AB7,color:#311B92,font-weight:bold
    style TXNTitle fill:#E3F2FD,stroke:#1565C0,color:#0D47A1,font-weight:bold
    
    Title["3. Key Security Constructs: RAR, SPIFFE and Transaction Token"]
    
    style Title fill:#F5F5F5,stroke:#9E9E9E,color:#212121,stroke-width:2px,font-weight:bold
```


<img width="1211" alt="image" src="https://github.com/user-attachments/assets/750d5725-2a52-4b59-a6ab-25612f1b7876" />

## Diagram4:

```
flowchart TD
    classDef user fill:#FFEBEE,stroke:#C62828,color:#B71C1C,rounded
    classDef agent fill:#E1F5FE,stroke:#0288D1,color:#01579B,rounded
    classDef oauth fill:#C8E6C9,stroke:#4CAF50,color:#2E7D32,rounded
    classDef rar fill:#E8F5E9,stroke:#388E3C,color:#1B5E20,rounded
    classDef svid fill:#D1C4E9,stroke:#673AB7,color:#311B92,rounded
    classDef txn fill:#E3F2FD,stroke:#1565C0,color:#0D47A1,rounded
    classDef broker fill:#FFF3E0,stroke:#E65100,color:#E65100,rounded
    classDef mcp fill:#E1BEE7,stroke:#8E24AA,color:#4A148C,rounded
    
    User(["User"]):::user
    Agent["AI<br>Agent"]:::agent
    
    OAuthServer["OAuth<br>Server"]:::oauth
    
    TxnService["Transaction Token<br>Service"]:::txn
    
    Broker["Identity<br>Broker"]:::broker
    
    MCP["MCP<br>Server"]:::mcp
    
    User --"1. User<br>Request"--> Agent
    
    Agent --"2. RAR Authorization Request<br>with authorization_details"--> OAuthServer
    
    OAuthServer --"3. RAR-Scoped Token"--> RARToken["RAR Token"]:::rar
    
    Agent --"4. Get Workload Identity"--> SVID["SPIFFE<br>SVID"]:::svid
    
    Agent --"5. Exchange RAR +<br>Workload Identity"--> TxnService
    
    RARToken --> TxnService
    SVID --> TxnService
    
    TxnService --"6. Transaction Token<br>with combined context"--> TxnToken["Transaction<br>Token"]:::txn
    
    Agent --"7. MCP Request +<br>Transaction Token"--> Broker
    
    Broker --"8. Validate Token,<br>Enforce Policy,<br>Inject Credentials"--> MCP
    
    MCP --"9. Results"--> Agent
    
    Agent --"10. Response"--> User
    
    style TxnToken fill:#E3F2FD,stroke:#1565C0,color:#0D47A1
    
    Title["4. Simple Agent Flow with Security Constructs"]
    
    style Title fill:#F5F5F5,stroke:#9E9E9E,color:#212121,stroke-width:2px,font-weight:bold
```

<img width="1039" alt="image" src="https://github.com/user-attachments/assets/be21ee87-e03b-452e-97b4-c94fd0bf4a5b" />


## Diagram5:

```
flowchart TB
    classDef user fill:#FFEBEE,stroke:#C62828,color:#B71C1C,rounded
    classDef agent fill:#E1F5FE,stroke:#0288D1,color:#01579B,rounded
    classDef specialagent fill:#B3E5FC,stroke:#0288D1,color:#01579B,rounded
    classDef mcp fill:#E1BEE7,stroke:#8E24AA,color:#4A148C,rounded
    classDef oauth fill:#C8E6C9,stroke:#4CAF50,color:#2E7D32,rounded
    classDef key fill:#FFECB3,stroke:#FFA000,color:#FF6F00,rounded
    classDef problem fill:#FFCDD2,stroke:#C62828,color:#B71C1C,rounded
    
    User(["<b>USER</b>"]):::user
    TravelAgent["Travel<br>Agent"]:::agent
    OAuth["OAuth<br>Server"]:::oauth
    Credentials["Broad Credentials:<br>Generic OAuth token + All API Keys"]:::key
    
    subgraph Specialists["Specialist Agents"]
        direction LR
        SpecialistAgents["Flight | Hotel | Food"]:::specialagent
    end
    
    subgraph MCP["MCP Servers"]
        direction LR
        MCPServers["Flight | Hotel | Restaurant"]:::mcp
    end
    
    subgraph Issues["Security Issues"]
        direction TB
        SecurityIssues["⚠️ 1: Embedded credentials in each agent<br>⚠️ 2: No workload identity verification<br>⚠️ 3: No transaction tokens for constraints<br>⚠️ 4: Context lost between agents"]:::problem
    end
    
    User --"1. Request: Japan trip,<br>budget & dietary constraints"--> TravelAgent
    TravelAgent --"2. Request<br>authorization"--> OAuth
    OAuth --"3. Generic scope token<br>(no fine-grained control)"--> Credentials
    TravelAgent --"4. Creates<br>specialists"--> Specialists
    Credentials --"5. Copy of all credentials<br>(no workload identity)"--> Specialists
    
    Specialists --"6. API calls with embedded keys<br>(no transaction tokens)"--> MCP
    MCP --"7. Results<br>(no constraint enforcement)"--> Specialists
    Specialists --"8. Return<br>results"--> TravelAgent
    TravelAgent --"9. Travel plan<br>(constraints only if<br>agent remembers)"--> User
    
    Credentials -.-> Issues
    
    style User font-size:18px
```
<img width="1127" alt="image" src="https://github.com/user-attachments/assets/8162093c-27f8-4653-a12f-6ca361c44a65" />


## Diagram6:

```
flowchart TB
    classDef user fill:#FFEBEE,stroke:#C62828,color:#B71C1C,rounded
    classDef agent fill:#E1F5FE,stroke:#0288D1,color:#01579B,rounded
    classDef specialagent fill:#B3E5FC,stroke:#0288D1,color:#01579B,rounded
    classDef mcp fill:#E1BEE7,stroke:#8E24AA,color:#4A148C,rounded
    classDef oauth fill:#C8E6C9,stroke:#4CAF50,color:#2E7D32,rounded
    classDef rar fill:#E8F5E9,stroke:#388E3C,color:#1B5E20,rounded
    classDef txn fill:#E3F2FD,stroke:#1565C0,color:#0D47A1,rounded
    classDef svid fill:#D1C4E9,stroke:#673AB7,color:#311B92,rounded
    classDef gateway fill:#FFF3E0,stroke:#E65100,color:#E65100,rounded
    classDef solution fill:#C8E6C9,stroke:#2E7D32,color:#1B5E20,rounded
    
    User(["<b>USER</b>"]):::user
    TravelAgent["Travel<br>Agent"]:::agent
    
    subgraph Security["Security Services"]
        direction LR
        RARService["RAR<br>Service"]:::oauth
        SPIFFEService["SPIFFE<br>Service"]:::svid
        TxnService["Transaction<br>Token Service"]:::txn
    end
    
    subgraph Tokens["Fine-Grained Authorization"]
        direction TB
        RARToken["Resource Access Rights (RAR):<br>{flight: under $1000, hotel: under $200,<br>food: no shellfish}"]:::rar
        SVID["SPIFFE Verifiable Identity<br>(workload identity)"]:::svid
        TransactionToken["Transaction Tokens:<br>Combined RAR constraints + SVID identity<br>for secure, scoped access"]:::txn
    end
    
    subgraph Specialists["Specialist Agents"]
        direction LR
        SpecialistAgents["Flight | Hotel | Food"]:::specialagent
    end
    
    Gateway["MCP<br>Gateway"]:::gateway
    
    subgraph MCP["MCP Servers"]
        direction LR
        MCPServers["Flight | Hotel | Restaurant"]:::mcp
    end
    
    subgraph Solutions["Security Solutions"]
        direction TB
        SecuritySolutions["✅ 1: No embedded credentials in agents<br>✅ 2: SPIFFE workload identity verification<br>✅ 3: Transaction tokens enforce constraints<br>✅ 4: Context preserved across agents"]:::solution
    end
    
    User --"1. Request: Japan trip,<br>budget & dietary constraints"--> TravelAgent
    TravelAgent --"2. Request fine-grained<br>authorization"--> RARService
    
    RARService --"3. Issues RAR with<br>specific constraints"--> RARToken
    TravelAgent --"4. Creates<br>specialists"--> Specialists
    
    Specialists --"5. Request workload<br>identity"--> SPIFFEService
    SPIFFEService --"6. Issues SVID<br>for each agent"--> SVID
    
    RARToken --"7a. Constraints<br>input"--> TxnService
    SVID --"7b. Identity<br>input"--> TxnService
    
    TxnService --"8. Issues tokens combining<br>identity and constraints"--> TransactionToken
    
    TransactionToken --"9. Secure, scoped<br>access rights"--> Specialists
    
    Specialists --"10. MCP request with<br>transaction token<br>(SVID+RAR combined)"--> Gateway
    
    Gateway --"11. Validates constraints,<br>verifies workload identity,<br>enforces limitations"--> MCP
    
    MCP --"12. Constraint-compliant<br>results (under budget,<br>no shellfish)"--> Gateway
    
    Gateway --"13. Verified<br>results"--> Specialists
    Specialists --"14. Secure<br>results"--> TravelAgent
    
    TravelAgent --"15. Safe travel plan<br>(all constraints guaranteed)"--> User
    
    TransactionToken -.-> Solutions
    
    style User font-size:18px
```

<img width="1007" alt="image" src="https://github.com/user-attachments/assets/00a416f4-81d5-49c2-88d3-b101ea57b666" />


## diagram7:

```
flowchart TB
    classDef user fill:#FFEBEE,stroke:#C62828,color:#B71C1C,rounded
    classDef agent fill:#E1F5FE,stroke:#0288D1,color:#01579B,rounded
    classDef specialagent fill:#B3E5FC,stroke:#0288D1,color:#01579B,rounded
    classDef mcp fill:#E1BEE7,stroke:#8E24AA,color:#4A148C,rounded
    classDef security fill:#C8E6C9,stroke:#4CAF50,color:#2E7D32,rounded
    classDef token fill:#E3F2FD,stroke:#1565C0,color:#0D47A1,rounded
    classDef gateway fill:#FFF3E0,stroke:#E65100,color:#E65100,rounded
    classDef solution fill:#C8E6C9,stroke:#2E7D32,color:#1B5E20,rounded
    classDef policy fill:#FFE0B2,stroke:#FF9800,color:#E65100,rounded
    
    User(["<b>USER</b>"]):::user
    TravelAgent["Travel<br>Agent"]:::agent
    
    SecurityServices["Security Services:<br>RAR (RFC 9521) + SPIFFE"]:::security
    
    TransactionToken["Transaction Token (RFC 9068):<br>Constraints + Workload Identity"]:::token
    
    subgraph Specialists["Specialist Agents"]
        direction LR
        SpecialistAgents["Flight | Hotel | Food"]:::specialagent
    end
    
    Gateway["MCP<br>Gateway"]:::gateway
    
    PolicyEngine["OPA Policy<br>Engine"]:::policy
    
    subgraph MCP["MCP Servers"]
        direction LR
        MCPServers["Flight | Hotel | Restaurant"]:::mcp
    end
    
    subgraph Solutions["Security Solutions"]
        direction TB
        SecuritySolutions["✅ 1: No embedded credentials<br>✅ 2: SPIFFE workload identity<br>✅ 3: Enforced constraints via Transaction Tokens<br>✅ 4: Preserved context across services"]:::solution
    end
    
    User --"1. Request with<br>constraints"--> TravelAgent
    TravelAgent --"2. Request<br>authorization"--> SecurityServices
    SecurityServices --"3. Issues Transaction Tokens<br>with constraints + identity"--> TransactionToken
    TravelAgent --"4. Creates<br>specialists"--> Specialists
    
    TransactionToken --"5. Secure, scoped<br>access rights"--> Specialists
    
    Specialists --"6. Requests with<br>Transaction Tokens"--> Gateway
    
    Gateway --"7. Policy check"--> PolicyEngine
    PolicyEngine --"Policy-based access evaluation:<br>RAR + Workload Identity + Agent Hops"--> Gateway
    
    Gateway --"8. Compliant<br>access"--> MCP
    
    MCP --"9. Compliant<br>results"--> Gateway
    
    Gateway --"10. Verified<br>results"--> Specialists
    Specialists --"11. Secure<br>results"--> TravelAgent
    
    TravelAgent --"12. Safe travel plan<br>(policy-enforced constraints)"--> User
    
    TransactionToken -.-> Solutions
    
    style User font-size:18px
```

<img width="1138" alt="image" src="https://github.com/user-attachments/assets/b0ebe38b-2dbd-4e05-a8db-1745b58afa4b" />


## diagram8:

```
flowchart TB
    classDef user fill:#FFEBEE,stroke:#C62828,color:#B71C1C,rounded
    classDef agent fill:#E1F5FE,stroke:#0288D1,color:#01579B,rounded
    classDef oauth fill:#C8E6C9,stroke:#4CAF50,color:#2E7D32,rounded
    classDef rar fill:#E8F5E9,stroke:#388E3C,color:#1B5E20,rounded
    classDef specialagent fill:#B3E5FC,stroke:#0288D1,color:#01579B,rounded
    classDef mcp fill:#E1BEE7,stroke:#8E24AA,color:#4A148C,rounded
    
    User(["USER"]):::user
    TravelAgent["Travel<br>Agent"]:::agent
    
    OAuth["OAuth<br>Service"]:::oauth
    
    subgraph RARExamples["Rich Authorization Requests (RAR)"]
        direction TB
        MainRAR["Travel RAR Token<br>{<br>  type: 'travel_planning',<br>  locations: ['Japan'],<br>  constraints: {<br>    flight_budget: '$1000',<br>    hotel_budget: '$200',<br>    dietary: 'no_shellfish'<br>  }<br>}"]:::rar
        
        FlightRAR["Flight RAR Token<br>{<br>  type: 'flight_only',<br>  locations: ['Japan'],<br>  constraints: {<br>    budget: '$1000'<br>  }<br>}"]:::rar
        
        HotelRAR["Hotel RAR Token<br>{<br>  type: 'hotel_only',<br>  locations: ['Japan'],<br>  constraints: {<br>    budget: '$200'<br>  }<br>}"]:::rar
        
        FoodRAR["Food RAR Token<br>{<br>  type: 'food_only',<br>  locations: ['Japan'],<br>  constraints: {<br>    dietary: 'no_shellfish'<br>  }<br>}"]:::rar
    end
    
    subgraph Specialists["Specialist Agents"]
        direction LR
        FlightAgent["Flight<br>Agent"]:::specialagent
        HotelAgent["Hotel<br>Agent"]:::specialagent
        FoodAgent["Food<br>Agent"]:::specialagent
    end
    
    User --"1. Request: Japan trip,<br>flight under $1000,<br>hotel under $200,<br>no shellfish"--> TravelAgent
    
    TravelAgent --"2. Request RAR<br>with constraints"--> OAuth
    OAuth --"3. Issues parent RAR<br>with all constraints"--> MainRAR
    
    MainRAR --"4. Derives child RAR<br>with flight constraints"--> FlightRAR
    MainRAR --"5. Derives child RAR<br>with hotel constraints"--> HotelRAR
    MainRAR --"6. Derives child RAR<br>with food constraints"--> FoodRAR
    
    FlightRAR --"7. Scoped authorization<br>with constraints"--> FlightAgent
    HotelRAR --"8. Scoped authorization<br>with constraints"--> HotelAgent
    FoodRAR --"9. Scoped authorization<br>with constraints"--> FoodAgent
```

<img width="821" alt="image" src="https://github.com/user-attachments/assets/4d7076d3-0d0d-4b92-9eae-0a6e20dcae89" />


## diagram 9

```
flowchart TB
    classDef user fill:#FFEBEE,stroke:#C62828,color:#B71C1C,rounded
    classDef agent fill:#E1F5FE,stroke:#0288D1,color:#01579B,rounded
    classDef specialagent fill:#B3E5FC,stroke:#0288D1,color:#01579B,rounded
    classDef rar fill:#E8F5E9,stroke:#388E3C,color:#1B5E20,rounded
    classDef svid fill:#D1C4E9,stroke:#673AB7,color:#311B92,rounded
    classDef txn fill:#E3F2FD,stroke:#1565C0,color:#0D47A1,rounded
    classDef gateway fill:#FFF3E0,stroke:#E65100,color:#E65100,rounded
    classDef mcp fill:#E1BEE7,stroke:#8E24AA,color:#4A148C,rounded
    classDef policy fill:#FFE0B2,stroke:#FF9800,color:#E65100,rounded
    
    User(["USER"]):::user
    TravelAgent["Travel<br>Agent"]:::agent
    
    subgraph TokenCreation["Transaction Token Creation"]
        direction TB
        FlightRAR["Flight RAR Token<br>{type: 'flight_only',<br>budget: '$1000'}"]:::rar
        
        SVID["SPIFFE Workload Identity<br>{<br>  svid: 'spiffe://example/agent/flight',<br>  parent: 'spiffe://example/agent/travel'<br>}"]:::svid
        
        TxnToken["Transaction Token (RFC 9068)<br>{<br>  sub: 'spiffe://example/agent/flight',<br>  authorization_details: {<br>    type: 'flight_only',<br>    constraints: {<br>      budget: '$1000'<br>    }<br>  },<br>  agent_chain: [<br>    'spiffe://example/agent/travel'<br>  ]<br>}"]:::txn
    end
    
    FlightAgent["Flight<br>Agent"]:::specialagent
    
    Gateway["MCP<br>Gateway"]:::gateway
    PolicyEngine["OPA Policy<br>Engine"]:::policy
    FlightMCP["Flight<br>MCP Server"]:::mcp
    
    User --"1. Request with constraints"--> TravelAgent
    TravelAgent --"2. Initiates token<br>creation process"--> TokenCreation
    
    FlightRAR --"3. RAR provides<br>constraints"--> TxnToken
    SVID --"4. SPIFFE provides<br>identity"--> TxnToken
    
    TravelAgent --"5. Creates<br>specialist"--> FlightAgent
    TxnToken --"6. Transaction token with<br>identity + constraints"--> FlightAgent
    
    FlightAgent --"7. Request with<br>transaction token"--> Gateway
    Gateway --"8. Validate<br>token"--> PolicyEngine
    
    PolicyEngine --"9. Verify identity<br>& constraints"--> Gateway
    
    Gateway --"10. Compliant<br>access"--> FlightMCP
    FlightMCP --"11. Results<br>(under budget)"--> Gateway
    Gateway --"12. Verified<br>results"--> FlightAgent
    FlightAgent --"13. Results"--> TravelAgent
    TravelAgent --"14. Safe travel plan"--> User
```

<img width="1039" alt="image" src="https://github.com/user-attachments/assets/f4c85867-355a-4f9f-b1e0-4910c29cfd34" />

