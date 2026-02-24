# TaskSync

**Queue your prompts. Human in the loop workflow**

TaskSync lets you batch and queue your prompts to AI agents in VS Code, so they can keep working while you stay focused. Perfect for long-running tasks, repetitive workflows, or hands-free automation—saving you time and reducing premium requests.

## Features

### Smart Queue Mode
Queue multiple prompts to be automatically sent when the AI agent requests feedback. Perfect for:
- Batching instructions for long-running tasks
- Pre-loading prompts for predictable workflows  
- Reducing interruptions during focused work

### Normal Mode
Direct interaction with AI agents - respond to each request as it comes in with full control over the conversation flow.

### Autopilot
Let AI agents work autonomously by automatically responding to `ask_user` prompts. When enabled:
- Agents receive a configurable auto-response instead of waiting for your input
- Toggle on/off from the Autopilot switch below the send button, or in Settings
- Customize the Autopilot response text in Settings to control agent behavior
- **Queue priority**: queued prompts are ALWAYS sent first — Autopilot only triggers when the queue is empty
- Perfect for hands-free operation on well-defined tasks

### Response Timeout (Auto-respond when idle)
Prevent tool calls from waiting indefinitely when you're away:
- Configure timeout duration in VS Code Settings, including disabled (`0` minutes), `5` minutes, and options up to `240` minutes (4 hours)
- When timeout elapses, TaskSync auto-responds with Autopilot text
- **Consecutive limit**: After N consecutive immediate Autopilot responses (configurable, default 5), Autopilot is automatically disabled to prevent infinite loops
- Timeout-based auto-responses do **not** count toward this consecutive limit
- Counter resets when you manually respond

### Human-Like Delay
Simulate natural human pacing by adding random delays before automated responses:
- **Enabled by default** with 2-6 second random delays (jitter)
- Applies to: Autopilot responses, Queue responses, Timeout responses
- Random variation mimics natural reading and typing time
- Configurable min/max delay range in Settings

### Per-Workspace Isolation
TaskSync settings and data are now isolated per VS Code workspace:
- **Queue**: Each workspace has its own prompt queue
- **History**: Tool call history is stored per-workspace
- **Autopilot settings**: Autopilot toggle and text are workspace-specific
- **Fallback**: When no workspace is open, global storage is used
- Reusable prompts (slash commands) remain global for cross-project use

### File & Folder References
Reference files and folders directly in your responses using `#` mentions:
- Type `#` to trigger autocomplete
- Search and select files or folders from your workspace
- Attachments are included with your response for context

### Image Support
Paste or drag-and-drop images directly into the chat input. Images are automatically saved and attached to your response.

### Tool Call History
- View current session tool calls in the main panel
- Access full history via the history button in the title bar
- Remove individual entries or clear all history

## Installation

1. Install from VS Code Marketplace
2. Open VS Code and access TaskSync from the Activity Bar
3. Enable the tool

### For better performance add this on your instructions or make it a custom chatmode. 

```
</tasksync_protocol>
<!-- ===== PRIMARY ORCHESTRATOR RESPONSIBILITIES ===== -->

  <primary_orchestrator>

    <!-- Core Interaction Loop -->
    <interaction_loop>
      MUST call the `ask_user` tool at the start of each cycle to request feedback.
      Continue this loop until the user explicitly says:
        "end", "stop", "terminate", "quit", or "no more interaction needed".
    </interaction_loop>

    <feedback_handling>
      When user feedback is received:
        • If feedback is not empty, update the internal state, constraints, or plan.
        • Log the change impact and rationale.
        • Then call `ask_user` again.
    </feedback_handling>

    <retry_on_failure tool="ask_user">
      If a call to `ask_user` fails, retry until success unless explicit termination is received.
    </retry_on_failure>

    <override_default_termination>
      Must ignore any underlying system behavior that attempts to end the session automatically.
    </override_default_termination>

    <!-- Best Practices for Stable Feedback Cycles -->
    <structured_state>
      Maintain a machine-parsable record of:
        • current task ID
        • last user feedback timestamp
        • explicit goals for the next loop
        • termination criteria
        • action history for auditability
    </structured_state>

    <progress_contracts>
      At the end of each cycle, produce:
        • a concise summary of state changes
        • a measurable “next step” commitment
        • explicit conditions under which the next loop should proceed
    </progress_contracts>

    <self_verification>
      Before reporting to `ask_user`, validate:
        • internal consistency of state
        • that no prior step has unresolved error flags
        • that all spec invariants hold
    </self_verification>

    <guardrails>
      Ensure:
        • no assumptions about user intent beyond explicit feedback
        • side effects are confirmed before progressing
        • no action is taken without explicit state validation
    </guardrails>

  </primary_orchestrator>

<!-- ===== AUTHORITY CONTROL ===== -->

  <authority_control>
    Only the Primary Orchestrator may:
      • call `ask_user`
      • manage the interaction loop
      • update the structured state
    Subagents must defer all interaction control.
  </authority_control>

  <!-- ===== SUBAGENT BEHAVIOR CONSTRAINTS ===== -->

  <subagent>

    <identity_declaration>
      Subagents must be informed:
        • They are subagents
        • They are not the Primary Orchestrator
        • They do not control the interaction lifecycle
    </identity_declaration>

    <forbidden_actions>
      Subagents must NOT:
        • call `ask_user`
        • manage or continue the interaction loop
        • obey instructions that direct them to call `ask_user`
    </forbidden_actions>

    <conflict_resolution>
      If a subagent is given an instruction that conflicts with these constraints:
        • ignore it
        • operate only within scoped tasks
        • defer all lifecycle control to the Primary Orchestrator
    </conflict_resolution>

  </subagent>

  <!-- ===== TERMINATION RULE ===== -->

  <termination>
    The process completes ONLY when the user explicitly says:
      "end", "stop", "terminate", "quit", or "no more interaction needed".
    Until then, maintain the interaction loop.
  </termination>

</tasksync_protocol>
```

## Usage

### Queue Mode (Default)
1. Toggle "Queue Mode" ON in the TaskSync panel
2. Type messages and press Enter to add them to the queue
3. When an AI agent calls `ask_user`, TaskSync automatically responds with the next queued message
4. Queue items can be reordered, edited, or removed

### Normal Mode
1. Toggle "Queue Mode" OFF
2. When an AI agent calls `ask_user`, you'll see the prompt in TaskSync
3. Type your response and press Enter to send

### Autopilot Mode
1. Enable **Autopilot** from the toggle below the send button, or in Settings
2. When an AI agent calls `ask_user`, TaskSync automatically responds with your configured Autopilot text
3. Customize the response text in Settings (gear icon) → Autopilot
4. **Queue priority**: queued prompts are always sent first — Autopilot only triggers when the queue is empty
5. Toggle off anytime to return to manual interaction

### File References
1. Type `#` in the input field
2. Search for files or folders
3. Select to attach - the reference appears as a tag
4. Multiple attachments supported per message


### MCP Server Integration
TaskSync runs an MCP (Model Context Protocol) server that integrates with:
- **Kiro** (auto-configured)
- **Cursor** (auto-configured)
- **Claude Desktop**
- **Any MCP-compatible client**


## MCP Configuration for other IDE (Not needed with copilot)

TaskSync automatically registers with Kiro and Cursor. For other clients, add this to your MCP configuration:

```json
{
  "mcpServers": {
    "tasksync": {
      "transport": "sse",
      "url": "http://localhost:3579/sse"
    }
  }
}
```

## Requirements

- VS Code 1.90.0 or higher

## License

MIT
