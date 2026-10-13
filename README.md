# AB-620_AB-100_Preparation

# AB-620 — Study & Revision Guide

> **Exam:** Designing and Building Integrated AI Agent Solutions in Copilot Studio  
> **Purpose:** Study notes and post-video revision, aligned with the supplied exam objectives.  
> **Scope:** Copilot Studio standard-harness features, including topics and generative answers nodes.

## How to use this guide

1. Watch the matching video chapter.
2. Read the explanation.
3. Focus on **when to choose each option**.
4. Check the **exam traps**.
5. Use the final checklist to test your understanding.

**Source priority:** The official exam objectives define the scope. Microsoft product documentation takes priority over simplified rules in the video or PDF.

**Version note:** Copilot Studio documentation distinguishes different agent-building experiences and harnesses. Do not mix instructions from another experience into this workflow without checking applicability.

This guide is a study and revision resource—not a guarantee that the video and notes alone are sufficient to pass.

### Study materials

- [Official AB-620 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ab-620)
- [Study video](https://www.youtube.com/watch?v=TygN-nIFP7Y)
- Supplied presentation: `AB_620_Study_Guide_Presentation.pdf`
- [Copilot Studio documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)

---

## Table of contents

1. [Exam coverage](#exam-coverage)
2. [The core mental model](#1-the-core-mental-model)
3. [Agent-level vs topicnode-level control](#2-agent-level-vs-topicnode-level-control)
4. [Child agents vs connected agents](#3-child-agents-vs-connected-agents)
5. [Routing accuracy and performance](#4-routing-accuracy-and-performance)
6. [Enterprise knowledge and connectors](#5-enterprise-knowledge-and-connectors)
7. [Generative answers, custom prompts, and RAG](#6-generative-answers-custom-prompts-and-rag)
8. [Azure AI Search and Foundry](#7-azure-ai-search-and-foundry)
9. [Responsible AI, security, identity, and channels](#8-responsible-ai-security-identity-and-channels)
10. [Topics, variables, Power Fx, and Adaptive Cards](#9-topics-variables-power-fx-and-adaptive-cards)
11. [Choosing a tool integration](#10-choosing-a-tool-integration)
12. [Agent flows and human-in-the-loop](#11-agent-flows-and-human-in-the-loop)
13. [Computer use](#12-computer-use)
14. [A2A vs MCP](#13-a2a-vs-mcp)
15. [Evaluations](#14-evaluations)
16. [Application Insights and production monitoring](#15-application-insights-and-production-monitoring)
17. [ALM](#16-alm-environments-solutions-variables-pipelines)
18. [Fast scenario-selection sheet](#17-fast-scenario-selection-sheet)
19. [Final revision checklist](#18-final-revision-checklist)

---

## Exam coverage

| Domain | Weight |
|---|---:|
| Plan and configure agent solutions | 30–35% |
| Integrate and extend agents in Copilot Studio | 40–45% |
| Test and manage agents | 20–25% |

Integration has the highest weighting, but planning, evaluation, and ALM are substantial parts of the exam—not optional extras.

**Reference:** [Official exam objectives](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ab-620)

---

## 1. The core mental model

**Video:** [05:50 — Agent Orchestration](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=350s)

An agent combines a model with the surrounding configuration and runtime that let it perform useful work.

```text
Agent = Model + Harness
```

For choosing integrations, think in three categories:

| Capability | Purpose | Example |
|---|---|---|
| Knowledge | Ground answers in information | Explain the return policy |
| Tools | Execute an operation or retrieve operational data | Check inventory or create a ticket |
| Other agents | Delegate work to a specialized reasoning system | Ask a finance agent to analyze spending |

### Important distinctions

- A tool does not have to modify data.
- A read-only operation such as `GetInventory` is still a tool.
- Knowledge is not limited to static documents.
- Supported real-time knowledge connectors can retrieve enterprise data at runtime.

### Exam decision

Ask:

- **Does the user need an answer grounded in content?** → Knowledge.
- **Does the user need a specific system operation?** → Tool.
- **Does the task require another agent’s independent reasoning?** → Agent delegation.

**Reference:** [Use connectors in Copilot Studio agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-connectors)

---

## 2. Agent-level vs topic/node-level control

**Video:** [05:50 — Agent Orchestration](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=350s)

These are **two places to expose or invoke capabilities**, not two mutually exclusive architectures.

### Agent level

You make a capability available to the orchestrator.

The orchestrator uses the request, instructions, descriptions, and conversation context to determine whether to use it.

```text
User request
    → Orchestrator selects suitable capability
    → Capability executes
    → Result informs the response or next step
```

**Choose this when:**

- Requests vary widely.
- Users combine several intentions.
- The agent needs flexibility in selecting resources.
- You do not need to author every invocation explicitly.

### Topic/node level

You author a conversational workflow and specify where resources are invoked.

```text
Collect order ID
    → Check eligibility
    → Evaluate condition
    → Ask for confirmation
    → Create return request
```

**Choose this when:**

- A step must occur before another step.
- Inputs require explicit validation.
- A business process needs controlled branching.
- A resource must be used at a specific point.

Connectors can be invoked at agent level or inside topics.

A topic can also explicitly redirect to a child or connected agent and resume after that agent finishes.

### How they work together

```text
Generative orchestration
    ├─ Selects a tool directly
    ├─ Searches knowledge
    ├─ Delegates to another agent
    └─ Selects an authored topic
           ├─ Runs questions and conditions
           ├─ Calls tools
           └─ Can include generative steps
```

### Important corrections

- The orchestrator does **not necessarily make a fresh routing decision for every message**. A message might be answering a question in an active topic.
- Entering a topic does **not mean the model disappears**. A topic can invoke prompts, generative answers, or other agents.
- “More deterministic” means **more explicitly controlled**, not “guaranteed to produce the same result.”
- Topics can branch, fail, be interrupted, or depend on variable external results.
- A topic is **not a security boundary**. Authorization and business validation must also be enforced by the underlying systems.

### Memory rule

> Agent level controls capability selection dynamically.  
> Topic/node level controls invocation points explicitly.

**References:**

- [Use connectors](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-connectors)
- [Add other agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-add-other-agents)

---

## 3. Child agents vs connected agents

**Video:** [12:20 — Connect Agents Together](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=740s)

| Dimension | Child agent | Connected agent |
|---|---|---|
| Relationship | Part of a parent agent | Separately built agent |
| Main purpose | Modularize one solution | Reuse or delegate to an independent capability |
| Lifecycle | Managed with the parent | Separate publishing/deployment lifecycle |
| Ownership | Usually one cohesive team | Same team or another team |
| Independent reuse | Not the primary design | Appropriate for multiple orchestrators |

### Choose a child agent when

- One team owns the complete solution.
- You want to group related instructions, tools, and knowledge.
- The component does not need independent deployment.
- The specialization is specific to the parent solution.

### Choose a connected agent when

- The required agent already exists.
- The component needs independent testing or deployment.
- Several orchestrators need to reuse it.
- Another platform or team owns the capability.

**The deciding factor is independence—not merely who built it.**

Both types can coexist in one solution.

### Connected-agent choices

| Requirement | Candidate |
|---|---|
| Existing published agent in the Copilot Studio environment | Copilot Studio agent |
| Specialized agent built and configured in Foundry | Foundry agent |
| Conversational analysis over supported Fabric data sources | Fabric data agent |
| External agent exposing the A2A protocol | A2A agent |
| Agent built with Microsoft 365 Agents SDK | Supported SDK-agent integration |

Availability and prerequisites differ by integration. Several external-agent integrations are documented as preview.

### Exam trap

“Too many choices” can justify modularization, but it does **not automatically mean child agent**.

First determine whether the extracted capability needs an independent lifecycle or reuse.

**Reference:** [Add other agents overview](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-add-other-agents)

---

## 4. Routing accuracy and performance

**Video:** [18:55 — Agent Performance](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=1135s)

### When the wrong resource is selected

Inspect the execution/activity trace, then check:

1. Resource name and description.
2. Whether descriptions overlap.
3. Whether the correct capability is available.
4. Input descriptions and requirements.
5. Agent instructions.
6. Whether the agent has too many competing capabilities.

#### Weak description

```text
Handles orders.
```

#### Better description

```text
Retrieves delivery status and tracking information for an existing order.

Use when the user asks where an order or package is.

Do not use to create, cancel, or return an order.
```

Microsoft gives **30–40 action choices** as a practical rule of thumb for possible routing degradation—not a hard platform limit.

Similar descriptions can cause problems with fewer choices.

Validate the design through evaluations.

### Performance trade-offs

Delegation adds work.

Connected agents can introduce:

- Extra orchestration hops.
- Latency.
- A larger testing surface.
- A larger governance surface.

**Do not memorize:**

```text
Child agents have zero cost or latency.
```

The safer rule is:

> Independent connected agents can add orchestration overhead. Actual performance must be measured.

**Reference:** [Multi-agent design considerations](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-add-other-agents)

---

## 5. Enterprise knowledge and connectors

**Video:**

- [23:00 — Agent Knowledge](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=1380s)
- [38:30 — Connectors](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=2310s)

### Copilot connectors

For the **indexed pattern described in the supplied PDF**:

```text
External content
    → Ingestion and indexing
    → Searchable enterprise knowledge
    → Grounded answer
```

Typical requirement:

> Make external enterprise content searchable and citable through Microsoft 365 experiences.

Think about:

- Administrator setup.
- Content freshness and ingestion.
- Permission-aware retrieval.
- Searchable content and citations.

### Power Platform connectors as knowledge

Supported real-time knowledge connectors:

- Index metadata such as table and column names.
- Execute requests against the source at runtime.
- Use the user’s authentication context.
- Retain source-system access controls.
- Do not replicate the underlying enterprise data into the knowledge index.

### Power Platform connectors as tools

They expose operations such as:

- Retrieve a record.
- Create a ticket.
- Update a customer.
- Send an email.

They can be used as:

- Agent tools.
- Topic actions.
- Actions inside agent flows.

### Important correction

This rule is too simplistic:

```text
Power Platform connectors = tools only
```

Use this instead:

```text
Power Platform connector
    ├─ Tool/action integration
    └─ Supported real-time knowledge integration
```

### Exam decision table

| Scenario | Strong candidate |
|---|---|
| Search indexed external policy content | Copilot connector |
| Answer using current enterprise data without replicating records | Supported real-time knowledge connector |
| Create or update a record | Connector tool |
| Ground answers in an existing Azure search index | Azure AI Search |

**References:**

- [Power Platform connectors as knowledge](https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-real-time-connectors)
- [Use connectors in agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-connectors)

---

## 6. Generative answers, custom prompts, and RAG

**Video:** [26:10 — Generative Answers](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=1570s)

### Generative answers node

A topic node that retrieves from configured knowledge and generates an answer.

**Use it when:**

- Knowledge must be queried at a specific point.
- A topic needs a controlled source selection.
- You want to capture or format a grounded answer.

Node-specific knowledge sources take priority over agent-level sources for that node.

### Terminology trap

> The **generative answers node** belongs in a topic.  
> Generating answers from knowledge is **not limited to authored topics**.

Agent-level orchestration can also use knowledge.

### Custom prompt

Defines an AI task with instructions and inputs.

Examples:

- Summarize a support ticket.
- Classify a complaint.
- Extract fields from text.
- Rewrite a response in a required format.

| Requirement | Choose |
|---|---|
| “What does our policy say?” | Knowledge retrieval / generative answers |
| “Summarize this ticket into three fields” | Custom prompt |
| “Submit this ticket to ServiceNow” | Tool/action |

A prompt may use supplied grounding data, but **a prompt alone does not create a retrieval pipeline**.

### RAG

RAG stands for **Retrieval-Augmented Generation**.

```text
Question
    → Retrieve relevant information
    → Add retrieved information to model context
    → Generate a grounded answer
```

**RAG is not model training or fine-tuning.**

It provides information at inference time.

### Grounding trap

Turning off ungrounded responses does not guarantee that a model never incorporates general knowledge.

Microsoft documents this setting as blocking turns that use no knowledge source or tool—not eliminating all underlying model knowledge.

**References:**

- [Knowledge sources summary](https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio)
- [Knowledge sources in generative answers nodes](https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-generative-answers-sharepoint-onedrive)

---

## 7. Azure AI Search and Foundry

**Video:** [28:29 — Azure AI Search](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=1709s)

### Azure AI Search

A retrieval service for indexed content, including vector-search scenarios.

**Choose it when:**

- The organization already maintains Azure search indexes.
- You need retrieval over large enterprise document collections.
- The solution needs vector search or semantic ranking.

### Understand the components

| Component | Role |
|---|---|
| Embedding model | Converts content/query into vector representations |
| Search index | Stores searchable content and metadata |
| Vector search | Retrieves semantically similar content |
| Semantic ranker | Improves ordering of suitable search results |
| Generative model | Produces the answer using retrieved information |

### Configuration sequence

1. Prepare and index the source content.
2. Configure vectorization and/or semantic ranking as required.
3. Establish the Azure AI Search data connection.
4. Select the index.
5. Configure retrieval and test representative questions.
6. Verify citations and access.

For citations:

- Include document URLs in the index.
- Ensure users have access to the linked source documents.

### Foundry integration: three different purposes

Do not confuse:

1. **Foundry agent integration** — delegate to an independently configured agent.
2. **Foundry model integration** — use a supported deployed model for a custom prompt.
3. **Azure AI Search integration** — retrieve grounding information.

For custom prompts, supported Foundry model connections let you use specialized or optimized deployed models.

Consider:

- Task suitability.
- Latency.
- Cost.
- Input modality.
- Availability.
- Security.

Do not select a model based only on its size.

### Exam trap

Changing a custom prompt’s model is not the same as replacing the entire agent’s orchestration model.

**References:**

- [Add Azure AI Search as knowledge](https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-azure-ai-search)
- [Use a Foundry model in prompt builder](https://learn.microsoft.com/en-us/power-platform/release-plan/2025wave1/ai-builder/use-own-generative-ai-model-azure-ai-foundry-prompt-builder)

---

## 8. Responsible AI, security, identity, and channels

**Video:** [30:25 — Content Moderation](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=1825s)

This section also covers planning objectives that the PDF treats lightly.

### Separate these concerns

| Concern | Question |
|---|---|
| Authentication | Who is the caller? |
| Authorization | What may the caller access or change? |
| Moderation | Is the content harmful or inappropriate? |
| Grounding | Is the answer supported by information? |
| Governance | Which capabilities and data paths are allowed? |
| Human oversight | Which decisions require review or approval? |

**Moderation does not enforce database permissions.**

**Instructions do not replace authorization.**

### Agent sign-in vs tool authentication

These are separate:

- The user can be signed in to the agent.
- A downstream tool may still require its own connection, consent, scopes, and permissions.

### Execution identity

| Identity | Use when | Main risk |
|---|---|---|
| End-user/delegated | Access should respect each user’s permissions | Consent and connection availability |
| Maker-provided connection | A controlled shared connection is appropriate | Users may act with the connection owner’s access |
| Supported application/service identity | Background work should not depend on an employee | Excessive application permissions |
| No authentication | Endpoint intentionally allows anonymous access | Inappropriate exposure of sensitive operations |

Support varies by connector and integration.

Do not assume every tool supports every identity option.

**OAuth is not limited to Microsoft Entra ID.**

It is an authorization framework that can be used with other supported identity providers.

### Governance

Power Platform data policies can govern:

- Knowledge sources.
- Connectors.
- HTTP requests.
- Authentication.
- Channels.
- Triggers.
- Application Insights.

If an integration is blocked by policy, rewriting the agent instructions will not unblock it.

### Internal vs external audience

#### Internal agent

- Employee identity.
- Enterprise permissions.
- Approved internal knowledge.
- Suitable organizational channels.

#### External agent

- Explicit customer/public authentication strategy.
- Strict separation from internal-only data.
- Approved public-facing capabilities.
- Validation before sensitive actions.

### Channels and deployment

Evaluate:

- Where the intended audience works.
- Authentication support.
- Adaptive Card support.
- Tool and integration compatibility.
- Publication and sharing requirements.
- Administrator policies.

**Publishing is not the same as granting every user access to every downstream resource.**

**Reference:** [Copilot Studio security and governance](https://learn.microsoft.com/en-us/microsoft-copilot-studio/security-and-governance)

---

## 9. Topics, variables, Power Fx, and Adaptive Cards

**Supplement to the video — explicit exam objectives**

### Topic nodes

Be comfortable with:

- Message.
- Question.
- Condition.
- Set variable value.
- Call a tool or agent flow.
- Generative answers.
- Prompt invocation.
- HTTP request.
- Agent redirect.
- Topic/conversation ending behavior.

### Topic vs agent flow

> A topic controls the conversational experience.  
> An agent flow implements process automation.

A topic can gather information and invoke a flow to perform the business process.

### Variable scopes

| Scope | Purpose |
|---|---|
| Topic | Information local to a topic |
| Global | Information shared across topics in a conversation |
| System | Platform-provided context |
| Environment | Deployment-specific configuration |

**Global does not mean shared across all users.**

Map input and output types correctly.

A text value, record, and table are not interchangeable.

### Power Fx

Use it for:

- Conditions.
- Formatting.
- Transformations.
- Working with records/tables.
- Handling blank values.
- Building dynamic content.

Useful functions to recognize:

```text
If
IsBlank
Coalesce
Text
Value
Concatenate
Filter
LookUp
```

Illustrative condition:

```powerfx
If(
    Topic.OrderTotal > 1000,
    "Approval required",
    "Standard processing"
)
```

### Adaptive Cards

Structured UI for displaying information or collecting several inputs.

#### Configuration checklist

1. Define the card.
2. Give input fields identifiers.
3. Configure submission.
4. Map submitted values to variables.
5. Validate the values.
6. Test in the target channel.

Power Fx can make cards dynamic by referencing topic or agent variables.

### Exam trap

A card is a presentation/input mechanism—not backend authorization or business validation.

**References:**

- [Work with variables](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-variables)
- [Ask with Adaptive Cards](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-ask-with-adaptive-card)

---

## 10. Choosing a tool integration

**Video:**

- [32:30 — Agent Tools](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=1950s)
- [38:30 — Connectors](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=2310s)
- [39:40 — Custom Connectors](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=2380s)
- [41:30 — REST APIs](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=2490s)
- [42:40 — HTTP](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=2560s)
- [44:45 — MCP](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=2685s)

| Option | Best fit |
|---|---|
| Prebuilt connector | Supported operation already exists |
| Custom connector | Reusable integration with an internal/unsupported API |
| REST API tool | Expose API operations as callable agent tools |
| HTTP request | Explicit request at a controlled topic/flow step |
| MCP server | Discover and invoke capabilities exposed through MCP |
| Agent flow | Multi-step business automation |
| Computer use | Interact with an application through its UI |
| Connected agent | Delegate a task to independent reasoning |

### Prebuilt connector

Prefer it when it meets the requirements and governance constraints.

Check:

- Available operation.
- Authentication.
- Input/output schema.
- Licensing and policy.
- Connection permissions.

### Custom connector

Reusable wrapper around an API.

**Choose it when:**

- No suitable prebuilt integration exists.
- Multiple agents, flows, or apps need the API.
- You want standardized operation definitions and authentication.

**Trap:** One custom connector can expose several operations.

You do not need a connector per endpoint.

### REST API tool

Copilot Studio’s API-tool onboarding uses:

- An OpenAPI specification.
- Authentication details.
- Selected operations.

The documented creation process uses OpenAPI v2. Submitted v3 specifications are translated to v2.

### HTTP request

Understand:

```text
URL
Method
Headers
Authentication
Request body
Response schema
Status/error handling
```

#### Common troubleshooting cues

| Result | Inspect |
|---|---|
| 401 | Authentication/token |
| 403 | Permissions/authorization |
| 400 | Request shape or invalid inputs |
| 429 | Throttling/retry policy |
| 5xx | Downstream service failure |
| Parsing error | Response schema/type mismatch |

**Do not memorize:**

```text
HTTP is only for an API used once.
```

HTTP calls can run repeatedly.

The real distinction is explicit request control versus a reusable integration/tool abstraction.

### MCP

MCP stands for **Model Context Protocol**.

```text
Agent
    → MCP server
    → Exposed tools
    → Underlying systems
```

The server supplies:

- Tool names.
- Descriptions.
- Inputs.
- Outputs.

Copilot Studio dynamically reflects changes to exposed tools/resources.

Generative orchestration must be enabled for the documented MCP integration.

**Choose MCP when:**

- A server already exposes the capabilities.
- Tool discovery matters.
- The operation set evolves.
- You want a standardized agent-facing integration.

MCP access is also governed through Power Platform connector policies.

### Exam trap

MCP is an integration protocol—not an LLM and not automatic permission to use every operation.

**References:**

- [REST API tools](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agent-extend-action-rest-api)
- [Extend an agent with MCP](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agent-extend-action-mcp)
- [Connect to an existing MCP server](https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-add-existing-server-to-agent)

---

## 11. Agent flows and human-in-the-loop

**Video:** [48:25 — Agent Flows](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=2905s)

Use flows for:

- Multi-system operations.
- Branching business logic.
- Approvals.
- Human input.
- Error handling.
- Background processing.

### Flow called by an agent

The documented requirements include:

```text
When an agent calls the flow
    → Process inputs
    → Execute actions
    → Respond to the agent
```

The flow must:

- Be published.
- Use a real-time response configuration.
- Respond within the **100-second action limit**.

### Long-running work

Do not hold a synchronous agent call open while waiting hours for approval.

A suitable design can:

1. Create a request/job.
2. Return an acknowledgment and identifier promptly.
3. Continue through a supported background workflow.
4. Notify the user or expose status retrieval later.

**Important:** Returning an acknowledgment does not mean the business operation has completed.

### Human-in-the-loop

Use approval or **Request information** capabilities when human judgment or missing information is required.

The RFI action specifically incorporates human input into agent flows.

### Monitoring and error handling

Inspect:

- Failed action.
- Input/output values.
- Connection identity.
- Status/error code.
- Duration and timeout.
- Retry behavior.
- Output mapping.

Design safe retries.

Repeating a read is different from repeating a payment or record-creation action.

### Memory rule

> If the correct flow was selected but failed, investigate execution—not routing descriptions first.

**References:**

- [Add an agent flow as a tool](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flow-agent)
- [Request information from human review](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flows-request-for-information)

---

## 12. Computer use

**Supplement — explicit exam objective**

Computer use lets an agent interact with web and desktop applications through a virtual mouse and keyboard.

**Choose it when:**

- The task requires UI interaction.
- A suitable API or connector is unavailable.
- A legacy application must be operated visually.

### Configure

- Name and description.
- Model and task instructions.
- Execution machine.
- Connection and execution credentials.
- Website/application credentials.
- Access restrictions.
- Human supervision.

Microsoft warns that sharing a tool using maker-provided credentials can let users act with the author’s access on the configured machine.

### Monitor

Review:

- Activity map.
- Step logs.
- Screenshots.
- Outcomes.

Detailed computer-use logs can be stored in Dataverse and consume database, log, and file capacity.

### Exam trap

AI-based adaptation to UI changes is not a guarantee that every UI change will succeed.

**References:**

- [Automate apps with computer use](https://learn.microsoft.com/en-us/microsoft-copilot-studio/computer-use)
- [Monitor computer use](https://learn.microsoft.com/en-us/microsoft-copilot-studio/monitor-computer-use)

---

## 13. A2A vs MCP

**Video:** [51:10 — Agent2Agent Protocol](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=3070s)

| MCP | A2A |
|---|---|
| Access tools and capabilities | Delegate work to another agent |
| Caller reasons about tool results | Target agent performs its own reasoning |
| Example: retrieve stock count | Example: investigate a supply-chain problem |

### A2A concepts

- **Agent Card:** Describes identity, capabilities, and connection-related metadata.
- **Task:** Unit of delegated work.
- **Context:** Tracks related interactions.
- **Result/artifact:** Output produced by the target agent.

For the Copilot Studio integration:

- Enter the **agent communication endpoint**.
- Do not enter the Agent Card URL instead.
- A valid card at the supported standard location can populate the agent’s name and description.

### Exam traps

- Do not treat the PDF’s Agent Card path as a universal, version-independent constant.
- A2A is cross-platform.
- A2A is not restricted to non-Microsoft agents.

### Memory rule

> MCP: “Use this capability.”  
> A2A: “Handle this task.”

**Reference:** [Connect to an agent over A2A](https://learn.microsoft.com/en-us/microsoft-copilot-studio/add-agent-agent-to-agent)

---

## 14. Evaluations

**Video:** [53:20 — Agent Evaluations](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=3200s)

### Build a representative test set

Include:

- Normal requests.
- Paraphrases.
- Ambiguous requests.
- Missing inputs.
- Multi-intent requests.
- Out-of-scope questions.
- Tool failures.
- Permission-sensitive scenarios.
- Unsafe requests.
- Expected refusals.

Keep important cases stable so you can compare behavior after changes.

### Choose the evaluation method

| Method | Use when |
|---|---|
| General quality | Assess relevance, groundedness, completeness, and answering behavior |
| Compare meaning | Correct answers may use different wording |
| Tool use | Verify expected tools/topics were used |
| Keyword match | Required words/phrases must appear |
| Text similarity | Wording/construction should closely resemble an expected response |
| Exact match | Output must match exactly |
| Content safety | Assess harmful-content categories |
| Custom | Apply your own criteria and pass/fail labels |

General quality does not require expected answers.

Other methods may require:

- Expected answers.
- Keywords.
- Expected capabilities.

Missing required reference data can produce an **Invalid** result.

### Useful examples

| Requirement | Method |
|---|---|
| Correct explanation in different words | Compare meaning |
| Must invoke `GetInventory` | Tool use |
| Must return exactly `APPROVED` | Exact match |
| Must satisfy organization-specific policy | Custom |

### Refusal trap

General quality can flag an appropriate refusal because of its abstention criterion.

Use custom criteria when refusal is the desired passing behavior.

### Diagnose failures

These are **hypotheses to investigate**, not guaranteed diagnoses.

| Pattern | First checks |
|---|---|
| Same tool fails across many cases | Connection, permissions, endpoint, schema, timeout |
| Wrong tool for one intent | Overlapping/unclear descriptions |
| Broad routing problems | Instructions and capability complexity |
| Correct tool, wrong inputs | Input descriptions, extraction, mapping, validation |
| Correct retrieval, poor answer | Prompt, grounding, formatting |
| Intermittent failures | Model variability, throttling, latency, changing data/context |
| Evaluation marked Invalid | Missing expected data/configuration |

### Exam trap

Passing tool-use evaluation proves resource selection—not that the business operation succeeded.

**Reference:** [Choose evaluation methods](https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-overview)

---

## 15. Application Insights and production monitoring

**Supplement — explicit exam objective**

### Separate the monitoring mechanisms

| Mechanism | Purpose |
|---|---|
| Evaluations | Test behavior against defined cases |
| Activity map/trace | Inspect a specific execution |
| Flow run history | Diagnose workflow actions |
| Native analytics | Understand usage/outcomes |
| Application Insights | Capture and analyze agent telemetry |

Agent-level Application Insights telemetry can capture events useful for understanding topic activity and runtime behavior.

### Configuration and investigation sequence

1. Configure the telemetry connection.
2. Generate test/runtime activity.
3. Inspect events and correlate failures.
4. Distinguish test conversations from production where appropriate.
5. Apply privacy, retention, and access controls.

### Memory rule

> Evaluation measures expected quality.  
> Monitoring observes what actually happened.

**Reference:** [Agent-level telemetry with Application Insights](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-bot-framework-composer-capture-telemetry)

---

## 16. ALM: environments, solutions, variables, pipelines

**Video:**

- [57:20 — Agent Lifecycle Management](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=3440s)
- [01:01:50 — Environment Variables](https://www.youtube.com/watch?v=TygN-nIFP7Y&t=3710s)

### Environments

Separate development, testing, and production.

```text
Development
    → Test
    → Production
```

Environment separation supports:

- Safer testing.
- Distinct permissions.
- Environment-specific configuration.

### Solutions

Package and transport the agent and related components.

Include the relevant:

- Agent.
- Flows.
- Connector components.
- Connection references.
- Environment variables.
- Dependencies.

### Managed vs unmanaged

| Unmanaged | Managed |
|---|---|
| Development/source customization | Deployment artifact for downstream environments |
| Edit components during development | Controlled deployment and servicing |
| Export as unmanaged or managed | Use for test/production deployment |

### Correction to the PDF

This statement is too broad:

```text
All solutions must always be exported as managed.
```

An unmanaged source solution can be exported in either form.

Use:

- **Managed artifacts** for downstream deployments.
- **Unmanaged exports** for development/source-control purposes where appropriate.

Selecting **managed** during export does not turn the original development solution into a managed solution in place.

### Environment variables vs connection references

| Component | Purpose |
|---|---|
| Environment variable | Store environment-specific configuration |
| Connection reference | Bind a solution component to a connection in that environment |

Examples of configuration:

```text
Development API endpoint
Test API endpoint
Production API endpoint
SharePoint site URL
Feature flag
```

**A URL variable does not carry authentication credentials.**

### Republish behavior

For environment variables used by a published Copilot Studio agent:

| Variable type | Behavior after a value changes |
|---|---|
| Non-secret | Values are included when publishing; changes require republishing |
| Secret | Values are retrieved at runtime; changes do not require agent republishing |

Do not generalize this into:

```text
Every consuming system instantly sees every secret update.
```

### Power Platform Pipelines

Pipelines promote solution artifacts through configured stages.

Know how to:

- Configure source and target environments.
- Deploy the solution.
- Supply configuration and connection bindings.
- Apply approvals.
- Review deployment results.
- Extend the process with supported automation.

Power Platform Pipelines deploy managed solutions to target environments.

They also retain an unmanaged export in the host for development/source-control uses.

Pipeline extensibility supports additional automation, including approval-related scenarios.

### Exam trap

Successful solution deployment does not prove that all of these are correct:

- Target connections.
- Permissions.
- Channel publication.
- Runtime behavior.

**References:**

- [Solution concepts](https://learn.microsoft.com/en-us/power-platform/alm/solution-concepts-alm)
- [Variables overview and republish behavior](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-variables-about)
- [Power Platform Pipelines](https://learn.microsoft.com/en-us/power-platform/alm/pipelines)
- [Extend pipelines](https://learn.microsoft.com/en-us/power-platform/alm/extend-pipelines)

---

## 17. Fast scenario-selection sheet

Use these as decision cues—not blind keyword matching.

| Requirement | Strong candidate |
|---|---|
| Search enterprise policy content | Knowledge |
| Retrieve a specific current record | Tool or supported real-time knowledge, depending on the requirement |
| Create/update a record | Tool |
| Explicit ordered conversation | Topic |
| Multi-system business process | Agent flow |
| Human review or missing information | Approval/RFI |
| Existing suitable integration | Prebuilt connector |
| Reusable internal API integration | Custom connector |
| API operations exposed as agent tools | REST API tool |
| Explicit web request in a workflow | HTTP |
| Existing discoverable MCP capabilities | MCP |
| UI-only legacy application | Computer use |
| Modular specialization inside one solution | Child agent |
| Independent reusable agent | Connected agent |
| Cross-platform agent delegation | A2A |
| Existing enterprise vector/search index | Azure AI Search |
| Supported specialized model for a prompt | Foundry model connection |
| Verify expected resource selection | Tool-use evaluation |
| Equivalent meaning, different wording | Compare meaning |
| Exact output string | Exact match |
| Runtime telemetry | Application Insights |
| Deployment-specific configuration | Environment variable |
| Connection binding in a target environment | Connection reference |
| Controlled environment promotion | Solution + pipeline |

---

## 18. Final revision checklist

You should be able to explain each item **without looking at the notes**.

### Planning

- [ ] Knowledge vs tool vs agent delegation.
- [ ] Internal vs external audience.
- [ ] Agent authentication vs downstream tool authentication.
- [ ] Delegated vs maker-provided vs supported application identity.
- [ ] Why instructions are not authorization.
- [ ] Channel and governance considerations.

### Configuration

- [ ] Agent-level vs topic/node-level invocation.
- [ ] Why topics are more controlled—not guaranteed.
- [ ] Topic vs agent flow.
- [ ] Variable scopes and type mapping.
- [ ] Power Fx use cases.
- [ ] Adaptive Card input/output mapping.
- [ ] Generative answers vs custom prompt.

### Integration

- [ ] Copilot connectors vs real-time knowledge connectors.
- [ ] Prebuilt connector vs custom connector.
- [ ] REST API tool vs HTTP request.
- [ ] MCP vs A2A.
- [ ] Child vs connected agents.
- [ ] Foundry agent vs Foundry model.
- [ ] Fabric data-agent purpose.
- [ ] Azure AI Search, vectorization, and citations.
- [ ] Computer-use credentials, restrictions, and monitoring.
- [ ] Agent-flow 100-second response requirement.

### Testing and ALM

- [ ] Select the right evaluation method.
- [ ] Diagnose routing vs execution vs grounding failures.
- [ ] Distinguish Invalid evaluation configuration from agent failure.
- [ ] Evaluation vs production telemetry.
- [ ] Managed vs unmanaged solutions.
- [ ] Environment variables vs connection references.
- [ ] Non-secret vs secret republish behavior.
- [ ] Pipeline deployment and extension.

---

## Final exam habit

Identify the following first:

1. Actual business requirement.
2. Identity and permission constraints.
3. Lifecycle and reuse requirements.
4. Required level of workflow control.
5. Supported integration options.

Then choose the **simplest supported option that satisfies all of them**—not merely the option that sounds most advanced.
