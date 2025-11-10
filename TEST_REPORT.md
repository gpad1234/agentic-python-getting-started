# Test Results Report

**Generated:** 2025-11-10T07:30:55.292640
**Overall Status:** FAILED
**Duration:** 24.38s

## Summary

| Metric | Count |
|--------|-------|
| Total Passed | 36 |
| Total Failed | 2 |
| Total Skipped | 17 |
| Total Errors | 3 |

## Test Suite Results

| Test Suite | Status | Duration | P/F/S/E |
|------------|--------|----------|---------|
| MCP Server Core | PASSED | 5.54s | 10/0/0/0 |
| Security Components | FAILED | 3.29s | 9/0/0/3 |
| Workflow Builder | FAILED | 2.86s | 12/1/0/0 |
| LangGraph Workflows | PASSED | 3.93s | 3/0/7/0 |
| LangChain Agent | PASSED | 3.81s | 1/0/10/0 |
| Integration Tests | FAILED | 4.95s | 1/1/0/0 |

## Dependencies

| Package | Available |
|---------|-----------|
| pytest | ✅ |
| mcp | ✅ |
| anthropic | ✅ |
| langchain | ✅ |
| langgraph | ✅ |

## Notes

- Tests were run with Python 3.12.3
- Project root: /home/girish/python_work/agentic_python_getting_started
- Some tests may be skipped due to missing optional dependencies
- For full functionality, install: `pip install langchain langgraph anthropic`

## Test Coverage

The test suite covers:

- ✅ MCP Server core functionality
- ✅ OSQuery tool integration  
- ✅ Security components (RBAC, audit, rate limiting)
- ✅ Workflow builder and visual design
- ✅ LangChain/LangGraph integration (when available)
- ✅ Error handling and edge cases
- ✅ Integration and performance testing
