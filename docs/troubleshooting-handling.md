

## ERRORS

### START TEST
**Running on linux**

```shell
$ /bin/bash test-em-all.bash
test-em-all.bash: line 13: $'\r': command not found
test-em-all.bash: line 14: syntax error near unexpected token `$'{\r''
'est-em-all.bash: line 14: `function assertCurl() {

```

#### REASON
- **\r** new line char is not compatible 
  - remove **\r** and run again in linux terminal
