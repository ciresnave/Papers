## LLM-Native Universal Tool Interface Specification (LUTIS)

### Overview

The LLM-Native Universal Tool Interface Specification (LUTIS) defines a set of principles, data structures, and interaction patterns that enable tools to efficiently communicate with large language models (LLMs). It aims to create interfaces designed *for LLMs*, emphasizing efficiency, token-awareness, semantic navigation, progressive disclosure, and composability.

LUTIS incorporates industry-standard concepts like JSON Schema and builds on emerging protocols such as the Model Context Protocol (MCP). It supports modern LLM usage patterns including asynchronous execution, memory integration, multimodal input/output, and human-in-the-loop oversight.

### Goals

- **Minimize context pollution** by avoiding unnecessary data injection.
- **Enable semantic navigation** through large datasets and codebases.
- **Provide declarative, scrollable, and composable data windows.**
- **Support live feedback, previews, and expansion on demand.**
- **Facilitate composability and multi-perspective reasoning.**
- **Enable progressive disclosure** of information based on LLM needs.
- **Support stateless operations** where possible to maintain conversation coherence.
- **Support agentic behaviors** including planning, reflection, and dynamic tool orchestration.

### Core Principles

1. **Token Economy**
   
   - Conscious of token usage
   - Provide size/token estimates
   - Support configurable response limits
   - Use summaries or excerpts when possible

2. **Semantic Over Syntactic**
   
   - Prioritize meaningful content
   - Support intent-based queries
   - Conceptual data navigation

3. **Progressive Disclosure**
   
   - Start minimal, expand on request
   - Support data element expansion

4. **Composability**
   
   - Chain and pipeline operations
   - Multi-source correlation

5. **Interface Consistency**
   
   - Standard endpoints like `/view`, `/search`, `/compose`
   - Uniform response formats using JSON Schema

6. **Extensibility and Versioning**
   
   - Explicit versioning for tools and LUTIS itself
   - Backward compatibility and deprecation policies

7. **Lifecycle Management**
   
   - Support tool versioning, rollback, observability, and cost monitoring

8. **Security and Ethics by Design**
   
   - Strong validation, consent, audit, and secure execution

### Additional Design Enhancements

**Reflective Tool Choice Logging**

- Tools should expose optional `why_invoked` fields to capture LLM rationale
- Helps with debugging, fine-tuning, and establishing trust
- Enables LLMs to explain rejected tools or preferred alternatives

**Chain Templates and Workflow Patterns**

- Recommended chaining modes: `sequential`, `fallback`, `parallel`, `conditional`, `iterative`
- Tools may declare composability: `can_chain_with: ["ToolX", "ToolY"]`
- Optional orchestration hints: `default_order`, `blocking`, `dependent_on`

**Multilingual & Internationalization Support**

- Parameters may include `title_translations` and `description_translations`
- Supported format: `"description_translations": {"en": "...", "fr": "..."}`
- Tool manifest should declare `supported_locales`

**LLM Feedback Quality Channel**

- Endpoint: `/feedback`
- Fields: `tool_name`, `success`, `clarity`, `suggestions`, `timestamp`
- Tools may offer post-call `rate_tool()` interactions

**Formal MCP Compatibility Profile**

- Tools compatible with MCP must define:
  - `mcp_compatible: true`
  - `host_role`, `client_id`, `resource_types`, `tool_categories`
- Optional MCP extensions include `dynamic_tool_discovery`, `capability_expressions`, and `structured prompts`

### Context Condensation for Enhanced Reasoning

Condensing tool outputs and surrounding context prior to reasoning is critical for performance, comprehension, and relevance. Poor condensation leads to diluted attention and faulty logic. LUTIS recommends the following multi-phase context refinement architecture:

#### Condensation Pipeline Phases

1. **Initial Condensation**
   
   - Generate functional, structural, and causal summaries of tool outputs
   - Prioritize declarative, high-signal content

2. **Recursive Refinement**
   
   - Repeat condensation in multiple passes until token budget or relevance saturation is reached
   - Include heuristics or signal thresholds to halt summarization when quality plateaus

3. **Multi-Model Condensation**
   
   - Run diverse models in parallel (e.g. instruction-tuned, long-context, domain-specific)
   - Merge consensus insights or union perspectives
   - Enables reasoning over richer, more multidimensional context

4. **Cross-Referencing and Linking**
   
   - Link condensed summaries to original views or sources for follow-up expansion
   - Ensure every summary can be traced and verified by the reasoning model

5. **Expansion Phase**
   
   - Optionally trigger web search, local knowledgebase queries, or tool reruns
   - Discover missing dependencies or data gaps

6. **Post-Expansion Condensation**
   
   - Re-condense expanded data with the same high-rigor summarization process
   - Avoid polluting final context window with excess data

#### Model Assignment Strategies

- Condenser model may be:
  
  - The same model used for reasoning (simplifies integration)
  - A lightweight, specialized summarization model (optimizes cost)
  - An ensemble of small models for diversity

- Ideally, models used in condensation should:
  
  - Be alignment- and reasoning-compatible
  - Be memory-aware or chunk-aware
  - Support relevance scoring and compression targeting

#### Suggested Output Fields

```json
{
  "summary": "...",
  "compression_ratio": 0.18,
  "source_refs": ["tool:search_logs"],
  "quality_rating": 0.91,
  "confidence": 0.86
}
```

This condensation process is vital for enabling scalable, compositional, and efficient reasoning.

### Core Concepts

**1. View**

* Window into structured content
* Scrollable, labeled, and expandable
* Supports `scroll`, `resize`, `close`, `refresh`, and `get_view_metadata`

**2. Anchor**

* Semantic markers with tags, summaries, and relationships
* Supports `preview`, `open_view_from_anchor`, and `find_related_anchors`

**3. Semantic Search**

* Intent-aware search over structured/unstructured data
* Enhanced queries include filters, ranking, and concept scopes

**4. Summary / Preview**

* Summarize before retrieving full content
* Multiple strategies (functional, structural, conceptual)
* Includes metrics and token counts

**5. Compose**

* Merge views into new contextual datasets
* Strategies include `contextual_merge`, `hierarchical`, etc.

**6. Agent**

* Background monitor
* Can be created, updated, paused, and queried
* Supports triggers and callback definitions

**7. Context Bridge**

* Maintain session-wide state, focus history, and intent chains

**8. Diff Stream**

* Time-based tracking of changes with impact analysis

**9. Dependency Graph**

* Semantic relationship map between elements

**10. Streaming and Real-time Data**

* Protocols like WebSocket and chunked HTTP
* Stream parameter controls: update frequency, chunk size

**11. Agent Management**

* `start_agent`, `stop_agent`, `update_agent_config`, etc.
* Supports configurable update frequencies and triggers

### Response Format Standards

**Standard Response Envelope** { "operation": "open_view", "success": true, "data": {}, "metadata": { "token_count": 256, "processing_time_ms": 120, "cache_hit": true, "tool_version": "1.0.0" }, "context": { "view_id": "main_fn", "related_operations": ["scroll", "resize"] }, "suggestions": [ { "operation": "preview", "target": "core_loop", "reason": "Related code block found", "estimated_token_cost": 200 } ] }

**Error Handling** { "success": false, "error": { "code": "FILE_NOT_FOUND", "message": "Source file not found", "recoverable": true, "suggestions": [ { "operation": "list_files", "parameters": {"pattern": "*.rs"} } ] } }

**Error Code Table**| Error Code | Description | Recovery Action ||------------|-------------|------------------|| 400 | Invalid input format | Validate and retry || 403 | Permission denied | Request access || 504 | Timeout | Retry or use cached result |

### Best Practices

* Summarize first, expand later
* Use anchors for navigation
* Avoid full-file dumps unless asked
* Provide token estimates, quality metrics, and response context
* Include multilingual metadata when supported
* Enable `/feedback` endpoint for LLM suggestions

### Tool Capabilities Declaration

Includes `capabilities`, `limits`, `performance`, and `supported_languages`.

### Quality Metrics

Track relevance, completeness, token efficiency, and context preservation.

### Security Considerations

* **Authentication/Authorization**: API key or OAuth2 with RBAC
* **Validation**: Strict schema checks on input
* **Encryption**: AES-256 and TLS 1.3 minimum
* **Audit Logs**: Mandatory fields (timestamp, operation, success, LLM id)
* **Isolation**: Session-level data isolation
* **Regulatory Compliance**: GDPR/CCPA where applicable

### Performance Standards

| Operation         | Max Response Time | Cache Hit Rate |
| ----------------- | ----------------- | -------------- |
| Semantic Search   | 500ms             | 80%            |
| Open View         | 300ms             | 90%            |
| Compose Operation | 1000ms            | 70%            |

### Implementation Guidelines

* **Minimum**: Core ops + token metadata + basic errors
* **Recommended**: Add compose, agents, context tracking
* **Advanced**: Real-time streams, diff tracking, predictive caching

### Validation and Testing

* Standardized test suite
* Certification levels from Basic to Premium

### Documentation

* Required API documentation with examples
* Sample workflows and usage patterns

### Contributing

* Submit suggestions via GitHub (TBD)
* Encourage LLM contributions with structured feedback

### License

MIT

---

**Version**: 1.3 (Context Condensation Pipeline Added)\
**Last Updated**: July 2025\
**Status**: Community Draft for Multi-AI Review
