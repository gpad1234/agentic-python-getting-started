# API Reference Documentation

## Overview

This document provides comprehensive API reference for the OSQuery MCP Server with LangChain/LangGraph integration. The platform provides multiple API interfaces for different integration patterns.

## Table of Contents

- [MCP Server API](#mcp-server-api)
- [LangChain Agent API](#langchain-agent-api)  
- [LangGraph Workflow API](#langgraph-workflow-api)
- [Security API](#security-api)
- [Web Interface API](#web-interface-api)
- [REST API](#rest-api)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)

## MCP Server API

### Protocol Specification

The MCP Server implements the Model Context Protocol over JSON-RPC 2.0 via STDIO.

#### List Tools

**Method**: `tools/list`

**Description**: Retrieve all available OSQuery tools.

**Request**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}
```

**Response**:
```json
{
  "jsonrpc": "2.0", 
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "system_info",
        "description": "Get system information including OS, hardware, and configuration",
        "inputSchema": {
          "type": "object",
          "properties": {},
          "additionalProperties": false
        }
      },
      {
        "name": "process_analysis", 
        "description": "Analyze running processes with filtering and sorting options",
        "inputSchema": {
          "type": "object",
          "properties": {
            "filter": {"type": "string", "description": "Process filter criteria"},
            "limit": {"type": "integer", "description": "Maximum number of processes"}
          },
          "additionalProperties": false
        }
      },
      {
        "name": "network_analysis",
        "description": "Analyze network connections and interface statistics", 
        "inputSchema": {
          "type": "object",
          "properties": {
            "interface": {"type": "string", "description": "Network interface name"}
          },
          "additionalProperties": false
        }
      },
      {
        "name": "security_scan",
        "description": "Perform security analysis of system components",
        "inputSchema": {
          "type": "object", 
          "properties": {
            "scan_type": {"type": "string", "enum": ["quick", "full", "custom"]}
          },
          "additionalProperties": false
        }
      },
      {
        "name": "file_analysis",
        "description": "Analyze file system changes and permissions",
        "inputSchema": {
          "type": "object",
          "properties": {
            "path": {"type": "string", "description": "File or directory path"},
            "recursive": {"type": "boolean", "description": "Recursive analysis"}
          },
          "additionalProperties": false
        }
      }
    ]
  }
}
```

#### Call Tool

**Method**: `tools/call`

**Description**: Execute a specific OSQuery tool.

**Request**:
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "system_info",
    "arguments": {}
  }
}
```

**Response**:
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\n  \"hostname\": \"my-server\",\n  \"cpu_brand\": \"Intel Core i7\",\n  \"physical_memory\": \"16777216000\",\n  \"platform\": \"ubuntu\",\n  \"platform_version\": \"20.04\",\n  \"uptime\": \"142536\"\n}"
      }
    ],
    "isError": false
  }
}
```

#### Tool Examples

**System Information**:
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call", 
  "params": {
    "name": "system_info",
    "arguments": {}
  }
}
```

**Process Analysis with Filter**:
```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/call",
  "params": {
    "name": "process_analysis",
    "arguments": {
      "filter": "cpu_time > 1000",
      "limit": 10
    }
  }
}
```

**Security Scan**:
```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "tools/call",
  "params": {
    "name": "security_scan", 
    "arguments": {
      "scan_type": "full"
    }
  }
}
```

### Error Responses

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32602,
    "message": "Invalid params",
    "data": {
      "tool": "process_analysis",
      "error": "Invalid filter syntax"
    }
  }
}
```

## LangChain Agent API

### OSQueryAgent Class

The main agent class for LangChain-based orchestration.

#### Constructor

```python
from examples.langchain_agent import OSQueryAgent

agent = OSQueryAgent(
    api_key: str = None,           # OpenAI API key (optional)
    model: str = "gpt-4",          # LLM model to use
    temperature: float = 0.1,      # Response creativity (0.0-1.0)
    max_tokens: int = 2000,        # Maximum response tokens
    timeout: int = 30              # Request timeout in seconds
)
```

#### Methods

##### `analyze_scenario(scenario: str) -> dict`

**Description**: Analyze a security or operational scenario using AI.

**Parameters**:
- `scenario` (str): Natural language description of the scenario

**Returns**: 
- `dict`: Analysis results with findings and recommendations

**Example**:
```python
import asyncio
from examples.langchain_agent import OSQueryAgent

async def analyze_security():
    agent = OSQueryAgent(api_key="your-openai-key")
    
    result = await agent.analyze_scenario(
        "suspicious network activity from unknown IP addresses"
    )
    
    return result

# Example response
{
    "analysis": "Detected unusual network connections...",
    "findings": [
        {
            "type": "network_anomaly",
            "severity": "high", 
            "description": "Multiple connections to suspicious IPs",
            "evidence": {...}
        }
    ],
    "recommendations": [
        "Block suspicious IP addresses",
        "Investigate affected processes",
        "Review network logs"
    ],
    "confidence": 0.85
}
```

##### `execute_query(query: str) -> dict`

**Description**: Execute a specific OSQuery or natural language query.

**Parameters**:
- `query` (str): OSQuery SQL or natural language query

**Returns**:
- `dict`: Query results with metadata

**Example**:
```python
# SQL query
result = await agent.execute_query(
    "SELECT name, pid, cpu_time FROM processes ORDER BY cpu_time DESC LIMIT 5"
)

# Natural language query  
result = await agent.execute_query(
    "show me the top 5 processes using the most CPU"
)

# Example response
{
    "query": "SELECT name, pid, cpu_time FROM processes...",
    "results": [
        {"name": "chrome", "pid": 1234, "cpu_time": 15000},
        {"name": "python", "pid": 5678, "cpu_time": 8500}
    ],
    "metadata": {
        "execution_time": 0.25,
        "row_count": 5,
        "columns": ["name", "pid", "cpu_time"]
    }
}
```

##### `get_recommendations(context: dict) -> list`

**Description**: Get AI-generated recommendations based on context.

**Parameters**:
- `context` (dict): Context information for recommendations

**Returns**:
- `list`: List of recommended actions

**Example**:
```python
recommendations = await agent.get_recommendations({
    "type": "security",
    "findings": ["unusual_processes", "network_anomalies"],
    "severity": "high"
})

# Example response
[
    {
        "action": "isolate_suspicious_processes",
        "priority": "high",
        "description": "Terminate and investigate suspicious processes",
        "commands": ["sudo kill -9 1234", "sudo netstat -p | grep 1234"]
    },
    {
        "action": "block_network_traffic", 
        "priority": "medium",
        "description": "Block traffic to suspicious IP addresses",
        "commands": ["sudo iptables -A OUTPUT -d 192.168.1.100 -j DROP"]
    }
]
```

##### `get_tool_suggestions(query: str) -> list`

**Description**: Get suggested tools for a given query.

**Parameters**: 
- `query` (str): Natural language query

**Returns**:
- `list`: Suggested tool names with relevance scores

**Example**:
```python
suggestions = await agent.get_tool_suggestions(
    "check for malware on the system"
)

# Example response
[
    {"tool": "security_scan", "relevance": 0.95},
    {"tool": "process_analysis", "relevance": 0.78}, 
    {"tool": "file_analysis", "relevance": 0.65}
]
```

### Mock Agent

For environments without LangChain/OpenAI access, a mock agent is available.

```python
from examples.langchain_agent import MockAgent

agent = MockAgent()

# Same interface as OSQueryAgent
result = await agent.analyze_scenario("security check")
```

## LangGraph Workflow API

### Workflow Creation

#### `create_osquery_workflow() -> CompiledGraph`

**Description**: Create a compiled LangGraph workflow for OSQuery analysis.

**Returns**: 
- `CompiledGraph`: Compiled workflow ready for execution

**Example**:
```python
from examples.langgraph_example import create_osquery_workflow

workflow = create_osquery_workflow()
```

### WorkflowState Schema

```python
from typing import TypedDict, Dict, Any

class WorkflowState(TypedDict):
    """Workflow state schema"""
    query: str                    # Original query
    analysis_type: str           # Type of analysis
    results: Dict[str, Any]      # Accumulated results  
    current_step: str            # Current workflow step
    error_count: int            # Error tracking
    metadata: Dict[str, Any]    # Additional context
```

### Workflow Execution

#### Synchronous Execution

```python
# Basic execution
initial_state = {
    "query": "analyze system security",
    "analysis_type": "security", 
    "results": {},
    "current_step": "start",
    "error_count": 0,
    "metadata": {"start_time": time.time()}
}

final_state = workflow.invoke(initial_state)

print(f"Workflow completed: {final_state['current_step']}")
print(f"Results: {final_state['results']}")
```

#### Asynchronous Execution

```python
import asyncio

async def run_workflow():
    initial_state = {
        "query": "check running processes",
        "analysis_type": "process",
        "results": {},
        "current_step": "start"
    }
    
    final_state = await workflow.ainvoke(initial_state)
    return final_state

result = asyncio.run(run_workflow())
```

#### Streaming Execution

```python
# Stream workflow steps for real-time monitoring
for step in workflow.stream(initial_state):
    print(f"Current step: {step['current_step']}")
    print(f"Progress: {len(step['results'])} analyses complete")
    
    # Handle different steps
    if step['current_step'] == 'security_analysis_complete':
        print(f"Security findings: {step['results']['security_analysis']}")
    elif step['current_step'] == 'complete':
        print("Workflow finished!")
        break
```

### Workflow Builder API

#### WorkflowBuilder Class

```python
from web_interface.workflow_builder import WorkflowBuilder

builder = WorkflowBuilder()
```

##### `add_node(name: str, node_type: str, config: dict) -> None`

**Description**: Add a node to the workflow.

**Parameters**:
- `name` (str): Unique node identifier
- `node_type` (str): Type of node (start, tool, condition, end)
- `config` (dict): Node configuration

**Example**:
```python
builder.add_node("security_check", "tool", {
    "tool_name": "security_scan",
    "parameters": {"scan_type": "full"}
})
```

##### `add_edge(from_node: str, to_node: str, condition: str = "always") -> None`

**Description**: Add an edge between nodes.

**Parameters**:
- `from_node` (str): Source node name
- `to_node` (str): Target node name  
- `condition` (str): Edge condition (always, on_success, on_error)

**Example**:
```python
builder.add_edge("start", "security_check", "always")
builder.add_edge("security_check", "process_analysis", "on_success")
```

##### `validate_workflow() -> dict`

**Description**: Validate workflow structure and configuration.

**Returns**:
- `dict`: Validation results with errors and warnings

**Example**:
```python
validation = builder.validate_workflow()

# Example response
{
    "valid": True,
    "errors": [],
    "warnings": [
        "Node 'end_node' has no incoming edges"
    ],
    "suggestions": [
        "Consider adding error handling paths"
    ]
}
```

##### `export_workflow() -> dict`

**Description**: Export workflow definition for persistence or sharing.

**Returns**:
- `dict`: Complete workflow definition

**Example**:
```python
workflow_def = builder.export_workflow()

# Example response
{
    "version": "1.0",
    "nodes": [
        {
            "id": "start",
            "type": "start", 
            "position": {"x": 100, "y": 100}
        },
        {
            "id": "security_check",
            "type": "tool",
            "config": {"tool_name": "security_scan"},
            "position": {"x": 300, "y": 100}
        }
    ],
    "edges": [
        {
            "from": "start",
            "to": "security_check",
            "condition": "always"
        }
    ],
    "metadata": {
        "created": "2025-11-10T12:00:00Z",
        "author": "user@example.com"
    }
}
```

### Node Types

#### System Analyzer Node

**Purpose**: Initial system analysis and routing decisions

**Configuration**:
```python
{
    "type": "system_analyzer",
    "config": {
        "analyze_hardware": True,
        "analyze_os": True,
        "analyze_network": False
    }
}
```

#### Security Analyzer Node

**Purpose**: Security-focused analysis with threat detection

**Configuration**:
```python
{
    "type": "security_analyzer", 
    "config": {
        "scan_depth": "full",
        "include_network": True,
        "include_processes": True,
        "threat_threshold": 0.7
    }
}
```

#### Process Analyzer Node  

**Purpose**: Process monitoring and performance analysis

**Configuration**:
```python
{
    "type": "process_analyzer",
    "config": {
        "cpu_threshold": 80.0,
        "memory_threshold": 90.0,
        "include_children": True
    }
}
```

#### Network Analyzer Node

**Purpose**: Network connection and traffic analysis

**Configuration**:
```python
{
    "type": "network_analyzer",
    "config": {
        "interfaces": ["eth0", "wlan0"],
        "include_external": True,
        "port_scan": False
    }
}
```

## Security API

### SecurityPolicyEngine

#### `validate_tool_access(user_id: str, tool_name: str, arguments: dict) -> list`

**Description**: Validate user access to specific tools and arguments.

**Parameters**:
- `user_id` (str): User identifier
- `tool_name` (str): Tool being accessed
- `arguments` (dict): Tool arguments

**Returns**:
- `list`: List of policy violations (empty if valid)

**Example**:
```python
from security.security_policy import SecurityPolicyEngine

engine = SecurityPolicyEngine()

violations = engine.validate_tool_access(
    user_id="user123",
    tool_name="security_scan", 
    arguments={"scan_type": "full"}
)

# Example response
[
    {
        "type": "unauthorized_access",
        "severity": "high",
        "message": "User user123 not authorized for full security scans",
        "recommendation": "Assign security_analyst role"
    }
]
```

#### `assign_role(user_id: str, role_name: str, policy_name: str = "default") -> None`

**Description**: Assign a security role to a user.

**Parameters**:
- `user_id` (str): User identifier
- `role_name` (str): Role to assign (guest, user, analyst, admin)
- `policy_name` (str): Policy context (default: "default")

**Example**:
```python
engine.assign_role("user123", "analyst", "security_policy")
```

### AuditLogger

#### `log_action(user_id: str, action: str, tool_name: str, result: str) -> None`

**Description**: Log user actions for audit trails.

**Parameters**:
- `user_id` (str): User performing action
- `action` (str): Action being performed
- `tool_name` (str): Tool used
- `result` (str): Action result (success/failure)

**Example**:
```python
from security.audit_logger import get_audit_logger

logger = get_audit_logger()

logger.log_action(
    user_id="user123",
    action="security_scan", 
    tool_name="security_scanner",
    result="success"
)
```

#### `log_security_violation(violation_type: str, details: str, session_id: str = None) -> None`

**Description**: Log security policy violations.

**Parameters**:
- `violation_type` (str): Type of violation
- `details` (str): Violation details
- `session_id` (str): Optional session context

**Example**:
```python
logger.log_security_violation(
    violation_type="unauthorized_access",
    details="User attempted to access restricted table 'users'",
    session_id="session_abc123"
)
```

### RateLimiter

#### `is_allowed(user_id: str, action: str) -> bool`

**Description**: Check if action is allowed under rate limiting rules.

**Parameters**:
- `user_id` (str): User identifier
- `action` (str): Action being attempted

**Returns**:
- `bool`: True if allowed, False if rate limited

**Example**:
```python
from security.rate_limiter import RateLimiter

limiter = RateLimiter()

if limiter.is_allowed("user123", "security_scan"):
    # Proceed with action
    perform_security_scan()
else:
    # Rate limited
    return {"error": "Rate limit exceeded"}
```

## Web Interface API

### Workflow Builder Endpoints

#### GET `/api/workflows`

**Description**: List all available workflows.

**Response**:
```json
{
    "workflows": [
        {
            "id": "security_analysis",
            "name": "Security Analysis Workflow",
            "description": "Comprehensive security analysis",
            "created": "2025-11-10T12:00:00Z",
            "nodes": 5,
            "edges": 8
        }
    ]
}
```

#### POST `/api/workflows`

**Description**: Create a new workflow.

**Request**:
```json
{
    "name": "Custom Security Scan",
    "description": "Custom security analysis workflow",
    "nodes": [...],
    "edges": [...]
}
```

**Response**:
```json
{
    "id": "workflow_abc123",
    "status": "created",
    "validation": {
        "valid": true,
        "errors": []
    }
}
```

#### GET `/api/workflows/{id}/execute`

**Description**: Execute a specific workflow.

**Parameters**:
- `id` (str): Workflow identifier

**Query Parameters**:
- `query` (str): Input query
- `stream` (bool): Enable streaming responses

**Response**:
```json
{
    "execution_id": "exec_xyz789",
    "status": "running",
    "current_step": "security_analyzer",
    "progress": 60,
    "results": {...}
}
```

## REST API

### Base URL

```
http://localhost:8080/api/v1
```

### Authentication

```bash
# API Key authentication
curl -H "Authorization: Bearer your-api-key" \
     http://localhost:8080/api/v1/tools
```

### Endpoints

#### GET `/tools`

**Description**: List available tools.

**Response**:
```json
{
    "tools": [
        {
            "name": "system_info",
            "description": "System information retrieval",
            "category": "system",
            "parameters": {...}
        }
    ]
}
```

#### POST `/tools/{tool_name}/execute`

**Description**: Execute a specific tool.

**Parameters**:
- `tool_name` (str): Tool to execute

**Request**:
```json
{
    "arguments": {
        "filter": "name LIKE '%chrome%'",
        "limit": 10
    },
    "user_context": {
        "user_id": "user123",
        "session_id": "session_abc"
    }
}
```

**Response**:
```json
{
    "execution_id": "exec_123",
    "status": "completed",
    "results": [...],
    "metadata": {
        "execution_time": 0.25,
        "rows_returned": 5
    }
}
```

#### POST `/analyze`

**Description**: Perform AI-powered analysis.

**Request**:
```json
{
    "query": "analyze system for security threats",
    "type": "security_analysis",
    "options": {
        "include_recommendations": true,
        "confidence_threshold": 0.7
    }
}
```

**Response**:
```json
{
    "analysis_id": "analysis_456",
    "status": "completed", 
    "findings": [
        {
            "type": "suspicious_process",
            "severity": "high",
            "confidence": 0.85,
            "details": {...}
        }
    ],
    "recommendations": [...]
}
```

## Error Handling

### Error Response Format

All APIs use a consistent error response format:

```json
{
    "error": {
        "code": "ERROR_CODE",
        "message": "Human readable error message",
        "details": {
            "field": "specific error details"
        },
        "timestamp": "2025-11-10T12:00:00Z",
        "request_id": "req_abc123"
    }
}
```

### Common Error Codes

| Code | Description | HTTP Status |
|------|-------------|-------------|
| `INVALID_REQUEST` | Malformed request | 400 |
| `UNAUTHORIZED` | Authentication required | 401 |
| `FORBIDDEN` | Access denied | 403 |
| `TOOL_NOT_FOUND` | Unknown tool name | 404 |
| `VALIDATION_FAILED` | Parameter validation failed | 422 |
| `RATE_LIMITED` | Too many requests | 429 |
| `INTERNAL_ERROR` | Server error | 500 |
| `SERVICE_UNAVAILABLE` | Service temporarily unavailable | 503 |

### Error Handling Best Practices

```python
import requests
from requests.exceptions import RequestException

def call_api_with_retry(url, payload, max_retries=3):
    """Call API with exponential backoff retry"""
    
    for attempt in range(max_retries):
        try:
            response = requests.post(url, json=payload, timeout=30)
            
            if response.status_code == 200:
                return response.json()
            elif response.status_code == 429:  # Rate limited
                wait_time = 2 ** attempt
                time.sleep(wait_time)
                continue
            elif response.status_code >= 500:  # Server error
                if attempt < max_retries - 1:
                    time.sleep(1)
                    continue
            else:
                # Client error, don't retry
                response.raise_for_status()
                
        except RequestException as e:
            if attempt < max_retries - 1:
                time.sleep(1)
                continue
            raise
    
    raise Exception(f"API call failed after {max_retries} attempts")
```

## Rate Limiting

### Rate Limit Headers

All API responses include rate limiting information:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1636550400
X-RateLimit-Window: 60
```

### Rate Limiting Rules

| User Role | Requests/Minute | Burst Limit |
|-----------|----------------|-------------|
| Guest | 10 | 5 |
| User | 60 | 10 |
| Analyst | 120 | 20 |
| Admin | 300 | 50 |

### Handling Rate Limits

```python
def handle_rate_limit(response):
    """Handle rate limit responses"""
    
    if response.status_code == 429:
        reset_time = int(response.headers.get('X-RateLimit-Reset', 0))
        current_time = time.time()
        wait_time = max(0, reset_time - current_time)
        
        print(f"Rate limited. Waiting {wait_time} seconds...")
        time.sleep(wait_time + 1)  # Add 1 second buffer
        
        return True  # Indicates should retry
    
    return False  # No rate limiting
```

## SDK Examples

### Python SDK Usage

```python
from osquery_mcp_client import OSQueryClient

# Initialize client
client = OSQueryClient(
    base_url="http://localhost:8080/api/v1",
    api_key="your-api-key"
)

# List tools
tools = await client.list_tools()

# Execute tool
result = await client.execute_tool(
    "system_info", 
    arguments={}
)

# Analyze scenario
analysis = await client.analyze_scenario(
    "check for malware",
    type="security_analysis"
)

# Stream workflow execution
async for step in client.execute_workflow("security_workflow", initial_state):
    print(f"Step: {step['current_step']}")
```

### JavaScript SDK Usage

```javascript
import { OSQueryClient } from '@osquery-mcp/client';

const client = new OSQueryClient({
    baseUrl: 'http://localhost:8080/api/v1',
    apiKey: 'your-api-key'
});

// Execute tool
const result = await client.executeEtool('system_info', {});

// Analyze scenario
const analysis = await client.analyzeScenario(
    'suspicious network activity',
    { type: 'security_analysis' }
);

// Stream workflow
const workflow = client.executeWorkflow('security_workflow', initialState);
for await (const step of workflow) {
    console.log(`Current step: ${step.current_step}`);
}
```

## WebSocket API

### Connection

```javascript
const ws = new WebSocket('ws://localhost:8080/api/v1/ws');

ws.onopen = function() {
    console.log('Connected to OSQuery MCP WebSocket');
};

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    handleMessage(data);
};
```

### Message Format

```json
{
    "type": "workflow_update",
    "id": "workflow_123",
    "data": {
        "current_step": "security_analyzer",
        "progress": 75,
        "results": {...}
    },
    "timestamp": "2025-11-10T12:00:00Z"
}
```

### Subscription Types

- `workflow_updates`: Real-time workflow execution updates
- `tool_results`: Tool execution results
- `security_events`: Security violation notifications
- `system_alerts`: System health and performance alerts

---

*Last Updated: November 10, 2025*
*Version: 1.0.0*