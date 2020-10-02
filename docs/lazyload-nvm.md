# Lazyload NVM

## Fix nvm slowing down terminal initialisation

The root cause of the problem is that the nvm initialisation script that gets added to the `.zshrc` file on install takes about 500ms to run. The following alleviates the issue.

```sh
export NVM_DIR=~/.nvm

# Add every binary that requires nvm, npm or node to run to an array of node globals
NODE_GLOBALS=(`find ~/.nvm/versions/node -maxdepth 3 -type l -wholename '*/bin/*' | xargs -n1 basename | sort | uniq`)
NODE_GLOBALS+=(node nvm)

# Lazy-loading nvm + npm on node globals call
load_nvm () {
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
}

# Making node global trigger the lazy loading
for cmd in "${NODE_GLOBALS[@]}"; do
  eval "${cmd}(){ unset -f ${NODE_GLOBALS}; load_nvm; ${cmd} \$@; }"
done
```
