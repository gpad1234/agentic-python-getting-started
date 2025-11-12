# Troubleshooting Guide

## OSQuery MCP Multi-Orchestration Platform

*Version 2.0.0*

This comprehensive troubleshooting guide covers common issues, debugging techniques, and solutions for the OSQuery MCP multi-orchestration platform.

---

## Table of Contents

1. [Quick Diagnostics](#quick-diagnostics)
2. [MCP Server Issues](#mcp-server-issues)
3. [LangChain Agent Issues](#langchain-agent-issues)
4. [LangGraph Workflow Issues](#langgraph-workflow-issues)
5. [Security Issues](#security-issues)
6. [Performance Issues](#performance-issues)
7. [Database Issues](#database-issues)
8. [Deployment Issues](#deployment-issues)
9. [Integration Issues](#integration-issues)
10. [Monitoring and Logging](#monitoring-and-logging)
11. [Frequently Asked Questions](#frequently-asked-questions)

---

## Quick Diagnostics

### System Health Check

```bash
# Check all services status
curl http://localhost:8080/health
curl http://localhost:8081/health  
curl http://localhost:8082/health

# Check MCP server connection
echo '{"jsonrpc":"2.0","method":"tools/list","id":1}' | \
  curl -X POST -H "Content-Type: application/json" \
  -d @- http://localhost:8080/mcp

# Check LangChain agent
curl -X POST http://localhost:8081/api/agent/query \
  -H "Content-Type: application/json" \
  -d '{"query":"list running processes"}'

# Check LangGraph workflow
curl -X POST http://localhost:8082/api/workflow/execute \
  -H "Content-Type: application/json" \
  -d '{"workflow_id":"system_analysis","input":{}}'
```

### Environment Validation

```bash
# Validate Python environment
python --version  # Should be 3.8+
pip list | grep -E "(langchain|langgraph|fastapi|uvicorn)"

# Check OSQuery installation
osqueryi --version
osqueryi "SELECT version FROM osquery_info;"

# Validate required ports
netstat -tulpn | grep -E "(8080|8081|8082|6379)"

# Check system resources
df -h
free -m
top -n1 | head -20
```

---

## MCP Server Issues

### Issue: MCP Server Won't Start

**Symptoms:**
- Server fails to start
- Port binding errors
- Import errors

**Diagnostic Commands:**
```bash
# Check port availability
sudo netstat -tulpn | grep 8080

# Check Python imports
python -c "
import sys
sys.path.append('mcp_osquery_server')
from server import app
print('MCP server imports successful')
"

# Check OSQuery daemon
ps aux | grep osqueryd
sudo systemctl status osqueryd
```

**Solutions:**

1. **Port Already in Use:**
```bash
# Find and kill process using port 8080
sudo lsof -ti:8080 | xargs kill -9

# Or change port in configuration
export MCP_PORT=8090
python -m mcp_osquery_server.server --port 8090
```

2. **Missing Dependencies:**
```bash
# Reinstall dependencies
pip install -r requirements.txt --force-reinstall

# Check specific packages
pip install --upgrade fastapi uvicorn
```

3. **OSQuery Not Running:**
```bash
# Start OSQuery daemon
sudo systemctl start osqueryd
sudo systemctl enable osqueryd

# Manual start for debugging
sudo osqueryd --verbose --logger_plugin=stdout
```

### Issue: MCP Tools Not Working

**Symptoms:**
- Tool registration failures
- Query execution timeouts
- Invalid tool responses

**Diagnostic Commands:**
```bash
# Test tool registration
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":1}'

# Test specific tool
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc":"2.0",
    "method":"tools/call",
    "params":{
      "name":"get_processes",
      "arguments":{}
    },
    "id":2
  }'
```

**Solutions:**

1. **Tool Registration Issues:**
```python
# Debug tool registration
import logging
logging.basicConfig(level=logging.DEBUG)

from mcp_osquery_server.osquery_tools import OSQueryTools
tools = OSQueryTools()
print(f"Registered tools: {list(tools.tools.keys())}")
```

2. **Query Timeout Issues:**
```python
# Increase timeout in osquery_tools.py
QUERY_TIMEOUT = 30  # Increase from 10 seconds

# Or set environment variable
export OSQUERY_TIMEOUT=30
```

3. **Permission Issues:**
```bash
# Check OSQuery socket permissions
ls -la /var/osquery/
sudo chmod 666 /var/osquery/osquery.em

# Run server with appropriate permissions
sudo python -m mcp_osquery_server.server
```

---

## LangChain Agent Issues

### Issue: Agent Initialization Failures

**Symptoms:**
- Agent won't initialize
- OpenAI API connection errors
- Tool integration failures

**Diagnostic Commands:**
```bash
# Test OpenAI API connectivity
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"

# Test agent initialization
python -c "
from langchain.agents import initialize_agent
from langchain.llms import OpenAI
print('Agent initialization test passed')
"
```

**Solutions:**

1. **API Key Issues:**
```bash
# Set API key
export OPENAI_API_KEY="your-api-key-here"

# Verify in environment
echo $OPENAI_API_KEY

# Or set in .env file
echo "OPENAI_API_KEY=your-key" >> .env
```

2. **Model Access Issues:**
```python
# Test model access
import openai
openai.api_key = "your-key"

try:
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": "test"}],
        max_tokens=5
    )
    print("OpenAI API working")
except Exception as e:
    print(f"API Error: {e}")
```

3. **Rate Limiting:**
```python
# Implement exponential backoff
import time
from langchain.llms import OpenAI

llm = OpenAI(
    max_retries=3,
    request_timeout=60
)
```

### Issue: Tool Selection Problems

**Symptoms:**
- Agent selects wrong tools
- Poor query interpretation
- Inconsistent responses

**Diagnostic Commands:**
```python
# Debug tool selection
import logging
logging.getLogger("langchain").setLevel(logging.DEBUG)

# Test tool descriptions
from langchain.tools import Tool
print([tool.description for tool in agent.tools])
```

**Solutions:**

1. **Improve Tool Descriptions:**
```python
# Enhanced tool descriptions
tools = [
    Tool(
        name="get_system_info",
        description="""
        Get comprehensive system information including:
        - OS version and platform details
        - Hardware specifications
        - System uptime and boot time
        Use this for general system overview questions.
        """,
        func=osquery_tools.get_system_info
    )
]
```

2. **Custom Prompt Templates:**
```python
from langchain.prompts import PromptTemplate

template = """
You are a system administrator assistant with access to OSQuery tools.
When users ask about system information, always:
1. Identify the most specific tool for the query
2. Use exact parameter names
3. Provide clear, actionable results

Available tools: {tools}
User query: {input}
{agent_scratchpad}
"""

prompt = PromptTemplate(
    template=template,
    input_variables=["input", "tools", "agent_scratchpad"]
)
```

---

## LangGraph Workflow Issues

### Issue: Workflow Execution Failures

**Symptoms:**
- Workflow hangs at specific nodes
- State corruption
- Checkpoint save failures

**Diagnostic Commands:**
```python
# Debug workflow state
from langgraph import StateGraph
from your_workflow import WorkflowState

# Check state schema
print(WorkflowState.__annotations__)

# Debug node execution
import logging
logging.basicConfig(level=logging.DEBUG)
```

**Solutions:**

1. **State Schema Issues:**
```python
# Validate state schema
from typing import TypedDict, List
from langgraph import StateGraph

class ValidWorkflowState(TypedDict):
    query: str
    results: List[dict]
    step: str
    errors: List[str]

# Ensure all nodes return valid state
def process_node(state: ValidWorkflowState) -> ValidWorkflowState:
    # Always return complete state
    return {
        **state,
        "step": "processed",
        "results": state.get("results", [])
    }
```

2. **Checkpoint Configuration:**
```python
# Configure checkpoint storage
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.sqlite import SqliteSaver

# For development
checkpointer = MemorySaver()

# For production
checkpointer = SqliteSaver.from_conn_string("sqlite:///checkpoints.db")

workflow = StateGraph(WorkflowState)
workflow.set_checkpointer(checkpointer)
```

3. **Node Timeout Issues:**
```python
# Add timeout to node execution
import asyncio
from functools import wraps

def timeout_node(timeout_seconds=30):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            try:
                return await asyncio.wait_for(
                    func(*args, **kwargs),
                    timeout=timeout_seconds
                )
            except asyncio.TimeoutError:
                raise Exception(f"Node {func.__name__} timed out after {timeout_seconds}s")
        return wrapper
    return decorator

@timeout_node(30)
async def query_node(state: WorkflowState) -> WorkflowState:
    # Node implementation
    pass
```

### Issue: Conditional Routing Problems

**Symptoms:**
- Incorrect workflow paths
- Infinite loops
- Missing route conditions

**Solutions:**

1. **Debug Routing Logic:**
```python
def debug_router(state: WorkflowState) -> str:
    route = determine_route(state)
    print(f"State: {state}")
    print(f"Selected route: {route}")
    return route

# Add to workflow
workflow.add_conditional_edges(
    "start_node",
    debug_router,
    {
        "analyze": "analysis_node",
        "query": "query_node",
        "end": "END"
    }
)
```

2. **Validate Route Conditions:**
```python
def safe_router(state: WorkflowState) -> str:
    # Always provide fallback
    if "error" in state:
        return "error_handler"
    elif state.get("query"):
        return "query_node"
    else:
        return "END"  # Always have a way to end
```

---

## Security Issues

### Issue: Authentication Failures

**Symptoms:**
- Login failures
- Token validation errors
- Permission denied errors

**Diagnostic Commands:**
```bash
# Test authentication endpoint
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"test"}'

# Validate JWT token
python -c "
import jwt
token = 'your-jwt-token'
try:
    payload = jwt.decode(token, verify=False)
    print(f'Token payload: {payload}')
except Exception as e:
    print(f'Token error: {e}')
"
```

**Solutions:**

1. **JWT Configuration:**
```python
# Check JWT secret
import os
JWT_SECRET = os.getenv("JWT_SECRET")
if not JWT_SECRET:
    raise ValueError("JWT_SECRET environment variable required")

# Token validation
def validate_token(token: str) -> dict:
    try:
        payload = jwt.decode(
            token,
            JWT_SECRET,
            algorithms=["HS256"]
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(401, "Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(401, "Invalid token")
```

2. **RBAC Configuration:**
```python
# Debug RBAC permissions
def check_permissions(user_id: str, resource: str, action: str) -> bool:
    user_roles = get_user_roles(user_id)
    print(f"User {user_id} roles: {user_roles}")
    
    for role in user_roles:
        permissions = get_role_permissions(role)
        print(f"Role {role} permissions: {permissions}")
        
        if has_permission(permissions, resource, action):
            return True
    return False
```

### Issue: SQL Injection Vulnerabilities

**Symptoms:**
- Security scanner alerts
- Unexpected query results
- Error logs showing malformed SQL

**Solutions:**

1. **Enhanced SQL Validation:**
```python
import re
import sqlparse

def validate_osquery_sql(sql: str) -> bool:
    # Parse SQL
    try:
        parsed = sqlparse.parse(sql)
    except:
        return False
    
    # Check for dangerous patterns
    dangerous_patterns = [
        r'\b(DROP|DELETE|UPDATE|INSERT|ALTER|CREATE)\b',
        r'--',  # SQL comments
        r'/\*.*\*/',  # Block comments
        r'\bUNION\b',  # Union attacks
        r'\bEXEC\b',  # Execute statements
    ]
    
    sql_upper = sql.upper()
    for pattern in dangerous_patterns:
        if re.search(pattern, sql_upper):
            return False
    
    # Whitelist allowed tables
    allowed_tables = {
        'processes', 'users', 'system_info', 'network_interfaces',
        'file_events', 'process_events', 'socket_events'
    }
    
    # Extract table names and validate
    tables = extract_table_names(sql)
    for table in tables:
        if table not in allowed_tables:
            return False
    
    return True
```

---

## Performance Issues

### Issue: Slow Query Response Times

**Symptoms:**
- High response latency
- Timeout errors
- Resource exhaustion

**Diagnostic Commands:**
```bash
# Monitor system resources
top -p $(pgrep -f "python.*server")
iostat -x 1 5
vmstat 1 5

# Check OSQuery performance
osqueryi --verbose ".timer on" "SELECT * FROM processes;"

# Profile Python application
python -m cProfile -o profile.stats -m mcp_osquery_server.server
```

**Solutions:**

1. **Query Optimization:**
```python
# Implement query caching
import redis
from functools import wraps

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_query(timeout=300):
    def decorator(func):
        @wraps(func)
        def wrapper(sql_query, *args, **kwargs):
            cache_key = f"osquery:{hash(sql_query)}"
            
            # Try cache first
            cached_result = redis_client.get(cache_key)
            if cached_result:
                return json.loads(cached_result)
            
            # Execute query
            result = func(sql_query, *args, **kwargs)
            
            # Cache result
            redis_client.setex(
                cache_key,
                timeout,
                json.dumps(result)
            )
            return result
        return wrapper
    return decorator

@cache_query(timeout=60)
def execute_osquery(sql: str) -> List[dict]:
    # Original query execution
    pass
```

2. **Connection Pooling:**
```python
# OSQuery connection pool
import asyncio
from asyncio import Queue

class OSQueryPool:
    def __init__(self, max_connections=10):
        self.pool = Queue(maxsize=max_connections)
        self.max_connections = max_connections
        
    async def get_connection(self):
        try:
            return await asyncio.wait_for(
                self.pool.get(),
                timeout=5.0
            )
        except asyncio.TimeoutError:
            raise Exception("Connection pool exhausted")
    
    async def release_connection(self, conn):
        await self.pool.put(conn)
```

3. **Rate Limiting:**
```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)

@app.middleware("http")
async def rate_limit_middleware(request, call_next):
    try:
        response = await call_next(request)
        return response
    except RateLimitExceeded:
        return JSONResponse(
            status_code=429,
            content={"error": "Rate limit exceeded"}
        )

@app.post("/api/query")
@limiter.limit("10/minute")
async def query_endpoint(request: Request, query: QueryRequest):
    # Query implementation
    pass
```

### Issue: Memory Leaks

**Symptoms:**
- Continuously increasing memory usage
- Out of memory errors
- System slowdown

**Solutions:**

1. **Memory Profiling:**
```python
import tracemalloc
import gc

# Start memory tracing
tracemalloc.start()

def monitor_memory():
    current, peak = tracemalloc.get_traced_memory()
    print(f"Current memory: {current / 1024 / 1024:.2f} MB")
    print(f"Peak memory: {peak / 1024 / 1024:.2f} MB")
    
    # Get top memory consumers
    snapshot = tracemalloc.take_snapshot()
    top_stats = snapshot.statistics('lineno')
    
    print("Top 10 memory consumers:")
    for stat in top_stats[:10]:
        print(stat)

# Force garbage collection
def cleanup_memory():
    gc.collect()
    print(f"Garbage collected: {gc.get_count()}")
```

2. **Resource Cleanup:**
```python
import contextlib
from typing import AsyncContextManager

class OSQueryConnection:
    def __init__(self):
        self.connection = None
    
    async def __aenter__(self):
        self.connection = await create_connection()
        return self.connection
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.connection:
            await self.connection.close()
        self.connection = None

# Usage
async def execute_query(sql: str):
    async with OSQueryConnection() as conn:
        return await conn.execute(sql)
```

---

## Database Issues

### Issue: OSQuery Database Corruption

**Symptoms:**
- Query failures
- Inconsistent results
- Database lock errors

**Solutions:**

1. **Database Recovery:**
```bash
# Stop OSQuery service
sudo systemctl stop osqueryd

# Check database integrity
sqlite3 /var/osquery/osquery.db "PRAGMA integrity_check;"

# Repair database
sqlite3 /var/osquery/osquery.db "VACUUM;"

# Reset if corrupted
sudo rm /var/osquery/osquery.db
sudo systemctl start osqueryd
```

2. **Backup Strategy:**
```bash
#!/bin/bash
# backup_osquery.sh

BACKUP_DIR="/var/backups/osquery"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup database
cp /var/osquery/osquery.db "$BACKUP_DIR/osquery_$TIMESTAMP.db"

# Backup logs
tar -czf "$BACKUP_DIR/logs_$TIMESTAMP.tar.gz" /var/log/osquery/

# Clean old backups (keep 7 days)
find $BACKUP_DIR -name "*.db" -mtime +7 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete
```

### Issue: Redis State Store Issues

**Symptoms:**
- State persistence failures
- Cache miss errors
- Redis connection timeouts

**Solutions:**

1. **Redis Configuration:**
```bash
# Check Redis status
redis-cli ping

# Monitor Redis
redis-cli monitor

# Check Redis memory usage
redis-cli info memory

# Configure Redis persistence
echo "save 900 1" >> /etc/redis/redis.conf
echo "save 300 10" >> /etc/redis/redis.conf
```

2. **Connection Recovery:**
```python
import redis
import time
from functools import wraps

def redis_retry(max_retries=3):
    def decorator(func):
        @wraps(func)
        def wrapper(self, *args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(self, *args, **kwargs)
                except redis.ConnectionError as e:
                    if attempt == max_retries - 1:
                        raise e
                    time.sleep(2 ** attempt)
                    self.reconnect()
            return None
        return wrapper
    return decorator

class ResilientRedisClient:
    def __init__(self, host='localhost', port=6379):
        self.host = host
        self.port = port
        self.client = None
        self.reconnect()
    
    def reconnect(self):
        self.client = redis.Redis(
            host=self.host,
            port=self.port,
            socket_connect_timeout=5,
            socket_timeout=5,
            retry_on_timeout=True
        )
    
    @redis_retry()
    def set(self, key, value, ex=None):
        return self.client.set(key, value, ex=ex)
    
    @redis_retry()
    def get(self, key):
        return self.client.get(key)
```

---

## Deployment Issues

### Issue: Docker Container Problems

**Symptoms:**
- Container startup failures
- Port mapping issues
- Volume mounting problems

**Solutions:**

1. **Container Debugging:**
```bash
# Check container logs
docker logs osquery-mcp-server

# Inspect container
docker inspect osquery-mcp-server

# Enter container for debugging
docker exec -it osquery-mcp-server /bin/bash

# Check container resources
docker stats osquery-mcp-server
```

2. **Dockerfile Optimization:**
```dockerfile
FROM python:3.9-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    osquery \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m -u 1000 osquery

# Set working directory
WORKDIR /app

# Copy requirements first (for better caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Change ownership
RUN chown -R osquery:osquery /app

# Switch to non-root user
USER osquery

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

EXPOSE 8080
CMD ["python", "-m", "mcp_osquery_server.server"]
```

### Issue: Kubernetes Deployment Problems

**Symptoms:**
- Pod startup failures
- Service discovery issues
- Resource constraints

**Solutions:**

1. **Kubernetes Debugging:**
```bash
# Check pod status
kubectl get pods -l app=osquery-mcp

# Check pod logs
kubectl logs -l app=osquery-mcp -f

# Describe pod for events
kubectl describe pod <pod-name>

# Check service endpoints
kubectl get endpoints osquery-mcp-service

# Check resource usage
kubectl top pods -l app=osquery-mcp
```

2. **Resource Configuration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: osquery-mcp-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: osquery-mcp
  template:
    metadata:
      labels:
        app: osquery-mcp
    spec:
      containers:
      - name: mcp-server
        image: osquery-mcp:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

## Integration Issues

### Issue: Claude Desktop Connection Problems

**Symptoms:**
- MCP server not recognized
- Connection timeouts
- Tool execution failures

**Solutions:**

1. **Claude Desktop Configuration:**
```json
// ~/.claude_desktop_config.json
{
  "mcpServers": {
    "osquery": {
      "command": "python",
      "args": ["-m", "mcp_osquery_server.server"],
      "env": {
        "OSQUERY_SOCKET": "/var/osquery/osquery.em"
      }
    }
  }
}
```

2. **Connection Testing:**
```bash
# Test MCP server directly
python -m mcp_osquery_server.server --stdio

# Test with Claude Desktop debug mode
/Applications/Claude.app/Contents/MacOS/Claude --enable-logging --log-level=debug
```

### Issue: VS Code Extension Problems

**Symptoms:**
- Extension not loading
- Command execution failures
- WebView rendering issues

**Solutions:**

1. **Extension Debugging:**
```bash
# Open VS Code with extension logs
code --enable-logging --log-level=debug

# Check extension status
code --list-extensions --show-versions
```

2. **Extension Configuration:**
```json
// settings.json
{
  "osquery-mcp.serverUrl": "http://localhost:8080",
  "osquery-mcp.timeout": 30000,
  "osquery-mcp.logLevel": "debug"
}
```

---

## Monitoring and Logging

### Issue: Missing Logs or Metrics

**Symptoms:**
- No application logs
- Missing metrics data
- Monitoring dashboards empty

**Solutions:**

1. **Logging Configuration:**
```python
import logging
import sys
from logging.handlers import RotatingFileHandler

# Configure root logger
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.StreamHandler(sys.stdout),
        RotatingFileHandler(
            '/var/log/osquery-mcp/server.log',
            maxBytes=10485760,  # 10MB
            backupCount=5
        )
    ]
)

# Configure specific loggers
logging.getLogger("uvicorn").setLevel(logging.WARNING)
logging.getLogger("fastapi").setLevel(logging.INFO)
logging.getLogger("osquery").setLevel(logging.DEBUG)
```

2. **Metrics Collection:**
```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Define metrics
query_counter = Counter('osquery_queries_total', 'Total OSQuery queries')
query_duration = Histogram('osquery_query_duration_seconds', 'Query duration')
active_connections = Gauge('osquery_active_connections', 'Active connections')

# Instrument code
@query_duration.time()
def execute_query(sql: str):
    query_counter.inc()
    # Query execution
    pass

# Expose metrics endpoint
from prometheus_client import generate_latest

@app.get("/metrics")
async def metrics():
    return Response(
        generate_latest(),
        media_type="text/plain"
    )
```

3. **Structured Logging:**
```python
import json
import logging

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno
        }
        
        # Add extra fields
        if hasattr(record, 'user_id'):
            log_data['user_id'] = record.user_id
        if hasattr(record, 'query'):
            log_data['query'] = record.query
            
        return json.dumps(log_data)

# Configure JSON logging
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger = logging.getLogger('osquery_mcp')
logger.addHandler(handler)

# Usage
logger.info(
    "Query executed",
    extra={
        'user_id': 'user123',
        'query': 'SELECT * FROM processes',
        'duration': 0.5
    }
)
```

---

## Frequently Asked Questions

### Q: How do I enable debug logging?

**A:** Set the log level to DEBUG:

```bash
# Environment variable
export LOG_LEVEL=DEBUG

# Python logging
import logging
logging.getLogger().setLevel(logging.DEBUG)

# Command line
python -m mcp_osquery_server.server --log-level DEBUG
```

### Q: Why are my queries timing out?

**A:** Common causes and solutions:

1. **Increase timeout values:**
```python
# In osquery_tools.py
QUERY_TIMEOUT = 30  # seconds

# Environment variable
export OSQUERY_TIMEOUT=30
```

2. **Optimize queries:**
```sql
-- Instead of scanning all processes
SELECT * FROM processes;

-- Use specific filters
SELECT name, pid, cmdline FROM processes WHERE name LIKE 'python%';
```

3. **Check system load:**
```bash
top
iostat -x
```

### Q: How do I backup and restore the system?

**A:** Use the backup scripts:

```bash
# Backup
./scripts/backup_system.sh

# Restore
./scripts/restore_system.sh /path/to/backup
```

### Q: Can I run multiple instances?

**A:** Yes, configure different ports:

```bash
# Instance 1
python -m mcp_osquery_server.server --port 8080

# Instance 2  
python -m mcp_osquery_server.server --port 8081

# Instance 3
python -m mcp_osquery_server.server --port 8082
```

### Q: How do I update to the latest version?

**A:** Follow the update process:

```bash
# Backup current installation
./scripts/backup_system.sh

# Update code
git pull origin main

# Update dependencies
pip install -r requirements.txt --upgrade

# Run migrations
python scripts/migrate_database.py

# Restart services
sudo systemctl restart osquery-mcp-server
```

### Q: What are the system requirements?

**A:** Minimum requirements:

- **OS:** Linux (Ubuntu 18.04+, CentOS 7+), macOS 10.15+, Windows 10+
- **Python:** 3.8 or higher
- **Memory:** 2GB RAM minimum, 4GB recommended
- **Storage:** 1GB available space
- **Network:** Internet access for OpenAI API
- **OSQuery:** Version 5.0.0 or higher

### Q: How do I contribute to the project?

**A:** See the contribution guidelines:

1. Fork the repository
2. Create a feature branch
3. Write tests for new features
4. Submit a pull request
5. Follow the code review process

### Q: Where can I get help?

**A:** Multiple support channels:

- **Documentation:** [README.md](../README.md)
- **Issues:** GitHub Issues
- **Discussions:** GitHub Discussions  
- **Chat:** Discord/Slack community
- **Email:** support@osquery-mcp.com

---

## Emergency Procedures

### System Recovery

1. **Complete system failure:**
```bash
# Stop all services
sudo systemctl stop osquery-mcp-*

# Restore from backup
sudo ./scripts/restore_system.sh /path/to/latest/backup

# Verify restoration
sudo ./scripts/verify_system.sh

# Start services
sudo systemctl start osquery-mcp-*
```

2. **Database corruption:**
```bash
# Create emergency backup
cp /var/osquery/osquery.db /tmp/osquery.db.corrupt

# Restore from backup
sudo ./scripts/restore_database.sh

# Restart OSQuery
sudo systemctl restart osqueryd
```

### Contact Information

- **Emergency:** critical-support@osquery-mcp.com
- **General Support:** support@osquery-mcp.com
- **Documentation:** docs@osquery-mcp.com

---

*Last Updated: November 10, 2025*  
*Version: 2.0.0*