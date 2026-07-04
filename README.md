# Ask

A lightweight CLI tool that sends prompts to [Opencode Zen](https://opencode.ai/zen) and **prints the response directly in your terminal** — no browser, no GUI, no pants required.

## Description

`ask` is a single-file bash script that does the heavy lifting so you don't have to leave your terminal:

1. Reads your Zen API key from `~/.local/share/opencode/auth.json` (or wherever you stashed it).
2. Picks the correct [Opencode Zen](https://opencode.ai/zen) endpoint for the selected model family:
   - `/v1/chat/completions` for OpenAI-compatible models (e.g. `big-pickle`, `qwen-*`, `deepseek-*`, `glm-*`, `minimax-*`)
   - `/v1/models/{model}:generateContent` for Gemini models (e.g. `gemini-*`)
3. Checks that your machine is actually online before pestering the API.
4. Gives you real time list of models available to you to select, WITH PRICING!
5. Sends the request and shows a live spinner with elapsed time so you know it hasn't dozed off.
6. Extracts the response with `jq` and displays it with Markdown formatting via `bat` (falls back to plain text if `bat` isn't installed; use `-t` to force plain terminal output).
7. Prepends a customizable system skill to every prompt, because a little coaching never hurt anyone.
8. Warns you if you have an "expensive" model selected (based on a $/token threshold that you can set)

No additional windows, no web UIs, no context-switching — just ask and get an answer right where you're already working.

How many times do you find yourself in the terminal and have some one off question, or "how do I grep for that?", or you read the man page for some command and don't have time to read a novel, or you tldr something but the examples do not relate to what you need to do, or you just need a simple bash script to do something, or a python script???

Instead of opening a browser to search, or opening some AI Chat interface (Desktop, Web, or TUI), just type "**ask**" and your question right in your terminal no matter where you are at and get your answer right there in your terminal!

This is not to be compared to or correlated with actual chat agents/harnesses or coding agents/harnesses. So, it does not use tools, sub-agents, or persist "sessions". One ask, one response. That is it. Technically you could manually copy your last ask and the last response into the interactive terminal and basically be "keeping your context across separate requests" but at that point use an appropriate tool. Again this has a purpose for the peeps who want to stay in their terminal and quickly get info on something and do it simply, quickly, and intuitively, then get back to what they were doing.

I initially started playing around with this idea because I got irritated context switching so much in and out of the terminal. Not to mention copy and pasting from browser windows, Chat Gipity, or even TUI's like opencode is generally a PITA.

So, you know how it goes... Got it doing what I wanted and then the "It could also do.." started. Then the "I bet other peeps might use this..." started. So, I (to the best of my abilty and with help of a couple LLMs) tried to make it useable to anyone who would recognize it for what it is.

## Requirements

> Note: Because **Opencode Zen API** does provide a free no auth required "big-pickle" model through their API (however, it is not guaranteed to work at any level and could become unavailable at any time in the future), I included the ability for `ask` to work with no `auth.json` file. Use it for evaluating but you really need to get an Opencode Zen API key to fully utilize ask. On that note, I believe the Zen is technically $20 (2 cups of foofoo coffee) and that $20 is your "balance" to use on the various models they offer. It is "pay as you go", so if you use up that $20 you initially signed up with, you can add more $ to your balance or be limited to the free models only.

- A valid **Opencode Zen API key** in `~/.local/share/opencode/auth.json` (or wherever you told `ask` to look). You don't have to install the `opencode` CLI; you can create the JSON file yourself if you're feeling rebellious.
- **OR** no API key at all, in which case `ask` will try the free `big-pickle` no-authentication endpoint. If it works, you're off to the races with only that model.
- `jq`, `curl`, and `tput` (usually already hanging out on most Linux distributions)
- `bat` (optional — makes Markdown responses look pretty)
- A working internet connection, because `ask` refuses to shout into the void.

## Usage

### Inline prompt

```bash
ask "What is the difference between var, let, and const in JavaScript?"
```

### Interactive mode

```bash
ask
```

Run without arguments and the script will politely ask you to type a question.

### Help

```bash
ask -h
ask --help
```

### Change model

List available models with pricing and pick one interactively (`ask -m` auto-refreshes pricing data from `opencode.ai/docs/zen` if it hasn't been fetched today):

```bash
ask -m
```

Set a model directly:

```bash
ask -m big-pickle
```

Set a model and send a prompt in one command:

```bash
ask -m big-pickle "explain monads"
```

### Set cost warning threshold

Set a cost threshold (per 1M input tokens) for warnings before sending a query with an expensive model. Default is `2.00`.

```bash
ask -c
ask -c 2.50
```

When a model exceeds the threshold, `ask` pauses to ask if you really meant to spend that much. Press `Enter` to continue, `m` to switch models, or anything else to exit and keep your wallet happy.

> **Note:** If you are running without an API key, the only model available is `big-pickle`. Running `ask -m` will remind you of this limitation.

### Attach a file

End the prompt with `< path/to/file` and `ask` will scoop up that file and staple its contents to the end of your prompt:

```bash
ask summarize this file '<' /path/to/file.txt
ask -m gemini-3.5-flash "explain this code < src/main.py"
ask                                               # then type: review this file < README.md
```

The `<` must be followed by at least one space and the path must be the last token. If the path doesn't exist as a readable file, the prompt is sent unchanged — so asking "is 5 < 10" won't accidentally try to open a file named `10`.

> **Note:** `<` is also a shell redirection operator. On the command line you must quote or escape it (e.g. `"prompt < file"` or `'<` `file'`) so the shell passes it to `ask` as part of the prompt. In interactive mode you can type it normally.

### System skill

Every prompt is automatically prepended with a system skill. Think of it as your personal hype coach whispering instructions before the model takes the stage. By default it says:

> You: expert in computers. All answers to be cited. Cites dated from when cite was obtained in your training data. Responses should be markdown with diagrams if helpful, unless response less than 10 lines. Answer quick, short, and to the point. Do not ask for clarifications. If you make assumptions, include those assumptions in response.

Customize or silence it by editing `~/.config/ask/SKILLS.md`:

If, for example, you know the majority of questions you will be asking will revolve around basket weaving. You probably should change the SKILLS.md and put something in there like "you are an Olympic basket weaving coach." But understand that with that skill in there, if you do start asking stuff about computers, you might get some peculiar responses! ;)

> **Note:** If you never run `ask -s` and `~/.config/ask/SKILLS.md` doesn't exist, the default skill is prepended to every prompt automatically.

```bash
ask -s
```

- If `SKILLS.md` does not exist, `ask` creates it with the default skill and opens it in `$EDITOR`.
- If it already exists, `ask` just pops it open in `$EDITOR`.
- If `SKILLS.md` is empty, no system skill is prepended. Go wild and make the model fend for itself.

The `-s` and `-h` flags work even if you haven't set up authentication yet, because editing a local text file shouldn't require a VIP pass.

### Examples

```bash
ask explain tail recursion with examples
ask "my dotnet application is telling me i do not have any SDK's installed when trying to build but i do have dotnet SDKs installed on my linux machine. what would cause this and how to fix it."
ask how does a hash table work
ask give me a regex to find all occurances of the word 'floor' but only if it is the last word of a sentence.
ask -m gemini-3.5-flash "explain this code < src/main.py"
```

## Configuration

Defaults are stored in `~/.config/ask/ask.conf`:

```bash
MODEL="big-pickle"
COST_THRESHOLD="2.00"
MODEL_PRICING_DATE="2026-07-01"
declare -A PRICING=(
  [big-pickle]="-- --"
  [gemini-3.5-flash]="1.50 9.00"
  ...
)
```

The config file is created automatically when you set a model or threshold for the first time. Pricing is refreshed daily when you run `ask -m`, so you don't have to memorize the current rates.

## Installation

### Clone and run

```bash
git clone https://github.com/emo333/ask.git
cd ask
chmod +x ask
./ask "ready"
```

### Install to PATH

```bash
git clone https://github.com/emo333/ask.git
sudo cp ask/ask /usr/local/bin/ask
```

### Shell alias

```bash
echo 'alias ask="$HOME/ask/ask"' >> ~/.bashrc
source ~/.bashrc
```

### Shell function

Add to `~/.bashrc` or `~/.zshrc`:

```bash
ask() {
  /path/to/ask/ask "$@"
}
```

## Development

1. Clone the repo
2. Edit the `ask` script
3. Test with:

   ```bash
   ./ask "does this work"
   ```

4. Submit a pull request (especially if you find issues)

No build system, no test suite, no ceremony — just a bash script and a dream.

## Known Issues

- If `auth.json` uses an unexpected structure, `jq` may fail to extract the key. JSON can be picky like that.
- Some models may return temporary provider-side errors such as `failover_exhausted`. These are not `ask` errors; retry or switch models and don't shoot the messenger.

## Potential Future Additions

- Local LLM hook

## License

Unlicensed. This project is released without any license or warranty.
