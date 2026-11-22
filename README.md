# Enterprise AI Knowledge Workspace

**Recruitbots AI — LLM Application Engineer · Challenge 3 (Advanced)**

One implementation file: [`knowledge_workspace.py`](knowledge_workspace.py).

**Atrium** is a multi-user knowledge workspace: upload documents into collections, search and chat with retrieval under **role-based access control**, generate business reports, and run multi-step workflow agents. Outputs are structured JSON. Evaluation metrics cover retrieval quality and RBAC enforcement.

User-facing answers and reports carry no machine-origin labels.

---

## Quick start

```bash
python knowledge_workspace.py --check
python knowledge_workspace.py --evaluate

python knowledge_workspace.py --user u_mem --search "production deploy"
python knowledge_workspace.py --user u_mem --chat "What is required for a production deploy?"
python knowledge_workspace.py --user u_mgr --report "expense approval"
python knowledge_workspace.py --user u_mem --workflow "severity severity-1 response"

# HTTP API
pip install fastapi uvicorn
python knowledge_workspace.py --serve
```

---

## Architecture

```text
Users (admin / manager / member / viewer)
              │
              ▼
     Collections (min_role gate)
              │
              ▼
     Documents ──chunk──► Hybrid RAG index
                          (BM25 + dense + RRF)
              │
      ┌───────┼────────┐
      ▼       ▼        ▼
   Search   Chat    Agents
                     ├─ report_agent
                     └─ workflow_agent
```

---

## Success criteria

| Criterion | Implementation |
| --- | --- |
| Document uploads | `upload_document` into a collection; index rebuild |
| RAG architecture | Hybrid BM25 + hashing dense vectors + RRF, ACL-filtered |
| Agents for automation | `report_agent`, `workflow_agent` (retrieve → synthesize → escalate) |
| Multi-user access | Seeded users with distinct roles |
| Business reports | Structured sections + sources from accessible docs only |

---

## Constraints

| Constraint | How it is met |
| --- | --- |
| Retrieval | `RAGIndex.search` on every chat, report, and workflow |
| Structured outputs | Chat / report / workflow return explicit `structured` objects |
| Evaluation metrics | Top-1 / top-3 retrieval accuracy; RBAC probes |
| Role-based access | Collection `min_role`; finance board hidden from viewers |

---

## Roles

| Role | Rank | Typical access |
| --- | --- | --- |
| `viewer` | 1 | People handbook; **not** Finance board notes |
| `member` | 2 | Engineering + People; upload to open collections |
| `manager` | 3 | Finance policies and board cash summary |
| `admin` | 4 | All collections; create collections |

Collection `min_role` is the floor for read/write. Search and chat only query collections the caller may read.

---

## Seeded content

- **Engineering** — on-call runbook, deployment policy  
- **Finance** (`min_role=manager`) — expense policy, Q3 board cash summary  
- **People** (`min_role=viewer`) — leave handbook  

---

## Agents

**Report agent** — retrieves accessible evidence for a topic and returns summary, sections, and source list.

**Workflow agent** — permission scope → retrieve → synthesize report → complete or `needs_review` when evidence is thin.

---

## Evaluation

`python knowledge_workspace.py --evaluate`

- Retrieval top-1 / top-3 on labeled probes  
- Viewer blocked from finance board cash doc  
- Manager can retrieve the same doc  
- Chat returns structured payload  

---

## API surface

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/health` | Stats |
| GET | `/collections?user_id=` | Visible collections |
| GET | `/documents?user_id=` | Visible documents |
| POST | `/upload` | Upload document |
| POST | `/search` | RAG search |
| POST | `/chat` | Document chat |
| POST | `/agents/report` | Report agent |
| POST | `/agents/workflow` | Workflow agent |
| GET | `/evaluate` | Metrics |

---

## Unmarked text

`strip_machine_labels` removes “AI-generated”, “powered by AI”, and similar phrases from answers, report bodies, and summaries.

---

## Self-check

`python knowledge_workspace.py --check` asserts upload, retrieval, chat citations, RBAC separation, report/workflow agents, evaluation floors, and clean text.

---

## File layout

```text
knowledge_workspace.py          # full workspace
README_knowledge_workspace.md   # this document
```
