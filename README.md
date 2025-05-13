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




