# Testing Best Practices

## TDD Workflow Guidance

### Phase Discipline

When writing tests (RED phase):
- Write tests that FAIL because implementation doesn't exist
- Do NOT write implementation code during this phase
- Do NOT write tests that pass against existing code (unless characterization tests)

When implementing (GREEN phase):
- Write the minimum code to make tests pass
- Do NOT add features beyond what tests require
- Do NOT refactor during this phase

When refactoring (REFACTOR phase):
- Improve code structure without changing behavior
- Tests must continue to pass throughout
- This is the only phase for "cleanup" work

### Prompting Patterns

| Instead of... | Say... |
|---------------|--------|
| "Write tests for X" | "Write FAILING tests for X, do NOT implement" |
| "Add tests and code" | "First write failing tests, then implement minimally" |
| "Make sure tests pass" | "Write tests that fail, then implement to pass" |
| "Test this feature" | "Write tests that will fail until feature is implemented" |

### Premature Pass Detection

A test that passes before implementation indicates:
1. **Weak test** - Test doesn't exercise new code
2. **Duplicate work** - Feature already exists
3. **Wrong behavior** - Test is checking something other than intended

**Action**: Investigate before proceeding to GREEN phase. Ask:
- Does this test actually require new code?
- Am I testing the right behavior?
- Is there existing code I'm unaware of?

### Test-Implementation Boundary

- Tests define WHAT, not HOW
- Tests should not dictate implementation details
- Multiple valid implementations can pass the same test
- If a test is too specific about implementation, it's brittle

```python
# BAD - Test dictates implementation
def test_cache_uses_lru():
    cache = Cache()
    assert isinstance(cache._storage, LRUDict)  # Implementation detail!

# GOOD - Test specifies behavior
def test_cache_evicts_least_recently_used():
    cache = Cache(max_size=2)
    cache.set("a", 1)
    cache.set("b", 2)
    cache.get("a")      # Access "a", making "b" least recent
    cache.set("c", 3)   # Should evict "b"
    assert cache.get("a") == 1
    assert cache.get("b") is None
    assert cache.get("c") == 3
```

---

## Anti-Patterns to Avoid

### 1. Trivial Assertions
```python
# BAD - Tests nothing meaningful
assert module is not None
assert CONSTANT == "expected_value"
assert result.count >= 0

# GOOD - Tests specific behavior
assert result.count == 3
assert "error_message" in result.errors
```

### 2. Redundant Tests
```python
# BAD - Two tests for same behavior
def test_failure_without_marker(): ...
def test_execute_returns_failure_without_marker(): ...  # Same thing!

# GOOD - One comprehensive test
def test_failure_without_marker(): ...
```

### 3. Over-Mocking
```python
# BAD - Mocks so much that no real code runs
with patch("module.ClassA"), patch("module.ClassB"), patch("module.function"):
    result = system_under_test()
    assert mock_a.called  # Only testing the mock!

# GOOD - Mock external boundaries only
with patch("subprocess.run") as mock_run:
    mock_run.return_value = MockResult(stdout="TASK_COMPLETE")
    result = runner.execute()
    assert result.success is True  # Testing real logic
```

### 4. Weak Comparisons
```python
# BAD - Would pass with 0, 1, or 2
assert loop_count <= 3

# GOOD - Exact expected value
assert loop_count == 3
```

### 5. Existence-Only Checks
```python
# BAD - Only checks things exist
assert session.logs_dir is not None
assert session.logs_dir.exists()

# GOOD - Verify correct values/behavior
assert session.logs_dir == expected_path / "logs"
assert (session.logs_dir / "run.log").exists()  # Check actual file
```

## Best Practices

### 1. Test Behavior, Not Implementation
```python
# BAD - Tests call order (implementation detail)
assert call_order == ["step1", "step2", "step3"]

# GOOD - Tests observable outcome
assert task_file.read_text().count("[x]") == 3
```

### 2. Use Specific Assertions
```python
# BAD - Vague
assert "error" in output

# GOOD - Specific
assert "ValidationError: max_loops must be >= 1" in output
```

### 3. Document Test Limitations
```python
def test_telemetry_directory_created(self):
    """Test telemetry directory is created.

    TODO: Add tests for actual telemetry writing when feature is implemented.
    """
    ...
```

### 4. Match Test Name to Actual Behavior
```python
# BAD - Misleading name
def test_log_files_written_to_correct_subdirectory():
    assert logs_dir.exists()  # No files written!

# GOOD - Accurate name
def test_logs_directory_created():
    assert logs_dir.exists()
```

### 5. Mock Callbacks Must Be Invoked
```python
# BAD - Mock returns value but never invokes callbacks
mock_executor.execute.return_value = 0  # Caller checks callback output!

# GOOD - Mock invokes callbacks like the real implementation
def simulate_execute(**kwargs):
    output_callback = mock_executor_class.call_args[1]["output_callback"]
    output_callback(b"Expected output\n")
    output_callback(b"TASK_COMPLETE\n")
    return 0
mock_executor.execute.side_effect = simulate_execute
```

### 6. Success Conditions Beyond Exit Codes
```python
# BAD - Assumes exit code 0 means success
mock_subprocess.return_value.returncode = 0
assert result.success is True  # Fails if code checks stdout!

# GOOD - Understand what the real code checks
# If code scans stdout for "COMPLETE" marker:
mock_subprocess.return_value.returncode = 0
mock_subprocess.return_value.stdout = "TASK_COMPLETE"
assert result.success is True
```

### 7. Platform-Specific Mock Targets
```python
# BAD - Assumes direct method call
mock_process.kill.assert_called_once()  # Fails on Windows!

# GOOD - Know what the implementation actually does per platform
import sys
if sys.platform == 'win32':
    # Windows uses taskkill subprocess
    mock_subprocess.run.assert_called_once()
else:
    mock_process.kill.assert_called_once()

# BETTER - Mock at a level that's platform-agnostic
killed_pids = kill_matching_processes()
assert target_pid in killed_pids  # Test outcome, not mechanism
```

### 8. Termination Conditions in Loop-Based Systems
```python
# BAD - Assumes completion means success
config = Config(max_loops=1)  # Only 1 loop allowed
result = runner.run()  # Has 3 tasks
assert result.success is True  # Fails! Hit max_loops before finishing

# GOOD - Understand termination precedence
# success=True only when termination_reason is NORMAL_COMPLETION
config = Config(max_loops=10)  # Enough for all tasks
result = runner.run()
assert result.success is True
assert result.termination_reason == TerminationReason.NO_PENDING_TASKS
```

## Test Categories

| Category | Purpose | Mock Level |
|----------|---------|------------|
| Unit | Test single function/class | Mock dependencies |
| Integration | Test component interactions | Mock external systems |
| E2E | Test full user workflow | Minimal mocking |

## Common Mock Gotchas

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Test passes but code doesn't work | Over-mocking hides bugs | Mock only at boundaries |
| Mock never called | Wrong patch target | Patch where it's *used*, not defined |
| Callback-based code fails | Mock doesn't invoke callbacks | Add `side_effect` that calls callbacks |
| Platform-specific failures | Different implementations per OS | Test outcome, not mechanism |
| "Method not found" errors | Mocking wrong object/level | Check actual class hierarchy |

## Checklist Before Committing Tests

- [ ] Each assertion tests meaningful behavior
- [ ] No duplicate tests for same behavior
- [ ] Test names match what's actually tested
- [ ] TODOs added for incomplete coverage
- [ ] Mocking is at appropriate boundaries
- [ ] Mocks invoke callbacks if the real code uses them
- [ ] Tests work on all target platforms (or are skipped appropriately)

## Test Audit Patterns

When auditing tests for quality, categorize issues by severity:

| Severity | Description | Action |
|----------|-------------|--------|
| CRITICAL | Tests with no meaningful assertions | Remove or enhance |
| HIGH | Tests that don't test what the name implies | Fix or rename |
| MEDIUM | Weak assertions that could be stronger | Strengthen assertions |
| LOW | Minor style/naming improvements | Address if time permits |

### Instantiation-Only Tests

Tests that only check instantiation are typically redundant:

```python
# REMOVE - Covered by other tests that use the class
def test_executor_instantiation(self):
    executor = TimeoutExecutor()
    assert executor is not None

# KEEP & ENHANCE - Actually test the behavior
def test_executor_callbacks_invoked(self):
    results = []
    executor = TimeoutExecutor(callback=lambda x: results.append(x))
    executor.execute(cmd=['echo', 'test'], timeout=10)
    assert len(results) >= 1  # Callback was invoked
```

### Schema/Config Tests

Tests verifying config schema (Pydantic, dataclasses) are valid if documented:

```python
def test_config_defaults(self):
    """Schema validation: Config uses correct default values.

    Note: This tests schema definition, not business logic.
    Catches accidental changes to defaults or field types.
    """
    config = Config()
    assert config.max_retries == 3  # Default
    assert config.timeout == 60  # Default
```

### File State Verification

When testing systems that modify files, verify the file state:

```python
# WEAK - Only checks output message
assert "Tasks completed: 2" in result.output

# STRONG - Verify actual file changes
task_content = task_file.read_text()
assert task_content.count("[x]") == 2  # Completed markers
assert task_content.count("[~]") == 1  # In-progress marker
```

### Return Type Documentation

When testing methods with complex return types, document the structure:

```python
def test_get_snapshot(self):
    """Test snapshot returns tracked processes.

    Returns: {(pid, create_time): peak_memory_mb}
    Initially empty before any child processes spawn.
    """
    snapshot = monitor.get_snapshot()
    assert isinstance(snapshot, dict)
    assert len(snapshot) == 0  # No children yet
```

### Mode-Dependent Behavior

Some flags may not affect all modes (e.g., --quiet in dry-run):

```python
def test_quiet_reduces_output(self):
    """Test quiet flag reduces output.

    Note: Quiet mode primarily affects execution output.
    In dry-run mode, output may be identical.
    """
    # Use <= not < if behavior is mode-dependent
    assert len(quiet_output) <= len(normal_output)
