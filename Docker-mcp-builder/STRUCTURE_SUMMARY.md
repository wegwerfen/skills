# Docker MCP Builder Skill - Structure Summary

## Main Sections

1. **Overview** - Introduction to Docker MCP server development
2. **Process** - Four-phase workflow:
   - Phase 1: Planning and Requirements Gathering
   - Phase 2: Implementation
   - Phase 3: File Templates and Patterns
   - Phase 4: Deployment and Installation
3. **Implementation Patterns** - Code examples and best practices
4. **Quality Checklist** - Comprehensive verification lists

## Key Features

### Dual Language Support
- Python (FastMCP) with specific restrictions for Docker MCP Gateway
- Node.js (TypeScript) with MCP SDK and Zod validation

### Critical Rules
- Python: Single-line docstrings, no complex type hints, empty string defaults
- Node.js: Strict typing, Zod schemas, explicit return types
- Universal: Docker containers, non-root users, stderr logging

### Complete Templates
- Dockerfiles for both languages
- Dependency management (requirements.txt / package.json)
- Server code structures
- Documentation templates (readme.txt, CLAUDE.md)

### Deployment Guide
- Docker image building
- Secret management via Docker Desktop
- Custom catalog configuration
- Claude Desktop integration
- Testing and verification

## Comparison with Existing mcp-builder Skill

This skill differs from the existing mcp-builder skill by:
- Focus on Docker containerization and Docker MCP Gateway
- Specific restrictions for FastMCP in Docker environment
- Docker-specific deployment workflow
- Custom catalog and Claude Desktop configuration
- Security considerations for containerized environments

The existing mcp-builder focuses on general MCP development with evaluation-driven design, while this skill focuses on the Docker deployment model.
