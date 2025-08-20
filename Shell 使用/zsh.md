
```bash
local user_host="%B%(!.%{$fg[red]%}.%{$fg[green]%})%n%{$reset_color%} "
PROMPT="╭─${conda_prompt}${user_host}${current_dir}${rvm_ruby}${vcs_branch}${venv_prompt}${kube_prompt}
╰─%B${user_symbol}%b "
RPROMPT="%B${return_code}%b"
```