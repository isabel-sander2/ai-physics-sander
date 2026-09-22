# ai-physics-sander

**PHYS 7440 — GenAI in Physics Research**
Week 02: Reproducible environment setup and a first hallucination audit

---

## What this notebook does

`week02_sander.ipynb` builds the minimal infrastructure for querying a large language model 
programmatically and uses it to assess how reliable that model's physics output is when 
checked against independent references. It defines `query_llm()`, a thin wrapper over the 
Anthropic Messages API that sends a single-turn prompt with no system prompt, no tools and no 
retrieval, so that every response reflects only the model's parametric knowledge — the 
condition under which hallucination is worth measuring. Three physics prompts are sent to 
`claude-haiku-4-5-20251001`: a literature search/citation request, a multi-step derivation of 
Hubble's constant from real Supernova data, and a error propagation request" -->. Each 
response is recorded verbatim to `responses.json`, compared against published values or 
provided calculations, and classified as correct, approximately correct, or hallucinated. 

## Environment

This project uses **conda**, with the environment pinned in `environment.yml`.

Conda was chosen over `pip freeze` because the assignment requires the Python version
itself to be part of the specification. A `requirements.txt` pins packages but says
nothing about the interpreter, whereas `conda env export` captures Python 3.11 and the
package set in a single file, so a fresh install reproduces the full stack rather than
just the libraries. The LLM SDKs are installed with `pip` inside the conda environment,
since they are not carried in the default conda channels; conda records them under the
`pip:` section of the export.

## Reproducing the results from scratch

**1. Clone the repository**

```bash
git clone https://github.com/<your-username>/ai-physics-sander.git
cd ai-physics-sander
```

**2. Create and activate the environment**

```bash
conda env create -f environment.yml
conda activate ai-physics
```

**3. Supply an API key**

The notebook reads an Anthropic API key from a `.env` file in the repository root.
This file is listed in `.gitignore` and is **not** part of the repository, so you must
create your own from the committed template:

```bash
cp .env.example .env
```

Then open `.env` in a text editor and paste your own key after the `=` sign:

```
ANTHROPIC_API_KEY=your-key-here
```

Keys are available from <https://console.claude.com> under *API keys*. Note that API
access is billed separately from a Claude.ai subscription. Use a text editor rather
than `echo` so the key does not enter your shell history, and do not add quotes,
spaces around the `=`, or an `export` prefix — `python-dotenv` will include them in
the value.

**4. Launch Jupyter and run the notebook**

```bash
jupyter notebook week02_sander.ipynb
```

Then *Kernel → Restart & Run All*.

Note that rerunning will issue fresh API calls and may not reproduce the stored
outputs verbatim. The responses analysed in this notebook are preserved
exactly as received in `responses.json`, with the model ID and a UTC timestamp for
each, so the analysis can be verified without re-running or even without an API key.

## API key handling

**No API key appears anywhere in this repository, in any committed file or in any
commit in its history.**

The key is read at runtime from a local `.env` file via `load_dotenv()` and accessed
through `os.environ`. It is never written as a literal in the notebook, never printed,
and never included in any stored cell output. `.env` is excluded by `.gitignore`, which
was committed before the `.env` file was created, so the key file was never tracked at
any point. A template `.env.example` is committed in its place, containing the variable
name and no value.

This was verified with:

```bash
git grep -nE "sk-ant-|sk-proj-|gsk_"              # tracked files, incl. notebook outputs
git log --all -p | grep -nE "sk-ant-|sk-proj-|gsk_"  # full commit history
git ls-files | grep -x ".env"                      # confirms .env is untracked
```

All three return no matches.

## Reflection on LLM reliability in a physics context At first I had a difficult time getting 
the LLM to offer me any concrete statements when asked directly for a reference or value with 
uncertainty. This is illustrated in prompt 1 of the responses.json where I request 3 citations 
for papers on cosmic ray acceleration between a certain year range. The LLM outright refuses 
to provide citations and even avoids giving vague references like "Look at Zhang et al. 2011". 
This might be due to previous models providing false citations/DOIs and now by default they 
avoid providing citations at all. Then, I moved to testing direct calculations of the Hubble 
constant from chat provided "real supernova data" which ended up being incorrect. The provided 
Supernovae are all exisiting objects but the given distances and recession velocities are not 
accurate. Then, it calculates Hubble's constant as 227 km/s/Mpc and does not acknowlege how 
different this is from the literature values between 67 - 76 km/s/Mpc. I believe the main 
issue the model had was with the velocity values because it was consistently providing 
explosion velocities rather than recession velocities. Lastly, I explored the LLM's ability to 
propogate error for a simple doppler shift calculation. The model was able to calculate the 
energy shift and uncertainty but the final provided value did not have correct significant 
figures. I deemed this example approximately correct.

   


## Repository contents

| File | Purpose |
|---|---|
| `week02_sander.ipynb` | The notebook: `query_llm`, three physics queries, verification and classification |
| `responses.json` | Verbatim model responses with model ID and UTC timestamps |
| `environment.yml` | Pinned conda environment, including Python 3.11 |
| `.env.example` | Template for the API key file |
| `.gitignore` | Excludes `.env`, `__pycache__/`, `*.pyc`, and Jupyter checkpoints |

## Model and reference sources

- Model: `claude-haiku-4-5-20251001` via the Anthropic Messages API (`anthropic` Python SDK) 

---

Isabel Sander · September 22nd, 2026
