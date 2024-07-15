# LRZSZ SUPPORT

```sh
brew install laggardkernel/tap/iterm2-zmodem
```

Profiles -> Advanced -> Trigger 

| Regular Expression  | Action               | Parameters                        | Instant |
| ------------------- | -------------------- | --------------------------------- | ------- |
| /*/*B0100           | Run Silent Coprocess | /usr/local/bin/iterm2-zmodem-send | √       |
| /*/*B00000000000000 | Run Silent Coprocess | /usr/local/bin/iterm2-zmodem-recv | √       |

