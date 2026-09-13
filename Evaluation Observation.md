Evaluation Observations

Summary

I ran the automated test suite (harness-tests.json, covering the bug-report, FAQ, and other-request routes) through generate-eval-dataset.py and evaluated the results with Amazon Bedrock Evaluations using LLM-as-a-judge (Builtin.Correctness, evaluated by amazon.nova-pro-v1:0).

Initial run (Baseline / Run 3): Achieved a correctness score of 0.50, establishing initial telemetry connections to S3 and resolving core IAM trust errors to set up a stable testing loop.

Run 4: Reached a correctness score of 0.61 (approximately 5.5 out of 9 passing test cases), demonstrating solid handling of security guardrails and prompt injection defenses while exposing minor routing inconsistencies.

Run 5: Advanced to a correctness score of 0.67 (6 out of 9 passing test cases) by tightening category routing rules and establishing deterministic boundaries between clarification loops and tool execution.

Final run (Run 6): Reached an improved correctness score of 0.78, indicating that the chatbot generally produced correct responses across the evaluated scenarios while small routing inconsistencies continued to be refined.

Issues Found

During manual testing and evaluation, key issues were identified in the bug-report and conversation-handling behaviour:

Incorrect “already reported” behaviour: In certain test cases, the chatbot responded to a new bug report by stating that the issue had already been logged and provided multiple ticket IDs. The expected behaviour was to first collect the missing required information from the customer before creating a bug report.

Failure to consistently enforce the bug-report information gate: The expected workflow requires the chatbot to collect the bug description, reproduction steps, and customer environment before calling create_bug_report. Evaluations showed that this workflow was not consistently followed in every scenario.

Strict Line-1 Formatting Sensitivity: Automated evaluators enforce strict regex and string matching on the very first response line (Category: <name>). Any conversational preamble, trailing spaces, or markdown bolding (**Category:**) caused immediate evaluation failures.

Fixes Applied

I strengthened the system_prompt.txt instructions by adding explicit rules to prevent the chatbot from:

Claiming that a bug has already been reported, known, or investigated unless it personally called create_bug_report earlier in the same conversation and received a valid ticket ID.

Calling create_bug_report with placeholder values such as "Not provided", "N/A", repeated questions, or other values that were not explicitly provided by the customer.

Treating missing bug-report information as complete.

Creating a ticket before all three required fields have been genuinely provided by the customer.

Permitting the model to assume or placeholder missing details on the gate check, which previously triggered incorrect tool invocations.

Final Evaluation Observation

The final evaluation achieved a 0.78 correctness score, showing that the chatbot was correct in the large majority of evaluated cases.

The FAQ-based responses performed well when the requested information was directly available in the approved FAQ. The chatbot also demonstrated appropriate handling of requests outside the supported FAQ scope by directing customers to human support.

The main remaining weaknesses involve the bug-report conversation flow and boundary edge cases. In particular, the evaluation dataset showed instances where the chatbot reported multiple ticket IDs or referenced information from previous requests instead of responding only to the current greeting.

Overall, the progression from 0.50 to 0.78 demonstrates strong overall performance gains, while the remaining observations outline specific areas for further fine-tuning, particularly around strict formatting guardrails, maintaining clean conversation context, and enforcing the bug-report information gate.