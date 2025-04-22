# Quizlog Trace Format Specification

## Purpose

The whole project is quizlog.py

## Usage

with 2 argument, answer a question in a questionare
```
quizlog.py record.yaml answer
```
or with 1 argument, just show the questionare until current status
```
quizlog.py record.yaml
```

The `quizlog` is a sandbox that trace format defines a minimal, reproducible, and verifiable structure to represent the progress of AI agents executing a multi-step Python script. It is designed to allow stepwise interaction, resumption, auditing, and replay of reasoning processes under deterministic conditions.

## Key Concepts

- **Script-driven AI task**: The Python script encodes prompts (via `print()`), answer checkpoints (via `input()`), and validation rules (via `raise Exception`).
- **Deterministic execution**: A fixed seed ensures the problem content and logic remain consistent across runs.
- **Incremental interaction**: AI interacts with the script one `input()` at a time, based on previously recorded answers.

## Data Structure

The following is an example of `record.yaml`

```json record.json
{
  "script":"example.py"
  "output":"logfile.yaml"
  "seed": 123456,
  "code_hash": "abc123def456...",  // Optional but recommended
  "inputs": ["2", "4", "YES", "[[1,0],[0,1]]"],
  "pointer": 4,
  "print": ["message1","message2"], //record the printed messagess.
  "last_error": {
    "message": "UUᵀ ≠ A",
    "line": 12,
    "score": 0.4
  }
}
```

### Fields

- \`\` *(int)*: The random seed used for all internal randomness (e.g. random numbers, matrices). Ensures reproducibility.
- \`\` *(str, optional)*: Hash of the Python script to verify trace compatibility.
- \`\` *(list of str)*: AI-provided answers in order. These are injected sequentially into the script via monkey-patched `input()`.
- \`\` *(int)*: The index of the next unanswered `input()` call. Execution is resumed from here.
- \`\` "print" field just record what the system printed while executing the thing. Alghough one can recover it from run the script again, but it is good to have some record.
- \`\` *(object, optional)*:
  - `message`: The message passed to `raise Exception`, if applicable.
  - `line`: The script line number where the error occurred.
  - `score`: Partial score assigned before the error, if available.
 
- \`\` OUTPUT: the file that receives the returning value

## Status Inference (no explicit `status` field)

The current status can be inferred by executing the script with the given `inputs`:

- If script runs to completion → \`\`
- If execution reaches `input()` but no value available → \`\`
- If execution raises an exception → \`\`

## Replay & Resume Workflow

1. Load script and trace.
2. Set random seed.
3. Inject answers from `inputs` up to `pointer`.
4. Resume execution from that point:
   - If input needed → return `next_prompt`
   - If exception raised → update `last_error`
   - If script completes → declare `success`

## Example Usage

```json
POST /quizlog/next
{
  "script_id": "example_001",
  "seed": 123456,
  "inputs": ["2", "4"]
}
```
Or
```json
{
  "sheet_id":"record.json"
  "input":"super"
}
```
The input can be None, if None, the program only shows up the status until the current input series. (until pointer)

```json
Response
{
  "next_prompt": "Is the matrix positive semi-definite?",
  "input_type": "yesno",
  "pointer": 2
}
```

## Extensions (Optional)

- Add `start_time`, `end_time`, or `execution_id` for analytics.
- Store `outputs` for interactive display/debug.
- Add `feedback` field to store comments or review metadata.

## Summary

The `quizlog` trace format captures just enough to make AI reasoning reproducible, traceable, and verifiable — without storing redundant execution logs. It supports multi-round, script-guided interaction and is well-suited for AI auditing, training feedback, and system integration.

