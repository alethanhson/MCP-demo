# The guide to MCP I never had.

AI agents are finally stepping beyond chat. They are solving multi-step problems, coordinating workflows and operating autonomously. And behind many of these breakthroughs is MCP.

MCP is going viral. But if you are overwhelmed by the jargon, you’re not alone.

Today, we will explore why existing AI tools fall short and how MCP solves that problem. We will cover the core components, why they matter, 3-layer architecture and its limitations. You will also find practical examples with use cases.

Just the guide I wish I had when I started.

---

### What is covered?

In a nutshell, we are covering these topics in detail.

*   The problem of existing AI tools.
*   Introduction to MCP and its core components.
*   How does MCP work under the hood?
*   The problem MCP solves and why it even matters.
*   The 3 Layers of MCP (and how I finally understood them).
*   The easiest way to connect 100+ managed MCP servers with built-in Auth.
*   Six practical examples with demos.
*   Some limitations of MCP.

We will be covering a lot so let's get started.

---

## 1. The problem of existing AI tools.

If you’ve ever tried building an AI agent that actually does stuff like checking emails or sending Slack messages (based on your workflow), you know the pain: `the process is messy and most of the time the output is not worth it`.

Yes, we have got amazing APIs. Yes, tools exist. But practical usage and reliability aren't that much.

Even tools like Cursor (which got ultra hyped on Twitter) are getting recent complaints about poor performance.

### 1. Too many APIs, not nearly enough context

Every tool you want the AI to use is basically a mini API integration. So imagine a user says: "Did Anmol email me about yesterday's meet report?"

For an LLM to answer, it has to:

*   Realize this is an email search task, not a Slack or Notion query.
*   Pick the correct endpoint let's say `search_email_messages`
*   Parse and summarize the results in natural language

All while staying within the context window. That’s a lot. Models often forget, guess or hallucinate their way through it.

And if you cannot verify the accuracy, you don't even realize the problem.

### 2. APIs are step-based but LLMs aren’t good at remembering steps.

Let's take a basic example of CRM.

*   First, you get the contact ID → `get_contact_id`
*   Then, fetch their current data → `read_contact`
*   Finally, patch the update → `patch_contact`

In traditional code, you can abstract this into a function and be done. But with LLMs? Each step is a chance to mess up due to a wrong parameter, missed field or broken chain. And suddenly your “AI assistant” is just apologizing in natural language instead of updating anything.

### 3. Fragile tower of prompt engineering

APIs evolve. Docs change. Auth flows get updated. You might wake up one morning to find that your perfectly working agent now breaks due to third-party changes.

And unlike traditional apps, there’s no shared framework or abstraction to fall back on. Every AI tool integration is a fragile tower of prompt engineering, JSON crafting. It risks breaking your AI agent's "muscle memory."

### 4. Vendor lock-in.

Built your tools for GPT-4? Cool. But you will need to rewrite all your function descriptions and system prompts from scratch if you ever switch to other tools like Claude or Gemini.

It's not such a big issue but there is no such universal solution.

There has to be a way for tools and models to communicate cleanly, without stuffing all the logic into bloated prompts. That's where MCP comes in.

---

### 2. Introduction to MCP and its core components.

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) is a new open protocol that standardizes how applications provide context and tools to LLMs.

Think of it as a universal connector for AI. MCP works as a plugin system for Cursor which allows you to extend the Agent’s capabilities by connecting it to various data sources and tools.

![mcp server](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmdh8ztjauqv05yzrk0gj.png)
*credit goes to Greg Isenburg on YouTube*

MCP helps you build agents and complex workflows on top of LLMs.

For example, an MCP server for Obsidian helps AI assistants search and read notes from your Obsidian vault.

Your AI agent can now:

→ Send emails through Gmail
→ Create tasks in Linear
→ Search documents in Notion
→ Post messages in Slack
→ Update records in Salesforce

All by sending natural-language instructions through a standardized interface.

Think about what this means for productivity. Tasks that once required switching between 5+ apps can now happen in a single conversation with your agent.

At its core, MCP follows a client-server architecture where a host application can connect to multiple servers.

![mcp server](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4qblsimyt39tbg619b84.png)
*Credit goes to ByteByteGo*

### Core Components.

Here are the core components in any general MCP Server.

*   `MCP hosts` - apps like Claude Desktop, Cursor, Windsurf or AI tools that want to access data via MCP.
*   `MCP Clients` - protocol clients that maintain 1:1 connections with MCP servers, acting as the communication bridge.
*   `MCP Servers` - lightweight programs that each expose specific capabilities (like reading files, query databases...) through the standardized Model Context Protocol.
*   `Local Data Sources` - files, databases and services on your computer that MCP servers can securely access. For instance, a browser automation MCP server needs access to your browser to work.
*   `Remote Services` - External APIs and cloud-based systems that MCP servers can connect to.

If you're interested in reading the architecture, check out [official docs](https://modelcontextprotocol.io/docs/concepts/architecture). It covers protocol layers, connection lifecycle and error handling with the overall implementation.

We are going to cover everything but if you're interested in reading more about MCP, check out these two blogs:

*   [What is the Model Context Protocol (MCP)?](https://www.builder.io/blog/model-context-protocol) by the Builder.io team
*   [MCP: What It Is and Why It Matters](https://addyo.substack.com/p/mcp-what-it-is-and-why-it-matters) by Addy Osmani

---

## 3. How does MCP work under the hood?

The MCP ecosystem comes down to [several key players](https://www.builder.io/blog/model-context-protocol) that function together. Let's study in brief about them.

![mcp architecture](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn3wsl4nb7x1ratyyusws.png)
*Credit goes to Huggingface*

⚡ Clients.

Clients are the apps you actually use like Cursor, Claude Desktop, and others. Their job is to:

*   Request available capabilities from an MCP server
*   Present those capabilities (tools, resources, prompts) to the AI model
*   Relay the AI's tool usage requests back to the server and return the results
*   Provide models with the basic MCP protocol overview for consistent interaction

They handle communication between the system’s frontends: the user, the AI model, and the MCP server.

⚡ Servers.

MCP servers serve as intermediaries between users/AI and external services. They:

*   Offer a standardized [JSON-RPC](https://www.jsonrpc.org/) interface for tool and resource access
*   Convert existing APIs into MCP-compatible capabilities without requiring API changes
*   Handle authentication, capability definitions and communication standards

They provide context, tools and prompts to clients.

⚡ Service providers.

These are external systems or platforms (like Discord, Notion, Figma) that perform actual tasks. They don’t change their APIs for MCP.

This whole setup allows developers to plug any compatible API into any MCP-aware client, avoiding dependence on centralized integrations by large AI providers.

### MCP Building Blocks: Tools, Resources and Prompts

⚡ [Tools](https://modelcontextprotocol.io/docs/concepts/tools).

Tools represent actions an AI can perform such as `search_emails` or `create_issue_linear`. They form the foundation of how models execute real-world functions through MCP.

⚡ [Resources](https://modelcontextprotocol.io/docs/concepts/resources).

Resources represent any kind of data that an MCP server wants to make available to clients. This can include:

*   File contents
*   Database records
*   API responses
*   Live system data
*   Screenshots and images
*   Log files
*   And more

Each resource is identified by a unique URI (like `file://user/prefs.json`) which can be project notes, coding preferences or anything specific to you. It contains either text or binary data.

Resources are identified using URIs that follow this format.

```
[protocol]://[host]/[path]
```

For example:

*   `file:///home/user/documents/report.pdf`
*   `postgres://database/customers/schema`
*   `screen://localhost/display1`

Servers can also define their own custom URI schemes. You can read more on [official docs](https://modelcontextprotocol.io/docs/concepts/resources).

⚡ [Prompts](https://modelcontextprotocol.io/docs/concepts/prompts).

Tools let the AI do stuff, but prompts guide the AI on how to behave while doing it.

It's like instructions to the model during tool usage. They act like operational guides helping the AI follow specific styles, workflows or safety protocols like if it follows a specific safety checklist before hitting that `delete_everything` button.

🎯 Let’s explore a practical scenario:

Imagine a Google Calendar MCP server. The Calendar API is powerful but talkative, every event includes fields for guests, time zones, reminders, attachments and more. If you ask an AI model to `reschedule all my meetings with Alice next week`, it may struggle to filter the relevant data from the noise.

This is where `prompts` and `resources` come in.

An MCP prompt could instruct the model: "When working with calendar events, only modify those with title or participant match. Extract relevant events using the `list-events` tool, copy them into a temporary resource (`Resource B`), apply changes there and use `update-events-from-resource` to sync them back."

This pattern lets the AI focus on clean, editable data in a controlled state (`the resource`), guided by reusable, standardized instructions (`the prompt`) with proper actions (`tool`).

![builder.io notion example](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6pk6nh004srmvyr59shf.png)
*Notion example by Builder.io*

You should also read the notion example on [Builder.io](https://www.builder.io/blog/model-context-protocol), they have covered it under MCP prompts.

When a client connects to an MCP server, it asks for the list of available tools, resources and prompts. Based on the user’s input, the client selects what to show the model. When the model chooses an action, the client executes it via the server and ensures proper authorization with data flow.

---

## 4. The problem MCP solves and why it even matters.

Let's briefly discuss the problems that MCP solves.

⚡ One common protocol = thousands of tools.

This common protocol means one AI can integrate with thousands of tools as long as those tools have an MCP interface, eliminating the need for custom integrations for each new app.

Services describe what they can do ("send Discord message", "create Linear ticket") and how to do it (parameters, auth methods) using a consistent [JSON-RPC](https://www.jsonrpc.org/) format.

⚡ Clear separation of roles: model thinks, tools act.

It creates a clear separation between the AI model (the thinker) and the external tools (the doers). Your fancy AI agent doesn't break every time Slack tweaks its API.

⚡ With MCP, you don’t have to redo all your tool descriptions when swapping GPT for Claude or Gemini. Your tools and logic stay the same.

⚡ MCP supports memory and multi-step workflows meaning your agent can remember things across tasks and chain actions together intelligently.

⚡ It leads to fewer hallucinations. In a general manner, MCP uses clear, structured tool definitions. That helps AI stay grounded and accurate.

### Why does MCP matter?

MCP matters because:

*   It turns the `dream of a universal AI assistant` for developers into a practical reality.
*   The potential to compose these actions into sophisticated workflows (with the AI handling the logic) will lead to a `new era of intelligent automation`.

So MCP makes it way easier for developers to do more with AI.

---

## 5. The 3 Layers of MCP (and how I finally understood them).

This is how I've understood the concept in detail. I will attach a common example which will help you understand it very quickly.

⚡ Model ↔ Context: “Talk to the LLM in a way it understands”

Imagine the Model as the brain of a robot (`LLM`). It can process information but needs clear instructions. Context provides those instructions to work correctly.

For example: if you tell a robot, "Make me a sandwich," that’s too vague. But saying "Use this bread, ham, and cheese to make a sandwich" gives the robot a context to understand and execute the task.

*   Model is the robot (LLM).
*   Context is the specific instructions you give it (ingredients for the sandwich).

⚡ Context ↔ Protocol: “Give the LLM structured memory, tools, state”

Once the robot has instructions (Context), it needs a way to follow them, remember details and use tools. That is done by `Protocol`, it's the system that lets the robot use memory and tools to get the job done.

Let's take the same sandwich example. Giving it a protocol will help it remember the ingredients, know how to handle the knife and more.

*   Context tells the robot what to do.
*   Protocol gives it the tools and memory to do it.

It's the structure for getting things done.

⚡ Protocol ↔ Runtime: “Actually run the AI Agent”

The robot knows what to do (Context) and how to do it (Protocol). Now it needs to actually do it, which is possible using Runtime.

Going back to the sandwich example, the Runtime is the moment it starts executing it. It's like the environment where the task comes to life (like the kitchen).

*   Protocol gives the robot the method to follow.
*   Runtime is the environment where the robot actually works.

Let's take all three layers together and see what comes using the `restaurant version`.

*   The `Model` is the chef. They have the knowledge and skills to make food.
*   The `Context` is the menu. It tells the chef what ingredients are needed and how the meal should look and taste.
*   The `Protocol` is the waiter. The waiter brings the order to the chef, communicates exactly how the dish should be prepared and even remembers if you’re allergic to something.
*   The `Runtime` is the kitchen where the chef actually prepares the meal. It’s the place where all the tools, heat and preparation happen.

Once you understand the core components like servers and clients (covered in the "How MCP works under the hood" section), it all starts to make sense.

Each layer fits together to make the whole system work.

---

## 6. The easiest way to connect 100+ managed MCP servers with built-in Auth.

In this section, we will be exploring the easiest way to connect Cursor with MCP servers.

If you want to explore how to add and use custom MCP servers within the Cursor, read the [official docs](https://docs.cursor.com/context/model-context-protocol).

### Step 1: Prerequisites.

Install Node.js and ensure `npx` is available in your system.

### Step 2: Enable MCP server in Cursor.

You can open the command palette in Cursor with `Ctrl + Shift + P` and search for cursor settings.

![cursor settings](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpyhip49rwsnk7w02bu6o.png)

You will find an MCP option on the sidebar.

![mcp option in sidebar](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frbz0pzrflzwqxor3j4sy.png)

### Step 3: Using a pre-defined MCP server.

We can also create one from scratch but let's use predefined-one for the sake of simplicity.

We will use Composio for the servers since they have built-in auth. You can find the list at [mcp.composio.dev](https://mcp.composio.dev/).

⚡ Built-in Auth comes with support for OAuth, API keys, JWT and Basic Auth. This means you don't have to create your own login system.
⚡ Fully managed servers eliminate the need for complex setups, making it easy to integrate AI agents with 250+ tools like Gmail, Slack, Notion, Linear and more.
⚡ Provides 20,000+ pre-built API actions for quick integration without coding.
⚡ Can operate locally or remotely depending on your configuration needs.
⚡ Better tool-calling accuracy allows AI agents to interact smoothly with integrated apps.
⚡ It's compatible with AI agents which means it can connect AI agents to tools for tasks like sending emails, creating tasks or managing tickets in a single conversation.

It also means less downtime and fewer maintenance problems. You can check the status at [status.composio.dev/](https://status.composio.dev/).

![status composio](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsm17sx2jy7mkn6wb99b0.png)

You can easily integrate with a bunch of useful MCP servers without writing any code.

![composio](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcpkd5wirfdv58bz1ss9x.png)

With each of the options, you will find the total active users, its current version, how recently it was updated and all the available actions.

You will find instructions to install it using `TypeScript`, `Python` and it supports Claude (MacOS), Windsurf (MacOS) and Cursor as MCP hosts.

If you're interested in building an MCP client from scratch, check this [step-by-step guide](https://composio.dev/blog/mcp-client-step-by-step-guide-to-building-from-scratch/) by Harsh on Composio.

![gmail composio](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhssy1izsb2mbegq52b7l.png)

### Step 4: Integrating MCP server.

It's time to integrate one with the cursor. For now, we will be using the Gmail MCP server.

Previously it was with SSE but Cursor recently changed this method with the `npx command`. We will need to generate the terminal command. Check [this page](https://mcp.composio.dev/hackernews/thundering-petite-vulture-guyWBb) to generate yours.

The terminal command will look like this.

```
npx @composio/mcp@latest setup "https://mcp.composio.dev/gmail/xyzxyz..." --client cursor
```

You can run this command in the terminal and restart Cursor to notice the changes.

If you're using Python, here's how you can Install the composio-toolset.

```
pip install composio_openai
```

```python
from composio_openai import ComposioToolSet, App
from openai import OpenAI

openai_client = OpenAI()
composio_toolset = ComposioToolSet(entity_id="default")
tools = composio_toolset.get_tools(apps=[App.GMAIL])
```

You can place the final configuration in two locations, depending on your use case:

1) For tools specific to a project, create a `.cursor/mcp.json` file in your project directory. This allows you to define MCP servers that are only available within that specific project.
2) For tools that you want to use across all projects, create a `~/.cursor/mcp.json` file in your home directory. This makes MCP servers available in all your Cursor workspaces. The terminal would enforce the second option which will make it globally accessible.

![npx command](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F39igz0rqfh98q877jemu.png)

It will display the necessary actions and status green dot which indicates, that it's successfully integrated.

![mcp gmail server](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8wno3z6kp0uxszplfb3k.png)

The `mcp.json` will look like this.

```json
{
  "mcpServers": {
    "gmail_composio": {
      "url": "https://mcp.composio.dev/gmail/freezing-wrong-dress-7RHVw0"
    }
  }
}
```

You can check out the list of [sample servers and implementations](https://modelcontextprotocol.io/examples). You can integrate the community servers by following this structure (based on your choice of preference).

✅ SSE Server Configuration.

This configuration is supported in Cursor and you can specify the `url` field to connect to your SSE server.

```json
// This example demonstrated an MCP server using the SSE format
// The user should manually set and run the server
// This could be networked, to allow others to access it too
{
  "mcpServers": {
    "server-name": {
      "url": "http://localhost:3000/sse",
      "env": {
        "API_KEY": "value"
      }
    }
  }
}
```

✅ STDIO Server Configuration (Python)

This sets up an MCP server using the standard input/output (STDIO) transport with a Python script. This approach is mainly used for local development.

```json
// if you're using CLI server Python
// This example demonstrated an MCP server using the stdio format
// Cursor automatically runs this process for you
// This uses a Python server, run with `python`
{
  "mcpServers": {
    "server-name": {
      "command": "python",
      "args": [
        "mcp-server.py"
      ],
      "env": {
        "API_KEY": "value"
      }
    }
  }
}
```

✅ STDIO Server Configuration (Node.js)

```json
// if you're using CLI server Node.js
// This example demonstrated an MCP server using the stdio format
// Cursor automatically runs this process for you
// This uses a Node.js server, ran with `npx`
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-server"
      ],
      "env": {
        "API_KEY": "value"
      }
    }
  }
}
```

### Step 5: Using the server directly within Agent.

Before proceeding, make sure to check available actions on [composio mcp server page](https://mcp.composio.dev/gmail/freezing-wrong-dress-7RHVw0). You can also find the tools and actions on the dashboard.

![tools](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxuq7ahgz2k87xpwprdtd.png)

You can open the Chat using the `Ctrl + I` command.

You can enable `Agent Mode` which is the most autonomous mode in Cursor, designed to handle complex coding tasks with minimal guidance.

![agent mode](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F63aj5z7bu9u6w2raprfv.png)

I prefer having some control before executing, so I'm going with the default one. You can type any query. You just need to click on the `run tool` button.

As you can see, it will call the appropriate MCP server (if you have multiple of those) and it will accordingly use the correct action based on your prompt.

![run tool option](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1oy7shupeqmuymvpzhiu.png)

Since there is no active connection, it will first establish one. You will need to authorize the process.

![cursor establish connection](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvn5q8u43brlsq2s189yh.png)

![gmail authorize](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwtt2is3b9x4733bzwup0.png)

I'm using a dummy account (that I created a long time ago) and I recommend doing the same for testing purposes. Once you're satisfied, you can automate things with your primary account.

As you can see, it properly fetched the emails.

![fetched emails](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Futh3owpr9lmq7w6v441a.png)

Let's check by sending an email to `hi@anmolbaranwal.com` with the subject "Demo of Composio" and saying testing MCP server in the body of the email.

![output](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7s2ebz8sxhy19q6j9pcg.png)

As you can see I've received that email with the proper subject and body as specified in the prompt.

![sent email](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmthdbkdhjc6mr608iytu.png)
*sent the email*

![testing email](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Farjcxnbzf15itmgzjqoe.png)
*received the email*

With this MCP server, you can do lots of amazing things like `Get attachments`, `Create email draft`, `Modify thread labels`, `Reply to a thread`, `get contacts`, `delete message`, `move to trash`, `search people`, `send email` and much more.

And always remember, there is a limit to what you can do. I've tested with more than 15 prompts to analyze the edge cases.

![conversation too long](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F934iwt6idiv3qr3oxido.png)

---

## 7. Six practical examples with demos.

Here are five practical examples of MCP servers. Let's discuss the flow and see the application.

### ✅ [YouTube MCP Server](https://mcp.composio.dev/youtube/freezing-wrong-dress-7RHVw0)

![youtube mcp server](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3wvh9rm6vopv1y85i46t.png)

We will follow the same flow as discussed before, you can check the Composio server for YouTube where you can generate the url.

![generate url](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fpoldjjcogcal5mp7xvh6.png)

The command will be structured like:

```
npx @composio/mcp@latest setup "https://mcp.composio.dev/youtube/freezing-wrong-dress-xyz" --client cursor
```

If you notice in the `mcp.json`, the url for youtube will be added as soon as you run the terminal command. It will look something like this.

```json
{
  "mcpServers": {
    "youtube_composio": {
      "url": "https://mcp.composio.dev/youtube/freezing-wrong-dress-7RHVw0"
    }
  }
}
```

Since there is no active connection, it will first establish one. You will need to authenticate by copying the OAuth URL in the browser.

![establish connection](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fhh53jfwz2f1uthjamu1f.png)

You will have to provide access to the server so it can take action based on your prompt.

![access](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3rm5m3l03bh1dnxshmmq.png)

Now, you can put any prompt such as `Fetch me the top 5 videos about Model Context Protocol based on views and likes.`

It will accordingly generate a response.

![output](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F98b1nmliukvsljxa7btx.png)

![output](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Flz1pncbs77dprr72bwni.png)

With this MCP server, you can do lots of amazing things like `Search youtube for videos, channels, playlists`, `fetch video stats`, `load captions`, `subscribe channel`, `update video's metadata`, `update thumbnail` and much more.

### ✅ [Ahrefs MCP Server](https://mcp.composio.dev/ahrefs/freezing-wrong-dress-7RHVw0)

We will follow the same flow as discussed before, you can check [Composio server for Ahrefs](https://mcp.composio.dev/ahrefs/freezing-wrong-dress-7RHVw0).

If you're unaware, Ahrefs is an SEO and marketing platform offering site audits, keyword research, content analysis and competitive insights to improve search rankings and drive organic traffic.

![ahrefs](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fb9xli5u9yjrpijpcp6tt.png)

If you notice in the `mcp.json`, the ahrefs url will be added.

```json
"ahrefs_composio": {
  "url": "https://mcp.composio.dev/ahrefs/freezing-wrong-xyz"
}
```

As you can see, it's establishing a connection for the first time.

![permission required for establish connection](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F1r140mrfwakf9fcnm5sn.png)

Once you do that, you will be able to use all the actions like `retrieve organic keywords`, `fetch all backlinks`, `domain rating history`, `pages by traffic overview`, `retrieve public crawler ips`, `fetch competitors overview`, `list best by external links`, `fetch total search volume history` and much more.

Please note that you will need an API (which comes under the premium plan for Ahrefs) to complete the integration.

### ✅ [LinkedIn MCP server](https://mcp.composio.dev/linkedin/old-damaged-thailand-X5G8GA)

You can follow the same process to generate the URL and run it in the terminal. You will then need to establish a connection and authenticate by copying the OAuth URL in the browser.

You will get a confirmation message once it's done.

![confirmation message](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3acmx5eq6mui7wjtzdvo.png)

You can also check that based on the actions of the server. As you can see, there is an active connection.

![active connection](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3beakqf1gb8h7h2n7dk8.png)

I normally wouldn't recommend using it on your official accounts because you can never be too careful.

With this MCP server, you get the options like `get the info of profile`, `create post`, `get company info` and `delete a post`.

### ✅ [Autonomously reverse engineer apps using Ghidra MCP Server](https://github.com/lauriewired/ghidramcp).

This MCP server allows LLMs to autonomously reverse engineer applications. It exposes numerous tools from core [Ghidra](https://ghidra-sre.org/) functionality to MCP clients.

This includes decompiling and analyzing binaries, automatically renaming methods and data, and listing methods, classes, imports and exports.

A couple of use cases:

⚡ Automated vulnerability analysis using LLMs.
⚡ Reverse engineering of malware samples.

Here is the demo.

[![Ghidra MCP](https://img.youtube.com/vi/uH7Apjh-P_A/0.jpg)](https://www.youtube.com/watch?v=uH7Apjh-P_A)

The [GitHub Repository](https://github.com/lauriewired/ghidramcp) has 4k stars.

### ✅ [Read and modify Figma designs programmatically](https://github.com/sonnylazuardi/cursor-talk-to-figma-mcp)

There have been recent developments in generating the Figma board into production-ready apps.

This project implements the same thing using MCP integration between Cursor AI and Figma, allowing Cursor to communicate with Figma to read designs and modify them programmatically.

It allows for document & selection, annotations, creating elements, styling, layout and much more.

You can simply say: `design a modern-looking signup screen for mobile` and it will create it without you interacting with the Figma file.

Here is the demo.

[![Cursor talk to figma MCP](https://img.youtube.com/vi/71_6kqzK8Ag/0.jpg)](https://www.youtube.com/watch?v=71_6kqzK8Ag)

It has 3.1k stars on GitHub.

You can check the [GitHub Repository](https://github.com/sonnylazuardi/cursor-talk-to-figma-mcp) and [official Tweet](https://x.com/sonnylazuardi/status/1901325190388428999).

There is another great [MCP server](https://github.com/GLips/Figma-Context-MCP) (with 5k stars on GitHub which provides Figma layout information to AI coding agents.

### ✅ [Create 3D scenes using Blender MCP](https://github.com/ahujasid/blender-mcp)

Creating 3d stuff has always scared a lot of builders due to the complexities involved.

This connects Blender to Claude AI through the Model Context Protocol (MCP), allowing Claude to directly interact with and control Blender.

This integration enables prompt assisted 3D modeling, scene creation and manipulation. You can watch the [complete tutorial](https://www.youtube.com/watch?v=lCyQ717DuzQ) if you're interested in using it.

Prompt examples with demo videos:

⚡ "Create a low poly scene in a dungeon, with a dragon guarding a pot of gold"

[![Blender MCP Demo: AI Prompting a dragon in a dungeon](https://img.youtube.com/vi/DqgKuLYUv00/0.jpg)](https://www.youtube.com/watch?v=DqgKuLYUv00)

⚡ "Get information about the current scene, and make a threejs sketch from it"

[![Blender MCP Demo: Creating a Threejs scene from Blender](https://img.youtube.com/vi/jxbNI5L7AH8/0.jpg)](https://www.youtube.com/watch?v=jxbNI5L7AH8)

⚡ "Create a beach vibe using HDRIs, textures, and models like rocks and vegetation from Poly Haven"

[![Blender MCP: Use Polyhaven assets](https://img.youtube.com/vi/I29rn92gkC4/0.jpg)](https://www.youtube.com/watch?v=I29rn92gkC4)

If you're interested in more demos, the founder created a [thread on X with wild examples](https://x.com/sidahuj/status/1909986466723168344) of what others have created.

The [GitHub Repository](https://github.com/ahujasid/blender-mcp) has 10.3k stars on GitHub.

---

## 8. Some limitations of MCP.

MCP expectations and reality can be very different. You will understand what I mean as you go through the points.

![mcp expectations vs mcp reality](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5nfjuqt0h5vrvmpn7m9m.png)
*Credit goes to Builder.io team*

Don't get me wrong, MCP is very promising but these are some limitations you should be aware of:

⚡ Not all AI platforms support MCP.

Claude (especially with its desktop app) and tools like Cursor or Windsurf support MCP directly. But if you're using something like ChatGPT or a local LLaMA model, it might not work out of the box.

There are some open source tools trying to solve this, but until MCP becomes more widely adopted, support across all AI assistants is hard.

⚡ Agent autonomy is not perfect.

MCP gives the ability but the judgment by AI is still not perfect.

For example, tool use depends on how well the model understands tool descriptions and usage context. It often needs prompt tuning or agent-side logic to improve reliability.

⚡ Performance Overhead.

Using tools through MCP adds overhead. Each call is external and can be much slower than the AI just answering on its own. For example, scraping data from a webpage through an MCP tool might take seconds, while the model could have guessed the answer from training data in milliseconds.

Now, if you are orchestrating multiple tools, the latencies add up, like calling 5 different MCP servers in sequence to:

*   Fetch a file from Google Drive
*   Summarize the content using an LLM tool
*   Translate the summary
*   Generate a tweet based on the translation
*   Schedule it using a social media tool like Buffer

That chain might take 10–15 seconds, depending on server response times.

Some agents can handle parallel tool use so you can further optimize the process.

⚡ The trust issue.

Letting AI perform real actions can feel risky. Even if the AI usually gets it right, users often want to review things before they happen.

Right now, most tools are either fully autonomous or not at all. There's rarely a middle ground where AI can leverage autonomy but still give control to the user when it matters. We all need a `human in the loop`.

❌ Bad approach: The AI sends an email instantly without asking.
✅ Better approach: The AI says, `I'm about to email X with this message, is it okay to send?` and only acts after you approve.

⚡ The problem of scalability.

Most MCP servers today are built for single users, often just running on a developer’s laptop.

One MCP server serving multiple independent agents or users has not been much explored yet. To do that, companies need to handle more complex stuff like concurrent requests, separate data contexts and enforce rate limit usage.

This is an area where the ecosystem still has room to grow, especially with ideas like MCP gateways or enterprise-ready MCP server frameworks.

⚡ Security standards.

MCP doesn’t come with built-in authentication or authorization.

`Authentication & Authorization`: MCP doesn’t have built-in support for authenticating users or agents. If you expose an MCP server over a network, you have to add your own security.

Some implementations use OAuth 2.1 to add permission scoping (`read-only or write-only access`), but there's currently no standard approach, so each server handles auth differently.

`Correct Permissions`: Ideally, agents should only use the tools they need. But if multiple powerful tools are available (like browser access and terminal), nothing stops the AI from using the wrong one, unless you manually disable it.

`Prompt Injection`: AI can make mistakes if it misunderstands a prompt. Worse, someone could craft a malicious prompt to trick the AI into doing something harmful (`prompt injection`). The safeguards depend on how each MCP server is built.

If you want to understand how to mitigate security risks in MCP implementations, read this:

*   [Understanding and mitigating security risks in MCP implementations](https://techcommunity.microsoft.com/blog/microsoft-security-blog/understanding-and-mitigating-security-risks-in-mcp-implementations/4404667) by Microsoft.
*   [The Security Risks of MCP](https://www.pillar.security/blog/the-security-risks-of-model-context-protocol-mcp) by Pillar.

MCP is still new. There will be further developments to resolve more edge cases as needs are discovered.

On the AI model side, we will likely see models that are fine-tuned for tool use and MCP specifically. Anthropic already mentioned future `AI models optimized for MCP interaction`.

---

Here are some nice resources if you're planning to build MCP:

*   [mcp-chat](https://github.com/Flux159/mcp-chat) - is a CLI chat client for MCP servers. Used for testing & evaluating MCP servers and agents
*   [mastra registry](https://mastra.ai/mcp-registry-registry) - a collection of MCP server directories to connect AI to your favorite tools.
*   [smithery.ai](https://smithery.ai/) - extend your agent with 4,630 capabilities via MCP servers. A lot of details include `monthly tool calls`, `local option`, `tools`, `API`, `installation instructions for different clients`.
*   [Popular MCP Servers directory by official team](https://github.com/modelcontextprotocol/servers) - 20k stars on GitHub.
*   [Cursor directory](https://cursor.directory/mcp) of 1800+ MCP servers.
*   [Those MCP totally 10x my Cursor workflow](https://www.youtube.com/watch?v=oAoigBWLZgE) - YouTube video with practical use cases.

---

MCP is still evolving but its core ideas are here to stay and I've tried my best explaining the concepts. I hope you found something useful.

One single conversation with an Agent can help you automate complex workflows.

Now go build something wild with MCP and show the world.

Have a great day! Until next time :)

