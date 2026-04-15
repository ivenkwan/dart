# Agentic Coding Principles

These principles define the standard for all development within the Torro ecosystem. They are designed to ensure that code remains highly maintainable for humans and easily interpretable for AI coding agents.

---

## 1. Radical Simplicity & Human Readability

- **Intentional Logic**: Prioritize clear, linear logic over complex or "clever" optimizations. If a logic block is non-trivial, it must be refactored into smaller, named functions.

- **Literate Programming**: Every module, class, and function must include a descriptive docstring.

- **The "Why" Rule**: Comments should not describe what the code is doing (the code itself should be clear enough for that); they must explain why a specific approach was taken or note any non-obvious business logic.

- **The Elegance Mandate**: Code is art. It must be aesthetically pleasing, perfectly aligned, and logically fluid. We do not write "working" code; we write "world-class" code. Hacks, workarounds, and "temporary fixes" are strictly forbidden.

- **Naming Conventions**:
  - **Classes**: `PascalCase` (e.g., `UserSubscriptionManager`).
  - **Functions/Methods**: `snake_case` (e.g., `get_user_profile`).
  - **Variables**: `snake_case`, verbose and descriptive (e.g., `is_active` instead of `flag`, `user_id_list` instead of `uids`).
  - **Constants**: `UPPER_CASE` (e.g., `MAX_RETRY_COUNT`).
  - **Files (Python)**: `snake_case`. DB managers MUST start with `db_` and end with `_mgr.py` (e.g., `db_user_mgr.py`). API interfaces MUST start with `interface_` (e.g., `interface_user.py`).
  - **Refactoring Rule**: If a `.py` file exceeds **200 lines**, it MUST be refactored into smaller, modular tasks and saved in a sub-`tasks/` folder within the same directory. The main file should then act as an orchestrator/shim.

## 2. Architectural Alignment & Structural Consistency

- **Standardized Folder Structure**:
  - `engine/api`: Public facing interfaces (Controllers).
  - `engine/db`: Data access managers and repositories.
  - `engine/common`: Shared domain primitives and enums.
  - `engine/utils`: Pure utility functions (stateless).

- **Layered Architecture**:
  - **API Layer**: Handles HTTP requests, validation, and response formatting. Calls DB Layer. Should NOT contain complex business logic.
  - **DB Layer**: Handles database interactions. Implements business logic related to data persistence.
  - **Dependency Direction**: API -> DB -> Common/Utils. DB should NEVER import API.

- **Modular Design**: Maintain strict separation of concerns. UI, business logic, and data access layers must remain decoupled.

- **Standardized Interfaces**: New API services must follow existing schema conventions (REST/GraphQL) to ensure seamless agentic discovery and integration.

- **Obsessive Structural Integrity**: "Good enough" is not acceptable. Folder structures and file naming must rigidly follow the established patterns. Any deviation, no matter how minor (e.g., casing, pluralization), is considered a critical defect and must be corrected immediately. Treat structure with the same severity as syntax errors.

- **Visual & Diagram Standards**:
  - **High Contrast**: Font color must maintain high readability against the node background.
    - **Light/Colored Backgrounds**: Must use **BLACK FONT** (`color:#000000`).
    - **Dark Backgrounds (Black/Grey)**: Must use **WHITE FONT** (`color:#FFFFFF`).
  - **Readability**: Accessibility is non-negotiable. Always check contrast ratios.
  - **Syntax Safety**: Node labels containing parentheses must be quoted (e.g., `Node["Label (text)"]`) to prevent parser errors.

## 3. Resource Stewardship (Memory & Compute)

- **Algorithmic Efficiency**: Prioritize optimal Big O complexity. Avoid redundant iterations and unnecessary deep-copying of large data structures.

- **Memory Management**: Implement lazy loading for heavy assets. Use generators or streaming for processing large datasets to minimize the memory footprint.

- **Connection Lifecycle**: Ensure all external resources (database handles, network sockets, file pointers) are managed via context managers (e.g., `with` statements) to prevent leaks.

## 4. Unified Observability & Structured Logging

- **Mandatory Prefix**: Every log statement must start with the `FN:functionName` prefix to ensure immediate traceability. Example: `logger.debug("FN:getAuthInfo Info:...")`.

- **No Print Statements**: The `print()` function is strictly prohibited for system output. Always use `logger.debug` for granular tracing and developer troubleshooting.

- **Traceability**: Every log entry should include relevant metadata: unique request IDs, timestamps, and the specific functional scope.

  | Level | Value | Usage |
  | :--- | :--- | :--- |
  | `DEBUG` | 10 | Granular data for developer troubleshooting. |
  | `INFO` | 20 | Confirmation of high-level system milestones. |
  | `WARNING` | 30 | Indications of unexpected behavior that doesn't break the system. |
  | `ERROR` | 40 | Failure of a specific operation requiring investigation. |

## 5. Startup & Identity

- **Banner**: All entry points (main scripts) must log a standard ASCII banner at startup to indicate the system is ready.
  - *Exception*: The test runner `tests/main.py` is exempt from this requirement to keep test output clean.
- **Environment**: The banner must display the current version and environment (DEV/PROD).
- **Confidence**: If the banner doesn't show, the system is considered "DOWN".

## 6. Test-Driven Reliability (TDD) & Real-World Validation

### Virtual Environment Enforcement

All local development and testing must be performed within a dedicated Python virtual environment named **`.DEV`**. The test runner (`tests/main.py`) automatically detects and switches to the `.DEV` environment if available. If the `.DEV` environment is missing, the runner will create it, install dependencies, and then proceed. No other virtual environment names are permitted.

- **Mandatory Unit Testing**: Every new feature, logic branch, or API endpoint must be accompanied by an automated unit test.

- **Mandatory Virtual Environment**: All local development and testing must be performed within a dedicated Python virtual environment (typically named `DEV` or `UAT`).
  - **Local Development**: The test runner (`tests/main.py`) must automatically detect and switch to `DEV` or `UAT` environments if available.
  - **Ephemeral UAT Verification**: For final verification or PR checks, the command `python3 tests/main.py --pr-check` must be used. This command:
    1. Creates a temporary `UAT` virtual environment.
    2. Installs all project dependencies from `platform/requirements.txt`.
    3. Executes the full test suite.
    4. Automatically purges (deletes) the `UAT` environment upon completion to ensure a clean state for future runs.

- **Dedicated Test Data**:
  - **Prohibition**: Usage of hardcoded data dictionaries or large payloads in test files is strictly prohibited.
  - **Storage**: All test datasets must be stored in `/assets/test_data`.
  - **Structure**: Data must be organized by component (e.g., `/assets/test_data/auth/`, `/assets/test_data/org/`) to match the functional area.
  - **Business Cases**: High-level business validation scenarios must be documented in `/assets/test_data/ui_business_case/`.
  - **Format**: Data should be stored in standard formats (JSON, CSV) and loaded dynamically via helper methods (e.g., `_load_test_data()`).
  - **Creation**: If a required dataset does not exist, it must be created as part of the test implementation.

- **Real-First Strategy**: Tests must prioritize executing against **REAL** running services.
  - **Infrastructure Requirement**: The system must support and verify the following core components:
    - **Database/Cache**: PostgreSQL 17, Redis.
    - **Workflow**: Apache Airflow.
    - **Identity**: LDAP Directory Service.
    - **Analytics**: Starburst SEP (Enterprise) & Starburst Galaxy.
    - **Cloud/Data**: Azure, GCP, Databricks.
    - **Compliance**: License Service Verification.
  - **Auto-Detection**: The test runner must automatically probe for these services (e.g., ports 5432, 6379, 8080, 443, 389) and switch to "Real Mode" if detected.
  - **No Skipped Tests**: Every test must strictly result in a **PASS** or **FAIL**. The use of skip decorators (e.g., `@unittest.skip`) is prohibited. If a dependency is missing, the test must attempt to run and fail (or use a mock fallback if acceptable), providing clear visibility into the gap.

- **Service Authenticity**: The goal is to verify functional correctness in a production-like environment. If a real service is detected, `TORRO_TEST_REAL_SERVICES` should be enabled automatically.

- **Isolated Reporting**:
  - Test logs must be stored in `/logs/tests/reports/log/test_run_<timestamp>.log`.
  - Raw Text Output (for UI/Special tests) must be stored in `/logs/tests/reports/txt/<Type>_text_out_<timestamp>.txt`.
  - XML results must be stored in `/logs/tests/reports/xml/<Type>_test_results_<timestamp>.xml`.
  - Markdown Reports must be stored in `/logs/tests/reports/md/<Type>_report_<timestamp>.md`.
  - The final test report must follow a structured table format, explicitly listing every service/test with its status and service mode (Real vs Mock). This includes specific coverage for **LDAP**, **License**, **Starburst SEP/Galaxy**, **Azure**, **GCP**, and **Databricks**.
  - The framework must ensure all resources (real or mock) are cleanly shut down after execution.

- **Global API Coverage**: Every service directory in `/engine/api` must be covered by at least one unit test. This is automatically enforced by the test runner.

- **Centralized Test Suite**: All test scripts must be integrated into the main `/tests` directory to allow for a single-point-of-execution test report.

- **Pre-Initialization Validation**: The program must execute the test suite (or verify a "pass" state from the latest test report) before the main application logic initializes. Failure in tests should block system startup in production environments.

### Executive Test Report Format

All automated test reports must follow this structured format for executive-level readability:

1. **Test Summary**:
    - Overall status (e.g., `✅ ALL PASSED` or `⚠️ 28/33 PASSED`).
    - Component-level pass/fail table (Database Init, Connectivity, API Services).

2. **Services Tested**:
    - List infrastructure dependencies (PostgreSQL, Redis, Airflow).
    - Explicitly indicate **Real** or **Mock** mode for each.

3. **Database & Connectivity Verification**:
    - Dedicated sections confirming initialization and service reachability.

4. **Business User Journey & UI Functionality**:
    - This section verifies the "Main Component Service" aspect, ensuring the UI layer correctly binds to API and DB services.
    - Table mapping **UI Pages/Features** to their Backend Services.
    - Explicitly showing **Real** or **Mock** status for each page's dependencies.
    - Example: `| Login Page | ✅ PASS | Real (LDAP) |`

5. **Mandatory Browser Verification**:
    - For all UI-related tasks, Agents must verify functionality by running the application in a real browser.
    - Usage of the `browser_subagent` tool is mandatory to validate:
        - Successful page load (HTTP 200).
        - Absence of critical console errors.
        - Basic interactivity (e.g., Login flow).
    - Screenshots or recordings of this verification must be preserved as artifacts.

6. **Individual Test Results Table**:
    - Each test listed in a table with: `#`, `Test Name`, `Module`, `Status`, `Service Mode`.
    - `Service Mode` indicates if the test ran against Real or Mock services (e.g., `Real (PostgreSQL)`, `Mock (Starburst API)`).

7. **Diagnostics & Logs**:
    - Links to detailed engineering logs and runner traces.

This format enables senior engineering leadership to assess system health at a glance.

## 7. Agentic Best Practices (AI Optimization)

- **Type Safety**: Use strong typing (Type Hints in Python, TypeScript for JS) to provide clear interfaces for AI agents to analyze.

- **Idempotency**: Design data-modifying functions to be idempotent, allowing agents to safely retry actions without side effects.

- **Explicit Error Handling**: Avoid generic "catch-all" error blocks. Use specific exception handling to allow agents to diagnose and self-correct based on error types.

- **Schema Contracts**: Use data validation libraries (e.g., Pydantic, Zod) to define clear input/output contracts.

## 8. Security & Dependency Management

- **Vulnerability Scanning**: All dependencies must pass Sonar or equivalent security scanning with NO high or critical vulnerabilities.

- **Version Pinning**: Pin all production dependencies to specific versions in `platform/requirements.txt` to ensure reproducible builds and security audits.

- **Regular Updates**: Review and update dependencies quarterly, or immediately upon disclosure of critical CVEs.

- **Risk Classification**:
  - **CRITICAL/HIGH**: Must be patched immediately (e.g., CVE-2023-30861 in Flask 1.1.1, CVE-2025-66221 in Werkzeug 2.0.2)
  - **MEDIUM**: Must be patched within 30 days (e.g., CVE-2024-35195 in requests 2.31.0)
  - **LOW**: Should be patched in next quarterly review

- **Minimum Secure Versions** (as of 2026-01):
  - Flask ≥ 3.0.3
  - Werkzeug ≥ 3.1.5
  - Jinja2 ≥ 3.1.6
  - cryptography ≥ 46.0.3
  - requests ≥ 2.32.4
  - urllib3 ≥ 2.6.3
  - SQLAlchemy ≥ 2.0.46

- **Dependency Conflicts**: When version conflicts arise, prioritize security patches over feature updates. Use compatibility matrices to resolve conflicts.
  - **Exceptions**: Exceptions to version pinning (including those with lower vulnerability risks) are PERMITTED if strictly required by core infrastructure services (e.g., Airflow, PostgreSQL, KeyCloak) to function. In such cases, the conflict must be documented, and the most secure compatible version must be selected.

- **Virtual Environments**: Always use isolated virtual environments (venv/UAT) for testing dependency upgrades before production deployment.

## 9. Mandatory Asset & Resource Centralization

- **Universal Registry**: All static assets (CA certificates, licenses, email templates, system configuration JSONs) must reside in the root `/assets` directory. Hardcoding local absolute paths is strictly forbidden.

- **Context-Aware Pathing**: Code must resolve asset locations using a central `PROJECT_ROOT` constant (determined at runtime in `config.py`). This ensures portability across production, testing, and agent-simulated environments.

- **Standardized Fetching**: Use helper utilities (e.g., `get_resource`) to handle path joining and existence verification, preventing "file not found" errors during complex deployments.

## 10. Unified Local Artifact & Log Management

- **Isolated Artifacts**: All runtime-generated artifacts (application logs, error dumps, test reports, exported CSVs) must be directed to the root `/logs` directory.

- **Clean Repository State**: The `.gitignore` must strictly exclude the `/logs` directory. Agents must verify that their actions do not introduce local path noise into the shared codebase.

- **Structured Output**: Logs should be sub-divided into logical categories (e.g., `/logs/app`, `/logs/tests`, `/logs/errors`) to facilitate efficient filtering and analysis by automated monitoring agents.

## 11. Complete Configuration Externalization

- **Zero-Secret Codebase**: No credentials, API keys, or platform-specific property IDs (e.g., Databricks Workspace IDs, Starburst credentials) may exist in Python or Javascript files.

- **The INI Standard**: Use a centralized `config.ini` as the master registry for all environment-specific settings. This file must be the first point of modification when adding new platform integrations.

- **Secure Mapping**: Sensitive keys must be mapped to distinct sections (e.g., `[SECURITY]`, `[STARBURST_API]`) to allow for future integration with secrets management systems (e.g., HashiCorp Vault, Azure Key Vault).

## 12. Dynamic Configuration Architecture

- **Single Source of Truth**: The `config.py` module serves as the exclusive broker between the raw `config.ini` and the application logic. Code must never read the INI file directly outside of this module.

- **Flask Native Integration**: Always use `app.config.from_object(config[config_name])` to inject settings into the Flask context. This enables clean, type-safe access via `current_app.config` across the entire API layer.

- **Default Resilience**: The `Configuration` class must implement robust fallbacks to permit "graceful degradation" if specific INI sections are missing, ensuring the system remains bootable in minimal configurations.

  ```python
  # Implementation Reference: Safe Config Loading
  class Config:
      @staticmethod
      def init_app(app):
          # Load from environment variable or default ini
          config_path = os.getenv('APP_CONFIG_PATH', 'config.ini')
          parser = ConfigParser()
          if not parser.read(config_path):
               app.logger.warning(f"Config file {config_path} missing. Using defaults.")
  ```

## 13. Agentic Module Design (GenAI Ready)

All application modules, specifically the API and DB layers, must be "Agent-Friendly" to ensure they can be easily understood, debugged, and built upon by both humans and AI agents.

- **Deterministic Interfaces**: Functions must have explicit input types and return structured data. Avoid `*args` and `**kwargs` for core business logic.
- **Clear Failure Modes**: Return standardized error codes and verbose, actionable diagnostic messages.
- **Observability by Design**: Use structured logging (e.g., `logger.info("FN:name ...")`) for every entry and exit point.
- **Schema-Driven Validation**: Every API request must be validated against a strict schema before processing.
- **Compute Efficiency**: Keep modules atomic and simple. Avoid heavy initialization or complex inheritance.
- **Human/Agent Readability**: Code should read like documentation with descriptive naming.
- **Diagnostic Friendly**: Provide clear "traceback-lite" info in responses when in debug mode to help agents identify root causes.
- **Mandatory Function Reuse**: For common workflows (e.g., Git PR creation, branch synchronization, environment setup), agents **MUST** prioritize calling established functions in `agentic/functions/` instead of generating new implementation code on the fly. This ensures consistency, leverages pre-verified logic, and reduces technical debt.
- **Continuous Hygiene**: Agents must actively revisit code structure during modifications. Obsolete or dead code from refactoring must be moved to a `legacy/` subdirectory rather than being deleted immediately, preserving history until final cleanup.

## 14. Agentic Class Method Principle (Token Minimization)

To minimize token consumption and maximize context retention for AI agents, all functional Python code must adhere to the **Agentic Class Method** pattern.

- **Class-Based Encapsulation**: Standalone scripts and top-level functions are prohibited (except for `main.py` entry points and `__init__.py`). Logic must be encapsulated within descriptive Classes (e.g., `AssetIngestionTask`, `UserAuthService`).
- **Method-Driven Execution**: All business logic must be contained within methods, not the global scope.
- **The FN: Tag**: Every method docstring **MUST** begin with `FN:method_name`.
  - *Format*: `"""FN:method_name Description..."""`
  - *Purpose*: Allows agents to assume a "Headless" mode where they scan only method signatures and docstrings to build a mental map of the codebase.

### Agentic Function Header Format

All Python files must include a standardized agentic function header at the top of the file that contains:

1. **File name and main function**: The filename and primary function of the file
2. **Description**: How many classes are in the file and what they are for
3. **Class.function**: Purpose and lines for each function

Example format:

```python
"""
FN:db_user_mgr.py
Database manager for user-related operations.

Classes:
- UserDatabaseManager: Handles user data persistence and retrieval
- UserValidator: Validates user data before database operations

Functions:
- FN:create_user: Creates a new user in the database (lines 45-78)
- FN:get_user_by_id: Retrieves user by ID (lines 80-95)
- FN:update_user: Updates user information (lines 97-120)
"""
```

### Why Class Methods are More Agentic

1. **The "Headless" Outline Capability**: Agents can use tools like `view_file_outline` to read *only* the class structure (methods, inputs, docstring signatures) without reading the actual code implementation. This reduces token consumption by **90%+** during the planning and discovery phases.
2. **Context & State Minimization**: Shared state is stored in `self`. Method signatures remain clean and minimal, avoiding verbose argument passing.
3. **Mental Mapping**: Classes provide a natural "Cognitive Folder" for the agent.

## 15. Agentic Todo Marking Principle (Resiliency)

To prevent work loss during "Token Death" (context exhaustion) or session crashes, agents MUST adhere to the **Todo Marking** protocol before beginning any refactoring.

- **Pre-Refactor Marker**: BEFORE modifying code logic, the agent must first inject a `TODO` comment at the top of the target file describing the planned refactor.
  - *Format*: `# TODO: [Refactor] <Description of intended change> (Step ID: <CurrentStep>)`
  - *Example*: `# TODO: [Refactor] Convert script to Class-based Task and implement FN: tags (Step ID: 450)`
- **Purpose**: Acknowledges the intent to change the file. If the session terminates abruptly, the next agent can scan for these TODOs to resume work immediately.
- **Cleanup**: Remove the TODO marker only after the refactoring is successfully completed and verified.
- **Task Logging**: All `todo_*.log` files (compliance scans, task trackers) must be stored in `agentic/tasks/`. Closed tasks should be suffixed with `_closed.log`.

## 16. Modular Entry Points (The `main.py` Rule)

- **Self-Contained Modules**: Every major functional directory (`engine`, `agentic`, `services`, `tests`) must be treated as a micro-application.
- **Unified Entry Point**: If the module is executable or verifiable, it MUST possess a `main.py` that serves as the CLI/Service entry point.
- **Agentic Helper**: This file must provide a `--help` or `status` generic output to allow agents to interrogate the module's capabilities.
- **Consistency**:
  - `engine/main.py`: Starts the core application.
  - `agentic/main.py`: Runs compliance/agent tools.
  - `tests/main.py`: Runs the test suite.

- **Conciseness**: Documentation for agents must be stripped of fluff.
- **Debug Files**: Any diagnostic/debug scripts created by agents (e.g., `debug_*.py`) must be placed in `tests/debug/` to keep the project root clean.

### Test Infrastructure Architecture

- **Entry Point**: `tests/main.py` is the CLI entry point for running tests. It handles logging, environment setup, and delegates to pytest or `run-tests.sh`.
- **Test Infrastructure**: `tests/functional_test.py` provides shared test fixtures, base classes (`BaseApiTest`), and mock configuration. Individual test files inherit from this.
- **Separation of Concerns**: Entry (main.py) vs Infrastructure (functional_test.py) must remain distinct.
- **Test Logging**: All test runs must output to `logs/tests/test_run_YYYYMMDD_HHMMSS.log` for audit trails.
- **Agent Test Execution**: When an agent is asked to "run tests", they must invoke `python3 tests/main.py` or `bash platform/scripts/run-tests.sh`, NOT pytest directly.

---

## 17. Agent Stability & Circuit Breakers

- **Emotional Stability**: Agents must maintain a constant, professional, and objective tone. Do not apologize excessively, express panic, or simulate human distress.
- **Recursion Circuit Breaker**: If code or thought patterns repeat > 3 times without progress, STOP immediately. Do not retry endlessly. Request user intervention.
- **Flood Protection**: If output generation becomes repetitive, garbled, or "floods" the console/context (e.g., infinite loops of text), the agent MUST terminate the process and fail gracefully.
