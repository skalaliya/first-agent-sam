## Installation

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


Why we do this:
Isolate dependencies so you don’t pollute your global Python site-packages.

Control versions—everyone on the project uses the same library versions.

Easy cleanup—just delete the .venv folder when you’re done.

python app.py

What Happens When You Send a Chat Message
Whenever you type into the Chat box and hit Enter, here’s what happens under the hood:

Gradio captures your input.
Your browser sends your text to the Python server (app.py).

Agent receives the prompt.
The CodeAgent adds your message to its conversation history.

Agent plans its next move.
It uses the “Thought: / Code: / Observation:” cycle from prompts.yaml.

Agent calls the LLM.
The full conversation bundle is sent to Qwen2.5-Coder-32B-Instruct via HTTP.

Model returns text.
You usually get back a “Thought:” line or a code snippet.

(If needed) Code execution.
Any generated Code: block is run in your venv and its output becomes the next “Observation.”

Iteration.
The agent loops Thought → Code → Observation until it reaches a final answer.

Final answer.
The agent calls final_answer, and Gradio displays it in the chat window.

Under-the-Hood Code Paths
Here’s the core flow in app.py:
from smolagents import CodeAgent
from InferenceClientModel import InferenceClientModel
from tools.final_answer import FinalAnswerTool
from ui.gradio_ui import GradioUI

# 1) Initialize the LLM client
model = InferenceClientModel(
    model_id='Qwen/Qwen2.5-Coder-32B-Instruct',
    temperature=0.5
)

# 2) Set up tools
final_answer = FinalAnswerTool()

# 3) Create the agent
agent = CodeAgent(
    model=model,
    tools=[final_answer],
    prompt_templates=prompt_templates
)

# 4) Launch via Gradio
GradioUI(agent).launch()

Inside CodeAgent.run()
Build messages: system instructions + history + new user prompt.

Call the LLM: response = self.model.generate(messages).

Parse “Thought:” and “Code:” blocks.

Execute any Code: blocks and capture output.

Loop Thought → Code → Observation until final_answer is called.

Return the final answer to Gradio.

### Theoretical Foundations
Prompt Engineering & Role Conditioning via “Thought: / Code: / Observation:” templates.

Chain-of-Thought Reasoning lets you trace the model’s steps.

Tool Invocation as APIs keeps code execution, searches, and final-answer formatting modular.

Planning vs. Execution Loop mirrors software control loops.

Stateful Interactive Loop: variables persist across steps like a REPL.

Latency & Creativity Control: temperature and token limits tune performance and speed.

UI Abstraction: Gradio handles the web layer so your agent code stays focused.

Deployment Options
python app.py

(Connects to Hugging Face by default.)

Hugging Face Spaces
Push to HF Spaces for a hosted Gradio demo.

On-Premises
Host your own Qwen2.5-Coder inference server and point InferenceClientModel to it.
