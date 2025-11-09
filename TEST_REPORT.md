# Test Results Report

**Generated:** 2025-11-09T14:16:27.108740
**Overall Status:** FAILED
**Duration:** 28.27s

## Summary

| Metric | Count |
|--------|-------|
| Total Passed | 18 |
| Total Failed | 4 |
| Total Skipped | 17 |
| Total Errors | 0 |

## Test Suite Results

| Test Suite | Status | Duration | P/F/S/E |
|------------|--------|----------|---------|
| MCP Server Core | FAILED | 5.73s | 7/1/0/0 |
| Security Components | FAILED | 4.87s | 2/1/0/0 |
| Workflow Builder | FAILED | 4.54s | 5/1/0/0 |
| LangGraph Workflows | PASSED | 4.22s | 3/0/7/0 |
| LangChain Agent | PASSED | 3.09s | 1/0/10/0 |
| Integration Tests | FAILED | 5.81s | 0/1/0/0 |

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
