## Installation

```bash
# 1. Clone the GitHub repo
git clone https://github.com/skalaliya/first-agent-sam.git
cd first-agent-sam

# 2. Create a Python virtual environment
python3 -m venv .venv

# 3. Activate it
# macOS / Linux:
source .venv/bin/activate
# Windows (PowerShell):
.\.venv\Scripts\Activate.ps1

# 4. Install dependencies
pip install -r requirements.txt


### Why we do this:

1. **Isolate dependencies** so you don’t pollute your global Python site-packages.  
2. **Control versions**—everyone on the project uses the same library versions.  
3. **Easy cleanup**—just delete the `.venv` folder when you’re done.

After that, your **Usage** section stays the same:

```markdown
## Usage

```bash
# (venv should already be active)
python app.py



With those steps in place, anyone cloning your repo will know exactly how to spin up the right environment before running the app.



##################################################################################################################################################

## What Happens When You Send a Chat Message

Whenever you type into the “Chat Message” box and hit Enter, here’s what goes on under the hood:

1. **Gradio captures your input.**  
   Your browser sends your text over HTTP to the Python server running `app.py`.

2. **Your agent receives the prompt.**  
   The `CodeAgent` object inside the server adds your message to its conversation history.

3. **The agent plans its next move.**  
   - It prepends your prompt to its “Thought: / Code: / Observation:” cycle.  
   - It uses templates from `prompts.yaml` to decide what to do first (usually writing a “Thought:” line).

4. **The agent calls the language model.**  
   - It bundles the full conversation (system instructions, past steps, your new prompt).  
   - It sends that bundle via HTTP to the Qwen2.5-Coder-32B-Instruct model (by default hosted on Hugging Face).

5. **The model returns text.**  
   You usually get back a “Thought:” line or a code snippet.

6. **(If needed) Code execution.**  
   - If the agent generated a `Code:` block, it runs that Python code in your virtual environment.  
   - Any `print()` output or variables become the next “Observation.”

7. **Iteration.**  
   The agent loops through Thought → Code → Observation until it reaches a final answer.

8. **Final answer.**  
   The agent calls `final_answer` to signal it’s done. Gradio then displays that answer back in the chat window.

## Under-the-Hood Code Paths

Below is a simple, copy-and-paste–ready view of how everything ties together:

```python
# app.py
from smolagents import CodeAgent
from InferenceClientModel import InferenceClientModel
from tools.final_answer import FinalAnswerTool

# 1) Initialize the LLM client
model = InferenceClientModel(
    model_id='Qwen/Qwen2.5-Coder-32B-Instruct',
    temperature=0.5
)

# 2) Set up tools
final_answer = FinalAnswerTool()

# 3) Create the agent with prompts and tools
agent = CodeAgent(
    model=model,
    tools=[final_answer],
    prompt_templates=prompt_templates
)

# 4) Launch via Gradio
GradioUI(agent).launch()


### Inside `CodeAgent.run()`

1. **Build messages.**  
   Gather system instructions, past history, and the new user prompt into a `messages` list.

2. **Call the LLM.**  
   ```python
   response = self.model.generate(messages)
Sends messages over HTTP to the Qwen2.5-Coder-32B-Instruct model.

Parse “Thought:” and “Code:” blocks.
Split the response text into reasoning steps (Thought:) and Python code (Code:).

Run code blocks.
exec(code_block, globals())
observation = capture_stdout()

exec(code_block, globals())
observation = capture_stdout()

Any print() output becomes the next “Observation.”

Loop until done.
Repeat Thought → Code → Observation until the agent calls final_answer.

Return the final answer.
return final_answer_tool.format_output(value)
##################################################################################################

## Theoretical Foundations

The system brings together ideas from conversational AI, tool use, and software agent design:

1. **Prompt Engineering & Role Conditioning**  
   - We use a system prompt and templates (Thought:, Code:, Observation:) to shape the agent’s behavior.  
   - This structure guides the model to think in clear steps instead of jumping straight to an answer.

2. **Chain-of-Thought Reasoning**  
   - Each “Thought:” block lets the model explain its reasoning out loud.  
   - We can check or extend these thoughts by running Python code.

3. **Tool Invocation as API Calls**  
   - Code execution, web searches, and final-answer formatting all run as separate tools.  
   - Treating them like APIs keeps the core prompt simple and gives reliable results.

4. **Separation of Planning & Execution**  
   - The agent first plans (“Thought:”), then runs code (“Code:”), then reviews output (“Observation:”).  
   - This cycle mirrors control loops in software and robotics.

5. **Interactive Loop & State Persistence**  
   - Variables and imports stay loaded between code steps.  
   - That gives the agent memory across multiple reasoning steps, like a live REPL session.

6. **Transformer Inference with Latency Control**  
   - We batch messages and set temperature/token limits to balance creativity and speed.  
   - Step limits and verbosity settings help keep responses snappy.

7. **User Interface Abstraction**  
   - Gradio handles the HTTP/JS layer so the agent code doesn’t need to worry about networking.  
   - This makes the core agent logic UI-agnostic and easy to swap out if needed.

### Deployment Options

- **Local Deployment**  
  - Run `python app.py` on your machine.  
  - LLM calls still go to Hugging Face unless you host Qwen2.5-Coder locally.  
  - Requires internet to reach the Hugging Face API.

- **Hugging Face Spaces**  
  - Push your repo to HF Spaces for a hosted Gradio app.  
  - Everything runs on HF servers—users just need a browser.

- **Fully Offline (On-Premises)**  
  - Host your own inference server with Qwen2.5-Coder.  
  - Point `InferenceClientModel` to your local endpoint.  
  - Needs heavy compute but works without any internet connection.
