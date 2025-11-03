# Scout Report: E-commerce Customer Support Flowise Workflow

**Project Type**: Flowise Multi-Agent Workflow
**Complexity**: Moderate
**Agent Count**: 8 specialized agents
**Flow Type**: Multi-agent with intelligent routing

---

## Executive Summary

Building a comprehensive Flowise multi-agent workflow for e-commerce customer support with 8 specialized agents handling distinct domains: Order Tracking, Returns, Product Info, Payment, Shipping, Account, Promotions, and General Help. The workflow uses a central ConditionAgent router with Scout Mode for intelligent intent detection across all 8 agents.

**Key Decision**: This is a **Flowise-only project** - NOT a traditional web application. We will generate ONE complete workflow JSON file with all agents inline, following AGENT_PATTERN_REFERENCE.md patterns exactly.

---

## Flowise-Specific Requirements

### Mandatory Pattern Compliance

✅ **MUST follow**:
- Generate ONE complete JSON file (e.g., `ecommerce-support-flow.json`)
- ALL nodes inline in single `nodes` array
- NO separate config files (Pattern #3 violation prevention)
- Use type: "agentFlow" for all nodes (NOT "customNode")
- Self-contained agents (no separate model/memory nodes)
- Standard tools (currentDateTime + searXNG) with EXACT Flowise UI structure
- Scout Mode routing instructions (8+ agents requires comprehensive routing)

❌ **MUST NOT**:
- Create multiple JSON files or split nodes across files
- Use external agent config references
- Invent phantom tool names (Pattern #5)
- Use incorrect tool JSON structure (Pattern #6)
- Create disconnected agents (Pattern #4)
- Generate meta-description instead of complete flow (Pattern #1)

---

## Agent Specifications

### 1. Agent.OrderTracking

**Domain**: Order status inquiries, tracking numbers, delivery estimates
**Temperature**: 0.5 (operational/factual)
**Persona**:
```html
<p><em>You are an expert Order Tracking agent.</em> You help customers check order status, locate tracking numbers, provide delivery estimates, and troubleshoot shipment issues. You maintain a proactive, reassuring tone and always verify order numbers before providing information. You do NOT handle returns or exchanges - defer those to the Returns agent.</p>
```

**Tools**: currentDateTime (check delivery dates), searXNG (carrier tracking lookup)
**Knowledge**: Order tracking procedures, carrier contact info

---

### 2. Agent.Returns

**Domain**: Return policies, RMA processing, refund status, exchanges
**Temperature**: 0.5 (operational)
**Persona**:
```html
<p><em>You are an expert Returns specialist.</em> You guide customers through return policies, initiate RMA requests, track refund status, and process exchange requests. You maintain an empathetic, solution-focused tone. You handle returns up to $500 - escalate higher values to management. You do NOT handle payment method updates - defer to Payment agent.</p>
```

**Tools**: currentDateTime (check return windows), searXNG (policy lookup)
**Knowledge**: Return policies by product category, RMA procedures

---

### 3. Agent.ProductInfo

**Domain**: Product details, specifications, availability, recommendations
**Temperature**: 0.6 (balanced - informational with some personalization)
**Persona**:
```html
<p><em>You are an expert Product Information specialist.</em> You provide detailed product specifications, check availability across locations, answer compatibility questions, and make personalized recommendations. You maintain an enthusiastic, helpful tone. You do NOT handle pricing or promotions - defer to Promotions agent.</p>
```

**Tools**: currentDateTime (check stock dates), searXNG (product spec lookup)
**Knowledge**: Product catalog, compatibility matrices

---

### 4. Agent.Payment

**Domain**: Payment issues, billing questions, payment methods, transactions
**Temperature**: 0.5 (operational - accurate financial info)
**Persona**:
```html
<p><em>You are an expert Payment Support agent.</em> You resolve payment failures, answer billing questions, help update payment methods, and investigate transaction problems. You maintain a patient, security-conscious tone. You verify customer identity before discussing payment details. You do NOT handle refunds - defer to Returns agent.</p>
```

**Tools**: currentDateTime (transaction timestamps), searXNG (payment troubleshooting)
**Knowledge**: Payment troubleshooting guides, supported payment methods

---

### 5. Agent.Shipping

**Domain**: Shipping options, costs, address changes, delivery issues
**Temperature**: 0.5 (operational)
**Persona**:
```html
<p><em>You are an expert Shipping specialist.</em> You explain shipping options and costs, process address changes (pre-shipment only), resolve delivery issues, and provide carrier information. You maintain a clear, logistics-focused tone. You do NOT track existing orders - defer to Order Tracking agent.</p>
```

**Tools**: currentDateTime (delivery date calculations), searXNG (shipping rate lookup)
**Knowledge**: Shipping options by region, carrier capabilities

---

### 6. Agent.Account

**Domain**: Account management, profile updates, password resets, loyalty
**Temperature**: 0.5 (operational)
**Persona**:
```html
<p><em>You are an expert Account Management agent.</em> You assist with profile updates, password resets, email changes, and loyalty program questions. You maintain a friendly, security-aware tone. You verify identity before making account changes. You do NOT handle order-specific questions - defer to Order Tracking agent.</p>
```

**Tools**: currentDateTime (account activity timestamps), searXNG (loyalty program info)
**Knowledge**: Account security procedures, loyalty program tiers

---

### 7. Agent.Promotions

**Domain**: Deals, coupons, discount codes, current sales
**Temperature**: 0.6 (slightly creative - promotional language)
**Persona**:
```html
<p><em>You are an expert Promotions specialist.</em> You provide information about current deals, validate discount codes, explain promotion terms, and highlight relevant sales. You maintain an enthusiastic, value-focused tone. You do NOT apply discounts to existing orders - defer to Payment/Order agents.</p>
```

**Tools**: currentDateTime (check promotion expiry), searXNG (current sales lookup)
**Knowledge**: Active promotions, coupon policy, seasonal sales calendar

---

### 8. Agent.GeneralHelp

**Domain**: Fallback for unclear requests, general assistance, greetings
**Temperature**: 0.5 (balanced)
**Persona**:
```html
<p><em>You are a General Help agent.</em> You assist with questions that don't fit specialized agents, handle greetings, provide clarification, and route users to the correct specialist. You ask clarifying questions when intent is unclear. You maintain a warm, welcoming tone. When you determine the user needs a specialist, say "Let me connect you with our [X] team who can help with that."</p>
```

**Tools**: currentDateTime, searXNG (general search)
**Knowledge**: Company overview, general FAQ

---

## Routing Architecture

### ConditionAgent Configuration

**Router Type**: Scout Mode (comprehensive routing for 8+ agents)
**Temperature**: 0.2 (deterministic routing)
**Model**: gpt-4o-mini (fast, cost-effective for routing)

### Scout Mode Instructions

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
- Route to: Agent.OrderTracking
- Exception: If mentions 'return order' → Agent.Returns

Returns:
- Keywords: 'return', 'refund', 'RMA', 'exchange', 'send back', 'defective', 'damaged'
- Route to: Agent.Returns
- Exception: If only asking 'How do I find return page?' → Agent.GeneralHelp

Product Info:
- Keywords: 'product details', 'specs', 'compatibility', 'in stock', 'availability', 'recommend', 'compare'
- Route to: Agent.ProductInfo
- Exception: If asking about 'discount on product' → Agent.Promotions

Payment:
- Keywords: 'payment failed', 'billing', 'charge', 'credit card', 'payment method', 'transaction error'
- Route to: Agent.Payment
- Exception: If asking for 'refund' → Agent.Returns

Shipping:
- Keywords: 'shipping cost', 'delivery options', 'change address', 'expedited', 'shipping methods'
- Route to: Agent.Shipping
- Exception: If tracking existing order → Agent.OrderTracking

Account:
- Keywords: 'account', 'password', 'login', 'email change', 'profile', 'loyalty points'
- Route to: Agent.Account
- Exception: If asking about 'order history' → Agent.OrderTracking

Promotions:
- Keywords: 'discount', 'coupon', 'promo code', 'sale', 'deal', 'offer'
- Route to: Agent.Promotions

General Help:
- Keywords: 'hello', 'help', 'confused', 'not sure', 'question'
- Route to: Agent.GeneralHelp

MULTI-INTENT HANDLING:
- If question spans multiple domains (e.g., "Track my return"):
  → Route to agent handling PRIMARY action (Returns for return tracking)

FALLBACK RULES:
- Unclear informational questions → Agent.GeneralHelp
- Urgent/specific requests → Route to most likely specialist
- Greetings/small talk → Agent.GeneralHelp
```

### Scenarios Array

8 scenarios in ConditionAgent (one per agent):

1. "Order status, tracking, delivery"
2. "Returns, refunds, exchanges"
3. "Product information, availability"
4. "Payment issues, billing"
5. "Shipping options, costs"
6. "Account management, login"
7. "Promotions, discounts, coupons"
8. "General help, unclear requests"

**Critical**: Scenario count = Agent count (prevents Pattern #4 disconnected agents)

---

## Standard Tools (Auto-Included - CRITICAL)

🚨 **MANDATORY for ALL 8 agents**:

### 1. currentDateTime

Provides temporal context for evaluating:
- Delivery estimates (is it still on time?)
- Promotion expiry (is this code still valid?)
- Return windows (are they within 30 days?)

**Exact Flowise UI Structure**:
```json
{
  "agentSelectedTool": "currentDateTime",
  "agentSelectedToolRequiresHumanInput": "",
  "agentSelectedToolConfig": {
    "agentSelectedTool": "currentDateTime"
  }
}
```

### 2. searXNG

Federated search for real-time information:
- Carrier tracking lookup
- Current promotions
- Product availability
- Policy updates

**Exact Flowise UI Structure**:
```json
{
  "agentSelectedTool": "searXNG",
  "agentSelectedToolRequiresHumanInput": "",
  "agentSelectedToolConfig": {
    "apiBase": "https://s.llam.ai",
    "toolName": "searxng-search",
    "toolDescription": "Federated web/meta search. Use when you need fresh facts or sources.",
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
```

**⚠️ CRITICAL**: Use EXACT field names:
- "searXNG" (capital XNG) NOT "searxng-search"
- "apiBase" NOT "baseUrl"
- `""` (empty string) NOT `false` (boolean)

This prevents Pattern #6 (Incorrect Tool JSON Structure).

---

## Architecture Recommendations

### Node Structure

**Total Nodes**: 10
- 1 Start node (startAgentflow_0)
- 1 ConditionAgent router (conditionAgentAgentflow_0)
- 8 Agent nodes (agentAgentflow_1 through agentAgentflow_8)

**Edge Count**: 9
- 1 edge: Start → Router
- 8 edges: Router output[0-7] → Agent[1-8]

### Memory Configuration

All agents use built-in memory:
- `agentEnableMemory: true`
- `agentMemoryType: "allMessages"`
- `agentMemoryWindowSize: 20`

NO separate memory nodes (self-contained pattern).

### Model Configuration

**Router**: gpt-4o-mini, temperature 0.2 (deterministic)
**Agents**: gpt-4o-mini, temperature 0.5-0.6 (balanced/operational)

All models configured inline via `agentModelConfig` object.

---

## Testing Strategy

### JSON Validation

1. **Structure validation**:
   - File size > 3000 lines (expected for 8-agent system)
   - Has exactly 10 nodes (1 start + 1 router + 8 agents)
   - Has exactly 9 edges
   - All nodes have `type: "agentFlow"`

2. **Pattern compliance**:
   - No separate config file references
   - No phantom tool names (only currentDateTime + searXNG)
   - All agents have 2 standard tools with correct structure
   - Scenario count = Agent count (8 scenarios, 8 agents)

### Flowise Import Testing

1. Import JSON to Flowise
2. Verify all nodes render on canvas
3. Verify no disconnected nodes
4. Test each scenario routes to correct agent
5. Test tools are accessible (not blank/missing)

### Functional Testing

Test queries for each agent:
- "Where is my order #12345?" → OrderTracking
- "I want to return my purchase" → Returns
- "What are the specs for product XYZ?" → ProductInfo
- "My payment failed" → Payment
- "How much is expedited shipping?" → Shipping
- "I forgot my password" → Account
- "Do you have any current sales?" → Promotions
- "Hello, I need help" → GeneralHelp

---

## Known Risks & Mitigations

### Risk 1: Pattern #6 - Incorrect Tool JSON Structure

**Mitigation**: Use EXACT structure from AGENT-NODE-TEMPLATE.json
- Read template lines 341-367 for searXNG structure
- Copy field names precisely (apiBase, not baseUrl)
- Use correct tool name (searXNG, not searxng-search)

### Risk 2: Pattern #4 - Disconnected Agents

**Mitigation**: Ensure scenario count = agent count
- 8 scenarios in conditionAgentScenarios array
- 8 output anchors on router
- 8 edges connecting router outputs to agents

### Risk 3: Pattern #3 - Parallel Build Splitting

**Mitigation**: Single build task for workflow JSON
- Set `parallel_mode: false` in build-tasks.json
- Generate all nodes in one JSON file
- Documentation can be parallel, but NOT workflow JSON

---

## Deployment Readiness

### GitHub Deployment Checklist

- [ ] GitHub CLI (gh) installed: [Will check in Deploy phase]
- [ ] GitHub authentication: [Will check in Deploy phase]
- [ ] Git user configured: [Will check in Deploy phase]

**Note**: Missing GitHub tools will NOT fail the build. Deployment is optional.

---

## File Structure Plan

```
ecommerce-support-workflow/
├── ecommerce-support-flow.json          # SINGLE workflow file (3000+ lines)
├── README.md                             # With embedded Mermaid diagram
├── WORKFLOW-DIAGRAM.md                   # Standalone diagram file
├── INTEGRATION_GUIDE.md                  # Import instructions
└── .context-foundry/
    ├── scout-report.md                   # This file
    ├── architecture.md                   # Next phase
    └── ...
```

**Critical**: ONLY ONE workflow JSON file. All nodes inline.

---

## Timeline Estimate

**Total Duration**: 20-25 minutes

- Scout: ✅ Complete
- Architect: 5 minutes (detailed node specifications)
- Builder: 10-15 minutes (10 nodes + complete structure)
- Test: 3-5 minutes (JSON validation + import test)
- Documentation: 3 minutes (README + integration guide + diagram)
- Deploy: 2 minutes (GitHub push)

---

## Success Criteria

✅ **Complete when**:
1. Single JSON file with ALL nodes inline (3000+ lines)
2. Imports successfully to Flowise
3. All 8 agents render without errors
4. All agents have standard tools (currentDateTime + searXNG)
5. No disconnected nodes
6. Router correctly routes all 8 test queries
7. Mermaid diagram generated and embedded in README
8. Deployed to GitHub

---

**Scout Phase Complete**. Proceeding to Architect phase.
