# Test Results Report

**Generated:** 2025-11-10T10:43:58.602368
**Overall Status:** FAILED
**Duration:** 11.37s

## Summary

| Metric | Count |
|--------|-------|
| Total Passed | 50 |
| Total Failed | 2 |
| Total Skipped | 17 |
| Total Errors | 0 |

## Test Suite Results

| Test Suite | Status | Duration | P/F/S/E |
|------------|--------|----------|---------|
| MCP Server Core | PASSED | 2.17s | 10/0/0/0 |
| Security Components | PASSED | 1.53s | 21/0/0/0 |
| Workflow Builder | FAILED | 1.52s | 14/1/0/0 |
| LangGraph Workflows | PASSED | 2.17s | 3/0/7/0 |
| LangChain Agent | PASSED | 1.77s | 1/0/10/0 |
| Integration Tests | FAILED | 2.2s | 1/1/0/0 |

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
