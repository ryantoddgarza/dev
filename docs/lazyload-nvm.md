# Lazyload NVM

The root cause of the problem is that the nvm initialisation script in `zshrc` takes about 500ms to run. Several workarounds have been proposed to alleviate the issue.

## Setup profiling

Zsh offers a [profiling module](http://zsh.sourceforge.net/Doc/Release/Zsh-Modules.html) that can be implemented in _zshrc_.

Alternatively, Armno Prommarak proposes a simple command in his [article on the subject](https://armno.in.th/2020/08/24/lazyload-nvm-to-reduce-zsh-startup-time/). While slighly less verbose, it's fine for our purposes.

```sh
for i in $(seq 1 10); do /usr/bin/time zsh -i -c exit; done
```

## Method 1

```sh
# Postpone nvm script load
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" --no-use

# Prepend global calls
function node npm yarn {
  unfunction node npm yarn
  nvm use default
  command "$0" "$@"
}
```

Pros:

- Concise

Cons:

- Still slower than other methods

## Method 2

```sh
export NVM_DIR=~/.nvm

# Add every binary that requires nvm, npm or node to run to an array of node globals
NODE_GLOBALS=(`find ~/.nvm/versions/node -maxdepth 3 -type l -wholename '*/bin/*' | xargs -n1 basename | sort | uniq`)
NODE_GLOBALS+=(node nvm)

# Load nvm script
load_nvm () {
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
}

# Node global call triggers script loading
for cmd in "${NODE_GLOBALS[@]}"; do
  eval "${cmd}(){ unset -f ${NODE_GLOBALS}; load_nvm; ${cmd} \$@; }"
done
```

Pros:

- Fast

Cons:

- Some globals break (need to debug)
