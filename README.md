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
