# ttt

Logs metadata for shell commands; an extension to my shell history.

This is essentially a wrapper script to store metadata. The wrapper script `ttt` takes any other command as input, stores some metadata about it in a history file, and then runs the command as normal. Its quite literally:

```
#!/bin/sh
tttlog "$@"
exec "$@"
```

## But Why?

Shell history is nice, but it doesn't provide me context; like what directory was this command run in?

Additionally, not all commands are saved in shell history. If you have some keybinding to launch a script/application, or force-quit the terminal without exiting, your commands don't get saved in your shell history.

This gives me finer control on what gets logged, so I can do analysis on it later from [`HPI`](https://github.com/seanbreckenridge/HPI#readme).

## How?

This consists of:
  - `ttt` (the wrapper shell script)
  - `tttlog` (log metadata to the history file)

The point here is to be transparent and easy to add. So, at the top of any script which I want to log, I add the line:

```
command -v tttlog >/dev/null 2>&1 && tttlog "$(basename "$0") $@"
```

If I'm launching the command with a keybinding or from another program that accepts a command as input (e.g. [`rifle`](https://github.com/ranger/ranger) (my file manager) or from my [`window manager`](https://i3wm.org/), I'd modify the line like:

```
# launch firefox
bindsym $mod+f exec firefox-developer-edition
```

... to ...

```
bindsym $mod+f exec ttt firefox-developer-edition
```

That *does* mean that the command would fail if `ttt` isn't installed, but I'm willing to live with that.

If this can't find the current working directory (i.e. if you deleted the current directory and called it from there), it sets `-`, which signifies an error.

I recommend you symlink `#!/bin/sh` to something faster than `bash`, like `dash`, to improve startup times. See [`here`](https://wiki.archlinux.org/index.php/Dash) for more info.

As an example of what this logs to the CSV file:

```
1599517118,/home/sean/Repos/ttt,printf hello
1599517125,/home/sean/Repos/ttt,"some multi
line command"
```

One could also just use this to log generic events. `tttlog` just saves whatever arguments you pass it with some metadata about where/when, so could be used to track habits/my behaviour like:

```
#!/bin/sh

# every minute, if I'm watching something, save what movie/music I'm listening to
# using my https://github.com/seanbreckenridge/mpv-sockets script

while true; do
  if MEDIA_PATH="$(mpv-currently-playing)"; do
    tttlog "mpv_playing_media:$MEDIA_PATH"
  fi
  sleep 60
done
```


## Install

Download and install the scripts!

Requires `go`

```bash
# create temp dir
mkdir /tmp/install-ttt
cd /tmp/install-ttt
# download the scripts
wget "https://raw.githubusercontent.com/seanbreckenridge/ttt/master/tttlog.go"  "https://raw.githubusercontent.com/seanbreckenridge/ttt/master/ttt"
chmod +x ./ttt
# copy ttt onto your $PATH
cp ./ttt ~/.local/bin
go install ./tttlog.go
```

You can change which file `tttlog` writes to by setting the `TTT_HISTFILE` environment variable.

### Tests

```
cd test
./test
```

### Benchmarks

Just to have some numbers here, the overhead that `ttt` (wrapper shell script running `ttt`) causes is about `3ms`

```
Benchmark #1: /bin/sh
  Time (mean ± σ):       0.6 ms ±   0.3 ms    [User: 0.5 ms, System: 0.4 ms]
  Range (min … max):     0.0 ms …   1.5 ms    1621 runs

Benchmark #2: printf hi >/dev/null
  Time (mean ± σ):       0.1 ms ±   0.1 ms    [User: 0.2 ms, System: 0.2 ms]
  Range (min … max):     0.0 ms …   1.3 ms    1935 runs

Benchmark #3: /home/sean/Repos/ttt/test/../ttt printf hi >/dev/null
  Time (mean ± σ):       3.8 ms ±   0.7 ms    [User: 2.3 ms, System: 2.3 ms]
  Range (min … max):     2.3 ms …   5.8 ms    522 runs
```

#### ttt?

Naming things relating to shell history/logging is hard...

I just wanted something that would be easy to type, so I can add it to wherever I want quickly. Is just the first thing that came to mind, when I was trying to figure out how to track rifle/i3 bindsyms; 'track the things'

