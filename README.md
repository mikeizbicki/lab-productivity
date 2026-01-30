# Lab: Productivity

This lab will review a handful of techniques to make you more productive in the terminal.
These tips should make your homeworks faster, easier, and more enjoyable.

As always, there's an xkcd for that:

<img src="https://imgs.xkcd.com/comics/is_it_worth_the_time_2x.png"  />

## Part 0: Vim Macros

> **NOTE:**
> This part0 is optional.
> It is for students who like vim and want to be exposed to the *real* power of vim.
> If you are struggling with basic vim movements,
> your time would be better spent skipping this part for now
> and working through the `vimtutor` command again after class.

A [macro](https://en.wikipedia.org/wiki/Macro_(computer_science)) is a piece of code that writes code for you.
One of the most powerful tools in vim is the ability to write macros.

In this task you will setup the `@p` macro for debugging python programs.

1. Clone this repo and cd into the resulting folder.

1. Open the file `p_macro` in Vim.
   You should see contents that look like
   ```
   ^y$iprint("^[A=", ^[pa)^[^
   ```
   This is the "source code" for the macro.
   It represents a series of keystrokes (i.e. vim commands) that will automatically happen when you trigger the macro.
   The `^[` characters should appear in a slightly different color, and if you move your cursor over them, you'll notice they behave like a single character.
   This is how the `Esc` key gets rendered in the terminal, so each of these characters will cause the `Esc` key to be pressed.

1. Copy the line into the `p` register by typing the following sequence of commands in normal mode.
   (Ensure that you are in normal mode by pressing `Esc` before typing the commands.)
   ```
   "pyy
   ```
   The `yy` yanks (Vim's language for copying) the entire line,
   and the `"p` indicates that we are yanking into the `p` register (Vim's language for clipboard).
   Your typical muggle text editor has only a single clipboard to copy/paste from, but Vim has a separate register for every key on the keyboard.
   This lets us copy/paste many different things at the same time.
   Macros use the same registers as yanking/pasting, so by yanking into the `p` register we are also creating the `p` macro.
   
1. To ensure that your macro works, open a new tab with the command
   ```
   :tabe
   ```
   You can use the commands `gt` and `gT` to switch between tabs.

   In your new tab, type
   ```
   python_variable_name
   ```
   into the tab.
   With your cursor anywhere on the line, type `@p` to activate the macro.
   If you've created the macro correctly, you should get the result
   ```
   print("python_variable_name=", python_variable_name)
   ```
   This macro makes debugging python programs much easier.

1. (optional) For a detailed reference on writing your own Vim macros, see <https://vim.fandom.com/wiki/Macros>.
    These are quite useful for automating lots of small repetitive tasks.

## Part 1: Update `.bashrc`

Recall that `rc` stands for "run commands" and files that end in `rc` will be automatically *sourced* when the respective program starts.
For example, the `.bashrc` file will automatically be sourced whenever the bash shell starts
(which happens every time you login to the lambda server).
You can make your environment more comfortable by modifying the `.bashrc` file.

A simple way to modify `.bashrc` programmatically is with output redirection.
Run the following command
```
$ echo 'echo "Have a nice day :)"' >> .bashrc
```

> **Note:**
> It is common to write shell code that writes shell code like this.
> (And even shell code that writes shell code that writes shell code... and so on...)
> Lots of subtle bugs come about due to incorrect use of quotation marks.

Verify that you've modified the file by running
```
$ tail ~/.bashrc
```

Now logout and login again.
You should see the welcome message.

There are many routine tasks that we would like automated.
(And your `.bashrc` file is already very large, automating many of them.)
One task that is not currently being automated is loading up your venv that stores your python programs.

> **Exercise:**
> Modify your `.bashrc` so that:
> (1) the previous echo command is removed, and
> (2) your venv is automatically activated.
> Recall that `G` is the Vim command to move to the end of a file, `dd` deletes the current line, and `cc` changes the current line.

> **Note:**
> Notice that the exercise above does not ask you to verify that the changes were successful.
> It should be "obvious" to you how to verify if the command was successful.
> Tutorials generally assume that you are self-aware enough as a programmer to verify "obvious" steps like this.
> I will begin omitting these "obvious" verification steps in the future,
> and the real-world tutorials we will soon use will not have them either.

## Part 2: Better LLM Outputs

Recall that we previously installed and setup the `llm` command for using AI models from the command line.
And now that you are automatically loading the `venv` in your `.bashrc`, you will always have access to this command.
In this section, we will see how to make the command even more powerful.

### Part 2a: System Prompts

The `-s` flag allows us to specify the *system prompt* for the llm.
This is a set of instructions that guide how the model will respond to your query.
For example, try running the following command:
```
$ llm -s 'talk like a pirate' 'why is the shell useful for big data?'
```

By writing a good system prompt tailored to your system and set of problems,
you can get much better answers from an LLM that you would get from web interfaces like <https://chatgpt.com>.

> **NOTE:**
> A type of hacking called prompt injection can be used to extract the system prompt from any LLM.
> [Here is an example reddit post](https://www.reddit.com/r/PromptEngineering/comments/1j5mca4/i_made_chatgpt_45_leak_its_system_prompt/) explaining how to get the system prompt for ChatGPT.
> You can view the anthropic system prompts at [in the official documentation](https://platform.claude.com/docs/en/release-notes/system-prompts).
>
> Anthropic also has a great guide for [prompt engineering](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) that is worth browsing.

Here is an example of an LLM command with a good system prompt for this class:
```
$ llm -s "
Keep your response short, between 1-20 lines. Focus on a high signal to noise ratio.

If the question is about a computer, respond for the following system details:
- operating system: $(uname -a).
- bash version: $(bash --version | head -n1).
- python version: $(python3 --version).
- sqlite3 version: $(sqlite3 --version).
- git version: $(git --version).
I do not have root access.
" 'how do I install flake8?'
```
The output should be simple, easy to read, and something that you can immediately run on your system and get it to work.
(If you type "how do I install flake8?" into <https://chatgpt.com>, you will get answers related to installing on Windows or Mac machines, or answers that assume you have root access on Linux; none of that is applicable for you, however.)

But the command is obnoxiously long and hard to write.
One idea to simplify it is to use a bash variable to store the system prompt:
```
$ SYSTEM_PROMPT=$(cat <<EOF
Keep your response short, between 1-20 lines. Focus on a high signal to noise ratio.

If the question is about a computer, respond for the following system details:
- operating system: $(uname -a).
- bash version: $(bash --version | head -n1).
- python version: $(python3 --version).
- sqlite3 version: $(sqlite3 --version).
- git version: $(git --version).
I do not have root access.
EOF
)
$ llm -s "$SYSTEM_PROMPT" 'how to install flake8?'
```
This is much better, but we can do even better still by using a [bash function](https://www.w3schools.com/bash/bash_functions.php).
The syntax is shown below.
```
$ myllm() {
    SYSTEM_PROMPT=$(cat <<EOF
Keep your response short, between 1-20 lines. Focus on a high signal to noise ratio.

If the question is about a computer, respond for the following system details:
- operating system: $(uname -a).
- bash version: $(bash --version | head -n1).
- python version: $(python3 --version).
- sqlite3 version: $(sqlite3 --version).
- git version: $(git --version).
I do not have root access.
EOF
    )
    llm -s "$SYSTEM_PROMPT" -m groq-llama-3.3-70b "$@"
}
$ myllm 'how to install flake8?'
```

Before we add this new command to your `.bashrc` file,
we will make even more usability improvements.

### Part b: Coloring Output

[ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code) are the standard way to achieve graphical effects in the terminal.
For example the ANSI escape sequence `\033[94m` means to change the text color to light blue.
Try the following command:
```
$ echo -e "hello \\033[94mworld"
```
You should see hello in black and world in blue.

Notice:
1. The command above has a double backslash `\\`.
    The ANSI escape sequence contains a literal backslash character,
    and so we need to escape the backslash so that the shell doesn't interpret `\0` as the [null byte](https://en.wikipedia.org/wiki/Null_character).
2. The `-e` flag is needed for echo to interpret the ANSI escape codes.
    Try to see what happens without `-e`.

I like to put the output of my llm calls in light blue,
so that they are easy to identify when scrolling through my shell history.
The following bash function wraps the output of `llm` in blue.
```
llm_blue() {
    # printf is like echo,
    # but does not add a newline (\n) to the output automatically
    printf "\\033[94m"
    llm "$@"
    printf "\\033[0m"
}
```
Then you can redefine the `myllm` function to call `llm_blue` instead of `llm` at the end.
I also call my function name `groq` because it calls out to the `groq` API endpoint.
Here is the full function that I use:
```
groq() {
    SYSTEM_PROMPT=$(cat <<EOF
Keep your response short, between 1-20 lines. Focus on a high signal to noise ratio.

If the question is about a computer, respond for the following system details:
- operating system: $(uname -a).
- bash version: $(bash --version | head -n1).
- python version: $(python3 --version).
- sqlite3 version: $(sqlite3 --version).
- git version: $(git --version).
I do not have root access.
EOF
)
    llm_blue -s "$SYSTEM_PROMPT" -m groq-llama-3.3-70b "$@"
}
```
If you copy/paste it into the terminal, you should see colored output when you run the command `groq`.

> **Exercise:**
> `lolcat` is a command that uses ANSI escape codes to make text rainbowy.
> To see an example, try running:
> ```
> $ llm 'what is bash' | lolcat
> ```
> If you enjoy colors, modify your `groq` function to use `lolcat` instead of `llm_blue`.

> **Exercise:**
> Try modifying the `SYSTEM_PROMPT` variable to get an output that you like
> For example, maybe you like more detailed responses than me and want the output to be between 10-30 lines instead of 1-20.
> (I have a more detailed prompt that I use for my `claude` command that you can find in the Part2c section below.)

> **Exercise:**
> Add your `groq` function to your `.bashrc` file. 

### (Optional) Part 2c: Writing a function for claude opus

Anthropic's claude Opus series of models is widely regarded as the best coding model these days.
Unlike groq, however, it costs money.
In this optional task, we will get a nice LLM interface to claude-opus setup.

I recommend you get a claude API key for this class and buy $10 worth of credits.
As we will soon see, using claude from the command line is *very cheap*.
If you are subscribing to any monthly AI services (either OpenAI or Anthropic),
you are probably way overpaying.
You are also getting much worse answers (because you are using generic system prompts instead of specialized system prompts).

My `claude` function is similar to my `groq` function above,
but it has additional code for computing the amount of money I paid for each query.
A typical query costs less than a penny on the latest state of the art models.
The code is shown below:
```
function claude() {(
    # different model names and prices can be found at
    # <https://platform.claude.com/docs/en/about-claude/pricing>
    # costs are specified in dollars per million tokens
    model=anthropic/claude-opus-4-5-20251101
    cost_input=5
    cost_output=25

    # older/cheaper model info 
    #model=anthropic/claude-sonnet-4-0
    #cost_input=3
    #cost_output=15

    SYSTEM_PROMPT=$(cat <<EOF
Keep your response short, between 1-20 lines. Focus on a high signal to noise ratio. Your target audience is people with phds in math and computer science.

If the question is about a computer, respond for the following system details:
- operating system: $(uname -a).
- bash version: $(bash --version | head -n1).
- python version: $(python3 --version).
- sqlite3 version: $(sqlite3 --version).
- git version: $(git --version).
- docker version: $(docker --version).

When asked to implement a function:
- only output code, no markdown
- write comments that:
    - explain the underlying algorithm
    - do NOT explain minor syntax/library details
    - are stand-alone and do not require context of the input question to understand
- when writing a python function from scrach:
    - provide a full docstring and doctests
- when modifying an existing function:
    - do not repeat the existing docstring
EOF
)

    # Capture stderr while preserving stdout;
    # we will extract the number of tokens from stderr in order to compute the price we paid
    local stderr_file=$(mktemp)
    trap "rm -f '$stderr_file'" EXIT
    llm_blue -s "$SYSTEM_PROMPT" -m "$model" "$@" -u 2>"$stderr_file"
    stderr_content=$(cat "$stderr_file")
    local exit_code=$?
    latest_cid=$(llm logs list -n 1 --json | jq -r '.[] | .conversation_id' 2>/dev/null)
    # NOTE:
    # There is a minor race condition here.
    # The llm logs command above extracts the cid of the most recent conversation, which is almost certainly the conversation from the llm_blue call above.
    # But it is possible that a concurrently running llm process terminates after the llm_blue and before llm logs.
    # This shouldn't be a major concern in practice because this function is designed to be run interactively by a user and not inside a script.

    # automatically copy the output to the clipboard
    echo "$stderr_content" | xclip -selection clipboard

    # Try to extract token usage from stderr
    if [[ $stderr_content =~ Token\ usage:\ ([0-9,]+)\ input,\ ([0-9,]+)\ output ]]; then
        input_tokens="${BASH_REMATCH[1]//,/}"
        output_tokens="${BASH_REMATCH[2]//,/}"
        cost_input=$(echo "scale=10; $cost_input * $input_tokens / 1000000" | bc -l)
        cost_output=$(echo "scale=10; $cost_output * $output_tokens / 1000000" | bc -l)
        cost_total=$(echo "scale=10; $cost_input + $cost_output" | bc -l)
        printf "cost: $%.4f (input: $%0.4f, output: $%0.4f) --cid=$latest_cid\n" "$cost_total" "$cost_input" "$cost_output" >&2
    else
        # If pattern doesn't match, display the stderr content
        if [[ -n $stderr_content ]]; then
            echo "$stderr_content" >&2
        fi
        input_tokens=""
        output_tokens=""
    fi

    return $exit_code
) # the function is enclosed in a subshell to trigger the TRAP for cleanup
}
```
I recommend adding the `claude` function to your `.bashrc`.
You may want to adjust the `SYSTEM_PROMPT` variable slightly.

## Part 4: Even better LLM support

Simon Willison (the author of `llm`) has a number of other command line utilities for making better use of llms.
Two useful python libraries are `ttok` and `files-to-prompt`.
Install both libraries.
(If you forget how, ask `groq`.)

> **NOTE:**
> Simonw has a [great blog](https://simonwillison.net/).
> He is one of the best sources on the latest developments in AI.
> He's also independently wealthy (successful exit from a startup),
> and doesn't work for a company attached to an AI product,
> and so has fairly unbiased analysis.
> I recommend browsing through his blog at some point.

We will use these utilities to automate coding for us.
It will be more powerful (and cheaper!) than vibe-coding tools like [Claude Code](https://code.claude.com/docs/en/overview) or [OpenAI Codex](https://openai.com/codex/).

Clone last week's homework and cd into the repo.
```
$ git clone https://github.com/mikeizbicki/continuous-integration
$ cd continuous-integration
```
Make sure you cloned my repo (and not your repo) because we'll want the code to still be broken.

The `files-to-prompt` command takes a list of files as input and outputs their context in a format suitable for an llm prompt.
Try the following variations:
```
$ files-to-prompt README.md
$ files-to-prompt Fixme.py
```
Recall that `.` is a special folder name that refers to the current working directory.
When we pass a folder into `files-to-prompt`,
it will print out all the files in the folder.
```
$ files-to-prompt .
```

The `ttok` function can be used to measure the number of tokens in the input stream.
Recall that groq's free API endpoint limits your input tokens to 8096,
and other APIs pay-per-token.
So it can be useful to know the number of tokens in a prompt before actually invoking the llm.

We can count the number of tokens return by `files-to-prompt` with the following pipeline.
```
$ files-to-prompt . | ttok
2936
```
That's not very many, so we can pass it to groq/claude just fine with one of the following commands:
```
$ groq "$(files-to-prompt .)"
$ claude "$(files-to-prompt .)"
```
Notice that the llm reads the `README.md` file to understand the instructions, and then provides the correct output by modifying `Fixme.py` for you.

Claude (being state-of-the-art) is a bit better than groq and will also highlight the PEP8 errors.
Again, it knows to do this because of the instructions in the `README.md` file.

We could also run a command like the following to get groq/claude to explain to us the output of the flake8 command if we found it confusing:

```
$ groq "$(flake8 .)"
$ claude "$(flake8 .)"
```

Observe that you have a MUCH more powerful tool than the web interfaces to AI like <https://chatgpt.com>,
and it is free (groq) or basically free (claude cost me $0.02 to run the commands above).

## Part 5: Real world practice

Use the techniques you've learned above to quickly finish the lab posted to <https://github.com/mikeizbicki/lab-cat>.
