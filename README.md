# Shell History
Inspired by https://github.com/bamos/zsh-history-analysis.

Visualize your usage of Bash/Zsh through a web app
thanks to Flask and Highcharts.

- [Installation](#installation)
- [Dependencies](#dependencies)
- [Usage](#usage)
- [How it looks](#how-it-looks)
- [How it works](#how-it-works)
- [History file format](#history-file-format)
- [Chart ideas](#chart-ideas)
- [License](#license)

## Installation
Clone the repo with `git clone https://github.com/Pawamoy/shell-history`.

`shell-history` needs a lot of info to be able to display various charts.
The basic shell history is not enough. In order to generate the necessary
information, you have to source the `shellhistory.sh` script.

It will append your commands in a second file in your home:
`.shell_history/history`.

**Your shell history configuration will not be modified.**

Only two things will be modified in your shell environment:
- your `PROMPT_COMMAND` variable for Bash, or your `precmd` function for Zsh,
- the trap on `DEBUG` signal.

So, at shell startup, in `.bashrc` or `.zshrc`, put a line like the following:

```bash
# make sure nothing is prepended to PROMPT_COMMAND or precmd after this line
. '/path/to/shell-history/shellhistory.sh'
```

... and now use your shell normally!

## Dependencies
To launch the web app, you will need to install some Python packages:
`pip install -r requirements.txt`.

Python 2 is not supported right now, so unless your system Python is version 3
or above, install them in a Python 3 virtualenv with:

```bash
sudo apt-get install python-virtualenv
virtualenv -p python3 venv
. venv/bin/activate
pip install -r requirements.txt
```

You can specify the path to your virtualenv in the variable `SHELLHISTORY_VENV`
(by default `venv` directory will be used, if it exists).

You will also need Internet connection since assets are not bundled.

## Usage
Simply `./run.sh`, or run it manually with `FLASK_APP=app.py flask run`.
Now go to http://127.0.0.1:5000/ and enjoy!

For more convenience, you can put a script in your PATH, for example in
`/usr/bin/`, with the following contents:

```bash
#!/usr/bin/env bash

export SHELLHISTORY_VENV=/path/to/my/virtualenv
export SHELLHISTORY_FILE=/path/to/my/history_file
export SHELLHISTORY_DB=/path/to/my/history_db

cd /path/to/shellhistory || exit 1
./run.sh
```

## How it looks
![length chart](pictures/length.png)
![markov chart](pictures/markov.png)
![daily chart](pictures/daily.png)
![type chart](pictures/type.png)

## How it works
In order to append a line each time a command is entered, the `PROMPT_COMMAND`
variable and the `precmd` function are used, respectively for Bash and Zsh.
They allow us to execute arbitrary instructions just before command execution,
or before the command prompt is displayed, meaning, just after the last command
has returned.

This is where we compute the start and stop time, return code, working
directory and command type, and append the line into our history file.
Start and stop time are obtained with `$(date '+%s%N')`, return code is passed
directly with `$?`, working directory is obtained with `$PWD` and command
type with `$(type -t arg)` for Bash and `$(type -w arg)` for Zsh.

Values for UUID, parents, hostname, and TTY are computed only once, when
`shellhistory.sh` is sourced. Indeed they do not change during usage of the current
shell process. Hostname and TTY are obtained through commands `$(hostname)` and
`$(tty)`. UUID is generated with command `$(uuidgen)`. Also note that UUID
is exported in subshells so we know which shell is a subprocess of another, and
so we are able to group shell processes by "sessions", a session being an opened
terminal (be it a tab, window, pane or else). Parents are obtained with a
function that iteratively greps `ps` result with PIDs (see `shellhistory.sh`).

Values for user, shell, and level are simply obtained through environment
variables: `$USER`, `$SHELL`, and `$SHLVL`.

Start time is computed just before the entered command is run by the shell,
thanks to a trap on the DEBUG signal. From Bash manual about `trap`:

>If a sigspec is DEBUG, the command arg is executed before every simple command.

The last command is obtained with the command `fc`. It will feel like your
history configuration is mimic'd by the extended history. If the commands
beginning with a space are ignored, `shell-history` will notice it and will not
append these commands. Same for ignored duplicates. If you enter an empty line,
or hit Control-C before enter, nothing will be appended either. The trick behind
this is to check the command number in the current history (see `shellhistory.sh`
for technical details). Note however that if you type the same command in an
other terminal, it will still be appended, unless you manage to synchronize your
history between terminals.

## History file format
Fields saved along commands are start and stop timestamps, hostname, username,
uuid (generated), tty, process' parents, shell, shell level, command type,
return code, and working directory (path), in the following format:
`:start:stop:uuid:parents:host:user:tty:path:shell:level:type:code:command`.

- multiline commands are prepended with a semi-colon `;` instead of a colon `:`,
  starting at second line
- start and stop timestamps are in microseconds since epoch
- process' parents and working directory are encoded in base64 to avoid
  delimiter corruption

Example (multiline command):

```
:1510588139930150:1510588139936608:40701d9b-1807-4a3e-994b-dde68692aa14:L2Jpbi9iYXNoCi91c3IvYmluL3B5dGhvbiAvdXNyL2Jpbi94LXRlcm1pbmFsLWVtdWxhdG9yCi91c3IvYmluL29wZW5ib3ggLS1zdGFydHVwIC91c3IvbGliL3g4Nl82NC1saW51eC1nbnUvb3BlbmJveC1hdXRvc3RhcnQgT1BFTkJPWApsaWdodGRtIC0tc2Vzc2lvbi1jaGlsZCAxMiAyMQovdXNyL3NiaW4vbGlnaHRkbQovc2Jpbi9pbml0Cg==:myhost:pawamoy:/dev/pts/1:L21lZGlhL3Bhd2Ftb3kvRGF0YS9naXQvc2hlbGxoaXN0Cg==:/bin/bash:1:builtin:0:echo 'a
;b
;c' | wc -c
```

## Chart ideas
You can post your ideas in this issue: https://github.com/Pawamoy/shell-history/issues/9.

## License
Software licensed under the
[ISC](https://www.isc.org/downloads/software-support-policy/isc-license/)
license.
