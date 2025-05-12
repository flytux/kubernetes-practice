### install starship

- wget https://github.com/starship/starship/releases/download/v1.23.0/starship-x86_64-unknown-linux-gnu.tar.gz

### init startship

```
vi ~/.bashrc

eval "$(starship init bash)" 
```

### config starship

```
❯ cat ~/.config/starship.toml 
format = """
$username\
$hostname\
$localip\
$kubernetes\
$directory\
$git_branch\
$git_commit\
$git_state\
$git_metrics\
$git_status\
$package\
$helm\
$java\
$gradle\
$memory_usage\
$direnv\
$env_var\
$line_break\
$time\
$status\
$shell\
$character"""



~/workspace/demo is 📦 v0.0.1-SNAPSHOT via ☕ v21.0.7 via 🅶 v8.13 
❯ ls
bin  build  build.gradle  gradle  gradlew  gradlew.bat  HELP.md  settings.gradle  src
```
