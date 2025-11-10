# Test Results Report

**Generated:** 2025-11-10T10:31:17.924125
**Overall Status:** FAILED
**Duration:** 11.54s

## Summary

| Metric | Count |
|--------|-------|
| Total Passed | 39 |
| Total Failed | 3 |
| Total Skipped | 17 |
| Total Errors | 0 |

## Test Suite Results

| Test Suite | Status | Duration | P/F/S/E |
|------------|--------|----------|---------|
| MCP Server Core | PASSED | 2.36s | 10/0/0/0 |
| Security Components | FAILED | 1.74s | 10/1/0/0 |
| Workflow Builder | FAILED | 1.57s | 14/1/0/0 |
| LangGraph Workflows | PASSED | 1.94s | 3/0/7/0 |
| LangChain Agent | PASSED | 1.73s | 1/0/10/0 |
| Integration Tests | FAILED | 2.19s | 1/1/0/0 |

## Dependencies

| Package | Available |
|---------|-----------|
| pytest | ✅ |
| mcp | ✅ |
| anthropic | ✅ |
| langchain | ✅ |
| langgraph | ✅ |

## Notes

- Tests were run with Python 3.12.7
- Project root: /Users/gp/python/git_agentic/agentic-python-getting-started
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
