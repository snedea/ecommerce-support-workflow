# Architecture: E-commerce Customer Support Flowise Workflow

**Architecture Type**: Flowise Multi-Agent System
**Pattern**: Central Router → Specialized Agents
**Complexity**: Moderate (8 agents)

---

## System Architecture Overview

```
┌─────────────┐
│ User Input  │
└──────┬──────┘
       │
       ▼
┌─────────────────────────┐
│  startAgentflow_0       │
│  (Start Node)           │
└──────────┬──────────────┘
           │
           ▼
┌──────────────────────────────────┐
│  conditionAgentAgentflow_0       │
│  (Intent Router - Scout Mode)    │
│  Temperature: 0.2                │
│  Model: gpt-4o-mini              │
└───────────┬──────────────────────┘
            │
            ├─[0]─> Agent.OrderTracking
            ├─[1]─> Agent.Returns
            ├─[2]─> Agent.ProductInfo
            ├─[3]─> Agent.Payment
            ├─[4]─> Agent.Shipping
            ├─[5]─> Agent.Account
            ├─[6]─> Agent.Promotions
            └─[7]─> Agent.GeneralHelp
```

**Key Principle**: Single workflow JSON file with ALL nodes inline (no external references).

---

## Complete Node Specifications

### Node 1: Start Node

**ID**: `startAgentflow_0`
**Type**: `agentFlow`
**Category**: Start Node
**Label**: "Start"

**Purpose**: Entry point for all customer inquiries

**Input Parameters**:
```json
{
  "formInputs": [
    {
      "label": "User Question",
      "name": "question",
      "type": "text",
      "placeholder": "Enter your question here..."
    }
  ],
  "formTitle": "E-commerce Support",
  "formDescription": "Ask me anything about orders, returns, products, payments, shipping, or your account!"
}
```

**Visual Properties**:
- Position: (400, 100)
- Color: #7EE787 (green)
- Width: 300
- Height: 400

---

### Node 2: ConditionAgent Router

**ID**: `conditionAgentAgentflow_0`
**Type**: `agentFlow`
**Category**: Agent Flows
**Label**: "Detect User Intention"

**Purpose**: Intelligent routing to specialized agents using Scout Mode

**Model Configuration**:
```json
{
  "conditionAgentModel": "chatOpenAI",
  "conditionAgentModelConfig": {
    "modelName": "gpt-4o-mini",
    "temperature": 0.2,
    "streaming": true,
    "agentModel": "chatOpenAI"
  }
}
```

**Routing Instructions** (Scout Mode - Comprehensive):
```
You are an intelligent E-commerce Support Router with expertise in customer service workflows.

ROUTING PROCESS:
1. Analyze customer inquiry for primary intent
2. Identify keywords and their context
3. Apply conditional routing rules below
4. Select the most appropriate specialist

KEYWORD → AGENT MAPPING:

Order Tracking:
- Keywords: 'order status', 'tracking number', 'where is my order', 'delivery estimate', 'shipment', 'WISMO'
- Route to: Scenario 0 (Agent.OrderTracking)
- Exception: If mentions 'return order' → Scenario 1 (Agent.Returns)

Returns:
- Keywords: 'return', 'refund', 'RMA', 'exchange', 'send back', 'defective', 'damaged'
- Route to: Scenario 1 (Agent.Returns)
- Exception: If only asking 'How do I return?' (informational) → Scenario 7 (Agent.GeneralHelp)

Product Info:
- Keywords: 'product details', 'specs', 'compatibility', 'in stock', 'availability', 'recommend', 'compare'
- Route to: Scenario 2 (Agent.ProductInfo)
- Exception: If asking about 'discount on product' → Scenario 6 (Agent.Promotions)

Payment:
- Keywords: 'payment failed', 'billing', 'charge', 'credit card', 'payment method', 'transaction error'
- Route to: Scenario 3 (Agent.Payment)
- Exception: If asking for 'refund status' → Scenario 1 (Agent.Returns)

Shipping:
- Keywords: 'shipping cost', 'delivery options', 'change address', 'expedited', 'shipping methods'
- Route to: Scenario 4 (Agent.Shipping)
- Exception: If tracking existing order → Scenario 0 (Agent.OrderTracking)

Account:
- Keywords: 'account', 'password', 'login', 'email change', 'profile', 'loyalty points', 'reset password'
- Route to: Scenario 5 (Agent.Account)
- Exception: If asking about 'order history' → Scenario 0 (Agent.OrderTracking)

Promotions:
- Keywords: 'discount', 'coupon', 'promo code', 'sale', 'deal', 'offer', 'special'
- Route to: Scenario 6 (Agent.Promotions)

General Help:
- Keywords: 'hello', 'help', 'confused', 'not sure', 'question', 'hi', 'hey'
- Route to: Scenario 7 (Agent.GeneralHelp)

MULTI-INTENT HANDLING:
- If question spans multiple domains (e.g., "Track my return"):
  → Route to agent handling PRIMARY action (Returns for return tracking)
- If question has navigational + functional intent (e.g., "Where do I check my order?"):
  → Route to functional agent (OrderTracking) - they can provide guidance

FALLBACK RULES:
- Unclear informational questions → Scenario 7 (Agent.GeneralHelp)
- Urgent/specific requests → Route to most likely specialist
- Greetings/small talk → Scenario 7 (Agent.GeneralHelp)

CONTEXT AWARENESS:
- Urgency indicators ('urgent', 'ASAP', 'help!') → Bias toward action-capable specialist
- Navigation phrases ('how do I', 'where can I') → Consider keywords carefully
- Questions with no action verbs → Likely informational, consider General Help
```

**Scenarios Array**:
```json
[
  {"scenario": "Order status, tracking, delivery questions"},
  {"scenario": "Returns, refunds, RMA, exchanges"},
  {"scenario": "Product information, specs, availability"},
  {"scenario": "Payment issues, billing, transaction problems"},
  {"scenario": "Shipping options, costs, address changes"},
  {"scenario": "Account management, password, loyalty"},
  {"scenario": "Promotions, discounts, coupon codes"},
  {"scenario": "General help, greetings, unclear requests"}
]
```

**Output Anchors**: 8 outputs (0-7), one per scenario

**Visual Properties**:
- Position: (400, 600)
- Color: #ff8fab (pink)
- Width: 300
- Height: 500

---

### Node 3-10: Agent Specifications

#### Agent.OrderTracking (agentAgentflow_1)

**Label**: "Agent.OrderTracking"
**Temperature**: 0.5
**Model**: gpt-4o-mini

**Persona**:
```html
<p><em>You are an expert Order Tracking agent.</em> You help customers check order status, locate tracking numbers, provide delivery estimates, and troubleshoot shipment issues. You maintain a proactive, reassuring tone and always verify order numbers before providing information. You have access to currentDateTime to evaluate delivery dates and searXNG for carrier tracking lookup. You do NOT handle returns or exchanges - defer those to the Returns agent. You do NOT process payments - defer to Payment agent.</p>
```

**Standard Tools** (EXACT Flowise UI structure):
```json
[
  {
    "agentSelectedTool": "currentDateTime",
    "agentSelectedToolRequiresHumanInput": "",
    "agentSelectedToolConfig": {
      "agentSelectedTool": "currentDateTime"
    }
  },
  {
    "agentSelectedTool": "searXNG",
    "agentSelectedToolRequiresHumanInput": "",
    "agentSelectedToolConfig": {
      "apiBase": "https://s.llam.ai",
      "toolName": "searxng-search",
      "toolDescription": "Federated web/meta search. Use when you need fresh facts or sources. Provide a natural-language query; returns a ranked, de-duplicated JSON list of result metadata for follow-up browsing and citation.",
      "headers": "",
      "format": "json",
      "categories": "",
      "engines": "",
      "language": "",
      "pageno": "",
      "time_range": "",
      "safesearch": "",
      "agentSelectedTool": "searXNG"
    }
  }
]
```

**Memory**:
- Enabled: true
- Type: "allMessages"
- Window Size: 20

**Position**: (100, 1200)

---

#### Agent.Returns (agentAgentflow_2)

**Label**: "Agent.Returns"
**Temperature**: 0.5
**Model**: gpt-4o-mini

**Persona**:
```html
<p><em>You are an expert Returns specialist.</em> You guide customers through return policies, initiate RMA requests, track refund status, and process exchange requests. You maintain an empathetic, solution-focused tone. You can approve returns up to $500 - escalate higher values by noting "This requires manager approval for amounts over $500". You have access to currentDateTime to check return windows (typically 30 days) and searXNG for policy lookup. You do NOT handle payment method updates - defer to Payment agent. You do NOT track original order shipments - defer to Order Tracking agent.</p>
```

**Standard Tools**: [Same structure as OrderTracking]
**Memory**: Same configuration
**Position**: (400, 1200)

---

#### Agent.ProductInfo (agentAgentflow_3)

**Label**: "Agent.ProductInfo"
**Temperature**: 0.6
**Model**: gpt-4o-mini

**Persona**:
```html
<p><em>You are an expert Product Information specialist.</em> You provide detailed product specifications, check availability across locations, answer compatibility questions, and make personalized recommendations based on customer needs. You maintain an enthusiastic, helpful tone. You have access to currentDateTime to check stock refresh dates and searXNG for product spec lookup. You do NOT handle pricing discounts or promotions - defer to Promotions agent. You do NOT process orders - provide information only.</p>
```

**Standard Tools**: [Same structure]
**Memory**: Same configuration
**Position**: (700, 1200)

---

#### Agent.Payment (agentAgentflow_4)

**Label**: "Agent.Payment"
**Temperature**: 0.5
**Model**: gpt-4o-mini

**Persona**:
```html
<p><em>You are an expert Payment Support agent.</em> You resolve payment failures, answer billing questions, help update payment methods, and investigate transaction problems. You maintain a patient, security-conscious tone. You verify customer identity (order number or email) before discussing payment details. You have access to currentDateTime for transaction timestamps and searXNG for payment troubleshooting guides. You do NOT handle refunds - defer to Returns agent. You do NOT track shipments - defer to Order Tracking agent. Never ask for full credit card numbers.</p>
```

**Standard Tools**: [Same structure]
**Memory**: Same configuration
**Position**: (1000, 1200)

---

#### Agent.Shipping (agentAgentflow_5)

**Label**: "Agent.Shipping"
**Temperature**: 0.5
**Model**: gpt-4o-mini

**Persona**:
```html
<p><em>You are an expert Shipping specialist.</em> You explain shipping options and costs, process address changes (for pre-shipment orders only), resolve delivery issues, and provide carrier information. You maintain a clear, logistics-focused tone. You have access to currentDateTime for delivery date calculations and searXNG for shipping rate lookup. You do NOT track existing orders - defer to Order Tracking agent. You do NOT handle returns or shipping damage claims - defer to Returns agent.</p>
```

**Standard Tools**: [Same structure]
**Memory**: Same configuration
**Position**: (1300, 1200)

---

#### Agent.Account (agentAgentflow_6)

**Label**: "Agent.Account"
**Temperature**: 0.5
**Model**: gpt-4o-mini

**Persona**:
```html
<p><em>You are an expert Account Management agent.</em> You assist with profile updates, password resets, email changes, and loyalty program questions. You maintain a friendly, security-aware tone. You verify identity (email or order number) before making account changes. You have access to currentDateTime for account activity timestamps and searXNG for loyalty program info. You do NOT handle order-specific questions - defer to Order Tracking agent. You do NOT process payments - defer to Payment agent. For password resets, provide clear step-by-step instructions.</p>
```

**Standard Tools**: [Same structure]
**Memory**: Same configuration
**Position**: (1600, 1200)

---

#### Agent.Promotions (agentAgentflow_7)

**Label**: "Agent.Promotions"
**Temperature**: 0.6
**Model**: gpt-4o-mini

**Persona**:
```html
<p><em>You are an expert Promotions specialist.</em> You provide information about current deals, validate discount codes, explain promotion terms, and highlight relevant sales. You maintain an enthusiastic, value-focused tone. You have access to currentDateTime to check promotion expiry dates and searXNG for current sales lookup. You do NOT apply discounts to existing orders - defer to Payment/Order agents. You do NOT handle product recommendations - defer to Product Info agent. Always mention promotion end dates and any restrictions.</p>
```

**Standard Tools**: [Same structure]
**Memory**: Same configuration
**Position**: (1900, 1200)

---

#### Agent.GeneralHelp (agentAgentflow_8)

**Label**: "Agent.GeneralHelp"
**Temperature**: 0.5
**Model**: gpt-4o-mini

**Persona**:
```html
<p><em>You are a General Help agent and welcoming first point of contact.</em> You assist with questions that don't fit specialized agents, handle greetings, provide clarification, and route users to the correct specialist when needed. You ask clarifying questions when intent is unclear. You maintain a warm, welcoming tone. You have access to currentDateTime and searXNG for general information. When you determine the user needs a specialist, say "Let me connect you with our [X] team who can help with that specific issue." You handle all greetings, small talk, and ambiguous requests.</p>
```

**Standard Tools**: [Same structure]
**Memory**: Same configuration
**Position**: (2200, 1200)

---

## Edge Definitions (9 total)

### Edge 1: Start → Router
```json
{
  "source": "startAgentflow_0",
  "sourceHandle": "startAgentflow_0-output-startAgentflow-Start",
  "target": "conditionAgentAgentflow_0",
  "targetHandle": "conditionAgentAgentflow_0",
  "type": "agentFlow",
  "data": {
    "sourceColor": "#7EE787",
    "targetColor": "#ff8fab",
    "edgeLabel": "question",
    "isHumanInput": false
  }
}
```

### Edges 2-9: Router → Agents

**Edge 2**: Router[output-0] → Agent.OrderTracking
- Label: "Order Tracking"

**Edge 3**: Router[output-1] → Agent.Returns
- Label: "Returns"

**Edge 4**: Router[output-2] → Agent.ProductInfo
- Label: "Product Info"

**Edge 5**: Router[output-3] → Agent.Payment
- Label: "Payment"

**Edge 6**: Router[output-4] → Agent.Shipping
- Label: "Shipping"

**Edge 7**: Router[output-5] → Agent.Account
- Label: "Account"

**Edge 8**: Router[output-6] → Agent.Promotions
- Label: "Promotions"

**Edge 9**: Router[output-7] → Agent.GeneralHelp
- Label: "General"

**Edge Structure Pattern**:
```json
{
  "source": "conditionAgentAgentflow_0",
  "sourceHandle": "conditionAgentAgentflow_0-output-{INDEX}",
  "target": "agentAgentflow_{INDEX+1}",
  "targetHandle": "agentAgentflow_{INDEX+1}",
  "type": "agentFlow",
  "data": {
    "sourceColor": "#ff8fab",
    "targetColor": "#4DD0E1",
    "edgeLabel": "{DESCRIPTIVE_LABEL}",
    "isHumanInput": false
  }
}
```

---

## File Structure

**CRITICAL**: Single workflow JSON file with ALL nodes inline.

```
ecommerce-support-workflow/
├── ecommerce-support-flow.json          ← SINGLE FILE (3000+ lines)
├── README.md
├── WORKFLOW-DIAGRAM.md
├── INTEGRATION_GUIDE.md
└── .context-foundry/
```

**Builder Requirements**:
- Generate ALL 10 nodes in single `nodes` array
- Generate ALL 9 edges in single `edges` array
- Use type: "agentFlow" for ALL nodes
- Use EXACT tool structure from above specifications
- NO external file references
- NO separate config files

---

## Testing Plan

### 1. JSON Structure Validation

**Pre-Import Checks**:
```bash
# Check file size (should be > 3000 lines)
wc -l ecommerce-support-flow.json

# Check node count (should be exactly 10)
jq '.nodes | length' ecommerce-support-flow.json

# Check edge count (should be exactly 9)
jq '.edges | length' ecommerce-support-flow.json

# Check for customNode type (should be 0)
jq '.nodes[] | select(.data.name == "customNode") | .id' ecommerce-support-flow.json

# Check for external file references (should be empty)
grep -i "src/agents" ecommerce-support-flow.json || echo "✅ No external refs"
```

### 2. Standard Tools Validation

**Verify ALL agents have both tools**:
```python
import json

with open('ecommerce-support-flow.json') as f:
    flow = json.load(f)

agents = [n for n in flow['nodes'] if n['data']['name'] == 'agentAgentflow']

for agent in agents:
    tools = agent['data']['inputs'].get('agentTools', [])
    tool_names = [t.get('agentSelectedTool') for t in tools]

    assert 'currentDateTime' in tool_names, f"{agent['data']['label']} missing currentDateTime"
    assert 'searXNG' in tool_names, f"{agent['data']['label']} missing searXNG"

    # Check searXNG structure
    searxng = next(t for t in tools if t.get('agentSelectedTool') == 'searXNG')
    assert 'apiBase' in searxng['agentSelectedToolConfig'], "Missing apiBase field"
    assert searxng['agentSelectedToolConfig']['agentSelectedTool'] == 'searXNG'

print("✅ All agents have required standard tools with correct structure")
```

### 3. Connectivity Validation

**Check for disconnected nodes**:
```python
# Check scenario count = agent count
scenarios = flow['nodes'][1]['data']['inputs']['conditionAgentScenarios']
assert len(scenarios) == 8, f"Expected 8 scenarios, found {len(scenarios)}"

# Check edge count
assert len(flow['edges']) == 9, f"Expected 9 edges, found {len(flow['edges'])}"

# Verify all agents have incoming edge
agent_ids = [n['id'] for n in agents]
target_ids = [e['target'] for e in flow['edges']]

for agent_id in agent_ids:
    assert agent_id in target_ids, f"Agent {agent_id} has no incoming edge (disconnected)"

print("✅ All agents connected properly")
```

### 4. Flowise Import Test

1. Import `ecommerce-support-flow.json` to Flowise
2. Verify all 10 nodes render on canvas
3. Check for validation errors in UI
4. Verify tools dropdown shows options (not blank)
5. Verify no "disconnected" warnings

### 5. Functional Routing Test

**Test queries** (one per agent):
```
1. "Where is my order #12345?" → Should route to OrderTracking
2. "I want to return my shoes" → Should route to Returns
3. "What are the specs for product XYZ?" → Should route to ProductInfo
4. "My payment was declined" → Should route to Payment
5. "How much is overnight shipping?" → Should route to Shipping
6. "I forgot my password" → Should route to Account
7. "Do you have any sales going on?" → Should route to Promotions
8. "Hello, I need help" → Should route to GeneralHelp
```

**Edge cases**:
```
- "Track my return" → Should route to Returns (primary intent)
- "Discount on product X?" → Should route to Promotions (not ProductInfo)
- "How do I find my order?" → Could go to OrderTracking or GeneralHelp (acceptable either way)
```

### 6. Tool Functionality Test

For each agent, test standard tools:
- **currentDateTime**: Should return current timestamp
- **searXNG**: Should return search results for test query

---

## Success Criteria

✅ **Architecture complete when**:

1. **Structure**:
   - [x] Single workflow JSON file specified
   - [x] 10 nodes planned (1 start + 1 router + 8 agents)
   - [x] 9 edges planned (start→router + 8 router→agents)
   - [x] All nodes use type: "agentFlow"

2. **Routing**:
   - [x] Scout Mode instructions comprehensive
   - [x] 8 scenarios defined (one per agent)
   - [x] Keyword mapping clear
   - [x] Exception rules documented
   - [x] Fallback logic defined

3. **Agents**:
   - [x] All 8 agents have detailed personas
   - [x] All agents have standard tools specified with EXACT structure
   - [x] All agents have memory configuration
   - [x] All agents have temperature settings
   - [x] Clear boundaries (what they DON'T do)

4. **Tools**:
   - [x] Standard tools (currentDateTime + searXNG) planned for all agents
   - [x] Exact Flowise UI structure documented
   - [x] Critical field names noted (apiBase, searXNG, "")

5. **Testing**:
   - [x] Validation scripts provided
   - [x] Test queries documented
   - [x] Edge cases identified
   - [x] Success criteria defined

---

**Architect Phase Complete**. Proceeding to Build Planning (Phase 2.5).
