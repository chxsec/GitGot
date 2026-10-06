<p align="center">
  <img src="logo.png" width=350/>
</p>

#


![License](https://img.shields.io/badge/license-LGPLv3%2B-lightgrey.svg)
![Python version](https://img.shields.io/badge/python-3.x-blue.svg)

## Description

GitGot is a semi-automated, feedback-driven tool to empower users to rapidly search through troves of public data on GitHub for sensitive secrets.

<p align="center">
  <img src="example_usage.png" width=80%/>
</p>

### How it Works

During search sessions, users will provide feedback to GitGot about search results to ignore, and GitGot prunes the set of results. Users can blacklist files by filename, repository name, username, or a fuzzy match of the file contents.

Blacklists generated from previous sessions can be saved and reused against similar queries (e.g.,
`example.com` v.s. `subdomain.example.com` v.s. `Example Org`). Sessions can also be paused and resumed at any time.

Read more about the semi-automated, human-in-the-loop design here: https://bishopfox.com/blog/semi-automated-vs-automated-security-tools.

## Install Instructions

### Install with uv (recommended)

GitGot uses [uv](https://docs.astral.sh/uv/) to manage its Python environment
and dependencies from a lockfile (`uv.lock`), so installs are fast and
reproducible across machines.

[1] Install uv (see the [uv docs](https://docs.astral.sh/uv/getting-started/installation/) for other platforms):
```sh
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh
```

[2] Clone the repo and sync the locked dependencies:
```sh
git clone https://github.com/chxsec/GitGot.git
cd GitGot
uv sync
```

`uv sync` creates a local `.venv` and installs the exact versions pinned in
`uv.lock`. You then run GitGot with `uv run` (see [Usage](#usage)) - no manual
virtualenv activation and no `pip install` step required.

### Manual Instructions (pip)

All dependencies are pure-Python (fuzzy hashing uses `ppdeep`), so no system
packages or compilers are required:
```sh
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```

### Docker Instructions

Run `gitgot-docker.sh` to build the GitGot docker image (if it doesn't already exist) and execute the dockerized version of the GitGot tool.

On invocation, `gitgot-docker.sh` will create and mount `logs` and `states` directories from the host's current working directory. If this `gitgot-docker.sh` is executed from the GitGot project directory it will update the docker container with changes to `gitgot.py` or `checks/`:

```sh
./gitgot-docker.sh -q example.com
```

(See `gitgot-docker.sh` for specific docker commands)
## Usage

GitHub requires a token for rate-limiting purposes. Create a [GitHub API token](https://github.com/settings/tokens) with **no permissions/no scope**. This will be equivalent to public GitHub access, but it will allow access to use the GitHub Search API.

The token can be supplied three ways. When more than one is present, the order of precedence is: `--token` flag, then the `GITHUB_ACCESS_TOKEN` environment variable, then the `ACCESS_TOKEN` constant in `gitgot.py`.

1. Environment variable (recommended, so the secret never lives in the source tree):
```sh
export GITHUB_ACCESS_TOKEN="<NO-PERMISSION-GITHUB-TOKEN-HERE>"
```

2. Command-line flag (convenient for one-off runs; note that a token passed this way is visible in your shell history and in the process list):
```sh
uv run gitgot.py -q example.com --token "<NO-PERMISSION-GITHUB-TOKEN-HERE>"
```

3. Hardcoded at the top of `gitgot.py` (take care not to commit it):
```sh
ACCESS_TOKEN = "<NO-PERMISSION-GITHUB-TOKEN-HERE>"
```

Once the token is set, you are ready to go. If you installed with uv, prefix each command with `uv run` (for example, `uv run gitgot.py -q example.com`). If you installed manually and activated the venv, use `./gitgot.py` as shown below:
```sh
# Default RegEx list and logfile location (/logs/<query>.log) are used when no others are specified.

# Query for the string "example.com" using default GitHub search behavior (i.e., tokenization).
# This will find com.example (e.g., Java) or example.com (Website)
./gitgot.py -q example.com

# Query self-hosted GitHub instance
./gitgot.py -q example.com -u https://git.example.com

# Query for the exact string "example.com". See Query Syntax in the next section for more details.
./gitgot.py -q '"example.com"'

# Query through GitHub gists
./gitgot.py --gist -q CompanyName

# Using GitHub advanced search syntax
./gitgot.py -q "org:github cats"

# Custom RegEx List and custom log files location
./gitgot.py -q example.com -f checks/default.list -o example1.log

# Recovery from existing session
./gitgot.py -q example.com -r example.com.state

# Using an existing session (w/blacklists) for a new query
./gitgot.py -q "Example Org" -r example.com.state
```
### Query Syntax

GitGot queries are fed directly into the GitHub code search API, so check out [GitHub's documentation](https://help.github.com/en/articles/searching-code) for more advanced query syntax.

### UI Commands
* **Ignore similar [c]ontent:** Blacklists a fuzzy hash of the file contents to ignore
future results that are similar to the selected file
* **Ignore [r]epo/[u]ser/[f]ilename:** Ignores future results by blacklisting selected strings
* **Search [/(mykeyword)]:** Provides a custom regex expression with a capture group to searches on-the-fly (e.g., `/(secretToken)`)
* **[a]dd to Log:** Add RegEx matches to log file, including all on-the-fly search results from search command
* **Next[\<Enter\>], [b]ack:** Advances through search results, or returns to previous results
* **[s]ave state:** Saves the blacklists and progress in the search results from the session
* **[q]uit:** Quit
