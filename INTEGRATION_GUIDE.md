# E-commerce Customer Support Workflow - Integration Guide

**Complete setup instructions for importing and deploying this Flowise workflow**

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Import Workflow](#import-workflow)
3. [Configure API Credentials](#configure-api-credentials)
4. [Test Routing](#test-routing)
5. [Deploy to Production](#deploy-to-production)
6. [Custom Configuration](#custom-configuration)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Software

- **Flowise**: Version 1.4 or higher
  - Installation: `npm install -g flowise`
  - Or: Docker deployment
- **Node.js**: Version 18.x or higher (for npm install)
- **OpenAI API Key**: For GPT-4o-mini model

### Recommended Setup

- **Operating System**: Linux, macOS, or Windows with WSL2
- **Memory**: 2GB RAM minimum for Flowise
- **Storage**: 100MB for workflow and dependencies

---

## Import Workflow

### Step 1: Download Workflow File

```bash
# Download ecommerce-support-flow.json to your local machine
wget https://github.com/snedea/ecommerce-support-workflow/raw/main/ecommerce-support-flow.json

# Or clone the entire repository
git clone https://github.com/snedea/ecommerce-support-workflow.git
cd ecommerce-support-workflow
```

### Step 2: Start Flowise

**Option A: NPM Installation**
```bash
# Start Flowise on default port 3000
npx flowise start

# Or specify custom port
PORT=8080 npx flowise start
```

**Option B: Docker**
```bash
docker run -d \
  --name flowise \
  -p 3000:3000 \
  -v ~/.flowise:/root/.flowise \
  flowiseai/flowise
```

**Verify**: Open browser to `http://localhost:3000`

### Step 3: Import JSON File

1. **Navigate to Agentflows**
   - Click **Agentflows** in left sidebar
   - Or go to `http://localhost:3000/agentflows`

2. **Import Workflow**
   - Click **Import** button (top right)
   - Select `ecommerce-support-flow.json`
   - Click **Open**

3. **Verify Import Success**
   - You should see **10 nodes** on the canvas:
     - 1 green Start node (top)
     - 1 pink ConditionAgent router (middle)
     - 8 teal Agent nodes (bottom row)
   - All nodes should be connected (9 edges total)

✅ **Success Indicators**:
- No red error badges on nodes
- No "disconnected" warnings
- All agent labels visible (Agent.OrderTracking, Agent.Returns, etc.)

---

## Configure API Credentials

### Step 1: Add OpenAI API Key

1. **Open Credentials Manager**
   - Click **Credentials** in left sidebar
   - Or go to `http://localhost:3000/credentials`

2. **Create OpenAI Credential**
   - Click **Add Credential** button
   - Select **OpenAI API**
   - Enter credential name: `openai-main`
   - Paste your OpenAI API key
   - Click **Add**

3. **Link to Workflow**
   - Return to Agentflows → Open `ecommerce-support-flow`
   - Click on **ConditionAgent router** node
   - In **Model** dropdown, select your credential
   - Click each **Agent** node and repeat

**Note**: All 9 nodes (1 router + 8 agents) use the same OpenAI credential.

### Step 2: Verify Standard Tools

Standard tools (currentDateTime + searXNG) are **pre-configured** in the JSON. No additional setup needed!

**To verify**:
1. Click any Agent node
2. Scroll to **Tools** section
3. You should see:
   - ✅ currentDateTime (no config needed)
   - ✅ searXNG (apiBase: https://s.llam.ai)

If tools are missing or show errors, re-import the workflow JSON.

---

## Test Routing

### Step 1: Save and Deploy Workflow

1. Click **Save Agentflow** button (top right)
2. Click **Deploy** button
3. Copy the **Agentflow URL** (e.g., `http://localhost:3000/api/v1/prediction/abc123`)

### Step 2: Test with Sample Queries

Use the built-in chat widget or API calls:

**Chat Widget**:
1. Click **Test** button in Flowise UI
2. Try sample queries below
3. Verify correct agent responds

**API Test** (curl):
```bash
AGENTFLOW_URL="http://localhost:3000/api/v1/prediction/YOUR_FLOW_ID"

# Test OrderTracking
curl -X POST "$AGENTFLOW_URL" \
  -H "Content-Type: application/json" \
  -d '{"question": "Where is my order #12345?"}'

# Test Returns
curl -X POST "$AGENTFLOW_URL" \
  -H "Content-Type: application/json" \
  -d '{"question": "I want to return my shoes"}'

# Test ProductInfo
curl -X POST "$AGENTFLOW_URL" \
  -H "Content-Type: application/json" \
  -d '{"question": "What are the specs for product XYZ?"}'
```

### Step 3: Verify Routing Accuracy

Test all 8 agents:

| Query | Expected Agent | Test Result |
|-------|---------------|-------------|
| "Where is my order?" | OrderTracking | ⬜ |
| "I want a refund" | Returns | ⬜ |
| "Is product X in stock?" | ProductInfo | ⬜ |
| "My payment failed" | Payment | ⬜ |
| "How much is expedited shipping?" | Shipping | ⬜ |
| "Reset my password" | Account | ⬜ |
| "Any discount codes?" | Promotions | ⬜ |
| "Hello, I need help" | GeneralHelp | ⬜ |

**Pass Criteria**: 8/8 queries route to correct agent

### Step 4: Test Edge Cases

Test multi-intent and exception handling:

| Query | Expected Behavior |
|-------|------------------|
| "Track my return" | Routes to **Returns** (primary intent) |
| "Discount on product X?" | Routes to **Promotions** (not ProductInfo) |
| "How do I find my order?" | Routes to **OrderTracking** or GeneralHelp (either acceptable) |
| "Update my email for payroll" | Routes to **Account** (primary action) |

---

## Deploy to Production

### Option 1: Embed in Website

**Using Flowise Embed Widget**:
```html
<!-- Add to your HTML <head> -->
<script type="module" src="https://cdn.jsdelivr.net/npm/flowise-embed/dist/web.js"></script>

<!-- Add chat widget to page -->
<flowise-fullchatbot></flowise-fullchatbot>

<script>
  import Chatbot from "https://cdn.jsdelivr.net/npm/flowise-embed/dist/web.js"
  Chatbot.initFull({
    chatflowid: "YOUR_FLOW_ID",
    apiHost: "https://your-flowise-instance.com",
    theme: {
      button: {
        backgroundColor: "#4DD0E1",
        right: 20,
        bottom: 20,
        size: 48
      },
      chatWindow: {
        welcomeMessage: "Hello! How can I help you today?",
        backgroundColor: "#ffffff",
        height: 700,
        width: 400
      }
    }
  })
</script>
```

### Option 2: API Integration

**REST API Endpoint**:
```
POST https://your-flowise-instance.com/api/v1/prediction/{flowId}
```

**Request**:
```json
{
  "question": "User's question here",
  "overrideConfig": {
    "sessionId": "unique-session-id"
  }
}
```

**Response**:
```json
{
  "text": "Agent's response",
  "agentReasoning": [...],
  "sourceDocuments": [...]
}
```

**Example Node.js Integration**:
```javascript
const axios = require('axios');

async function querySupport(userQuestion, sessionId) {
  const response = await axios.post(
    'https://your-flowise-instance.com/api/v1/prediction/YOUR_FLOW_ID',
    {
      question: userQuestion,
      overrideConfig: { sessionId }
    },
    {
      headers: { 'Content-Type': 'application/json' }
    }
  );

  return response.data.text;
}

// Usage
const answer = await querySupport("Where is my order #12345?", "session-abc-123");
console.log(answer);
```

### Option 3: Flowise Cloud Deployment

1. Sign up at [Flowise Cloud](https://flowiseai.com)
2. Import workflow JSON
3. Deploy to production URL
4. Use provided API endpoint in your application

---

## Custom Configuration

### Modify Agent Personas

**To adjust tone or capabilities**:

1. Open workflow in Flowise
2. Click agent node (e.g., Agent.OrderTracking)
3. Edit **Messages** → **System Message**
4. Modify HTML content:

```html
<p><em>You are an expert Order Tracking agent.</em>
[Modify capabilities and tone here]
You do NOT handle returns - defer to Returns agent.</p>
```

5. Save agentflow

**Examples**:
- **Formal tone**: "You maintain a professional, formal tone..."
- **Casual tone**: "You maintain a friendly, conversational tone..."
- **Multilingual**: "You respond in the user's language..."

### Adjust Temperature Settings

**Router temperature** (critical - keep low):
- Current: 0.2
- Range: 0.1-0.3 (deterministic routing)
- **Do NOT** exceed 0.3 (causes inconsistent routing)

**Agent temperature**:
- Operational agents (OrderTracking, Payment, etc.): 0.4-0.6
- Creative agents (Promotions): 0.6-0.8
- Adjust per agent in **Model Config** section

### Add Custom Tools

**Example: Add order lookup API**

1. Go to **Tools** in Flowise
2. Click **Add Custom Tool**
3. Configure:
   ```json
   {
     "name": "order-lookup",
     "description": "Fetch order details from e-commerce API",
     "endpoint": "https://api.yourstore.com/orders/{orderId}",
     "method": "GET",
     "auth": "Bearer {API_KEY}",
     "parameters": {
       "orderId": {
         "type": "string",
         "required": true
       }
     }
   }
   ```
4. Open workflow → Agent.OrderTracking node
5. Add tool to **agentTools** array
6. Save and test

---

## Troubleshooting

### Import Issues

**Problem**: "Failed to import JSON"
**Cause**: File corrupted or incomplete download
**Solution**:
```bash
# Verify file integrity
wc -l ecommerce-support-flow.json
# Should output: 975

# Re-download if line count is wrong
wget https://github.com/snedea/ecommerce-support-workflow/raw/main/ecommerce-support-flow.json
```

---

**Problem**: Nodes render but connections missing
**Cause**: Edge definitions not loaded
**Solution**:
```bash
# Verify edge count
jq '.edges | length' ecommerce-support-flow.json
# Should output: 9

# If wrong, re-import workflow
```

---

### Routing Issues

**Problem**: All queries go to GeneralHelp
**Cause**: Router temperature too high or instructions not loaded
**Solution**:
1. Click ConditionAgent router node
2. Verify **Instructions** field contains Scout Mode text (should be ~50 lines)
3. Verify **Temperature** = 0.2
4. If empty, re-import workflow

---

**Problem**: Router sends queries to wrong agent
**Cause**: OpenAI credential not set or model mismatch
**Solution**:
1. Verify OpenAI credential configured in **Credentials**
2. Check router uses `gpt-4o-mini` model
3. Test with explicit queries ("I want to return X" should always go to Returns)

---

### Tool Issues

**Problem**: "searXNG not found" error
**Cause**: Tool structure incorrect (Pattern #6)
**Solution**: Re-import workflow - tools are pre-configured correctly

---

**Problem**: currentDateTime returns old timestamp
**Cause**: Caching in Flowise
**Solution**: Clear chat history and restart conversation

---

### Memory Issues

**Problem**: Agents forget previous messages
**Cause**: Memory not enabled
**Solution**:
1. Click agent node
2. Verify **Enable Memory** = true
3. Verify **Memory Type** = "allMessages"
4. Verify **Window Size** = 20

---

### Performance Issues

**Problem**: Slow response times (>10 seconds)
**Cause**: Model overhead or network latency
**Solution**:
- Use gpt-4o-mini (faster than gpt-4)
- Reduce memory window size if very long conversations
- Check OpenAI API status

---

**Problem**: High API costs
**Cause**: Large context windows or high traffic
**Solution**:
- Reduce memory window from 20 to 10 messages
- Use gpt-3.5-turbo for non-critical agents
- Implement rate limiting in production

---

## Advanced Configuration

### Multi-Language Support

Add language detection to router:

```json
{
  "conditionAgentInstructions": "Detect language first, then route...

  If language is Spanish, prepend 'es-' to scenario selection.
  If language is French, prepend 'fr-' to scenario selection.
  ..."
}
```

Then create language-specific agents (Agent.OrderTracking.ES, Agent.OrderTracking.FR, etc.).

### Human-in-the-Loop Approval

For sensitive operations (refunds > $500):

1. Create custom tool with `requiresHumanInput: true`
2. Add to Returns agent
3. Configure approval workflow in Flowise

### A/B Testing Routing Strategies

Duplicate workflow and modify router instructions:

- **Version A**: Scout Mode (current)
- **Version B**: Simple keyword-based routing
- Compare routing accuracy with test queries

### Analytics Integration

Log agent selections for analysis:

```javascript
// In custom tool or webhook
const logRouting = async (query, selectedAgent) => {
  await fetch('https://your-analytics.com/log', {
    method: 'POST',
    body: JSON.stringify({
      timestamp: new Date(),
      query,
      agent: selectedAgent
    })
  });
};
```

---

## Best Practices

### Security

- ✅ Store API keys in Flowise credentials (never in JSON)
- ✅ Use HTTPS for production deployment
- ✅ Implement rate limiting to prevent abuse
- ✅ Never log sensitive customer data (credit cards, passwords)

### Performance

- ✅ Keep memory window reasonable (10-20 messages)
- ✅ Use gpt-4o-mini for cost-effective performance
- ✅ Cache static knowledge in knowledge bases (not tools)
- ✅ Monitor API usage and set budgets

### Maintenance

- ✅ Review routing logs monthly for accuracy
- ✅ Update agent personas based on customer feedback
- ✅ Test workflow after Flowise upgrades
- ✅ Version control your workflow JSON in git

---

## Next Steps

1. ✅ Import workflow to Flowise
2. ✅ Configure OpenAI credentials
3. ✅ Test all 8 routing scenarios
4. ✅ Customize agent personas for your brand
5. ✅ Deploy to production (embed or API)
6. ✅ Monitor performance and iterate

---

## Support Resources

- **Flowise Documentation**: https://docs.flowiseai.com/
- **OpenAI API Reference**: https://platform.openai.com/docs/api-reference
- **GitHub Repository**: https://github.com/snedea/ecommerce-support-workflow
- **Workflow Diagram**: [WORKFLOW-DIAGRAM.md](./WORKFLOW-DIAGRAM.md)

---

**Last Updated**: January 2025
**Tested With**: Flowise v1.4.11, OpenAI GPT-4o-mini
