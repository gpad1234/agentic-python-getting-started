# Test Results Report

**Generated:** 2025-11-09T21:19:45.970524
**Overall Status:** FAILED
**Duration:** 12.25s

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
| MCP Server Core | PASSED | 2.36s | 10/0/0/0 |
| Security Components | FAILED | 1.58s | 9/0/0/3 |
| Workflow Builder | FAILED | 2.45s | 12/1/0/0 |
| LangGraph Workflows | PASSED | 1.98s | 3/0/7/0 |
| LangChain Agent | PASSED | 1.7s | 1/0/10/0 |
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
