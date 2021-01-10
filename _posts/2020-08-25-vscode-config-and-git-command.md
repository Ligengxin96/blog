---
title: '常用vscode配置和git命令备忘'
tags:
  - tools
  - tutorial
categories:
  - tutorial
---
&emsp;&emsp;记录下需要容易忘记但又常用的vscode配置和git命令.

 
## Git 命令行

### 撤销一次Pull 操作

`git revert --hard head`

### 删除分支

- 删除远程分支 `git push origin -d <branchName>`
- 删除本地分支 `git branch -d <branchName>`

## VScode 配置

### 调试Jest测试框架的某个单元测试TS文件

```json 
{
  "type": "node",
  "name": "Debug Jest Test",
  "request": "launch",
  "args": [
    "--runInBand"
  ],
  "cwd": "${workspaceFolder}",
  "console": "integratedTerminal",
  "internalConsoleOptions": "neverOpen",
  "disableOptimisticBPs": true,
  "program": "${workspaceFolder}/node_modules/jest/bin/jest"
}
```

### 调试Mocha测试框架的某个单元测试TS文件

```json 
{
  "type": "node",
  "request": "launch",
  "name": "Debug Mocha Test",
  "program": "${workspaceFolder}/node_modules/mocha/bin/_mocha",
  "args": [
    "-r",
    "ts-node/register",
    "--timeout",
    "999999",
    "--colors",
    "${workspaceFolder}/${relativeFile}"
  ],
  "env": { "TS_NODE_COMPILER_OPTIONS": "{\"module\": \"commonjs\"}" },
  "envFile": "${workspaceFolder}/../.env",
  "protocol": "inspector"
}
```

### 调试Mocha测试框架的某个单元测试JS文件

```json 
{
  "type": "node",
  "request": "launch",
  "name": "Mocha Current File",
  "program": "${workspaceFolder}/node_modules/mocha/bin/_mocha",
  "args": ["--timeout", "999999", "--colors", "${file}"],
  "console": "integratedTerminal",
  "internalConsoleOptions": "neverOpen"
}
```

### 调试Mocha测试框架的某个文件夹的所以单元测试JS文件

```json 
{
  "type": "node",
  "request": "launch",
  "name": "Mocha All",
  "program": "${workspaceFolder}/node_modules/mocha/bin/_mocha",
  "args": ["--timeout", "999999", "--colors", "${workspaceFolder}/test"],
  "console": "integratedTerminal",
  "internalConsoleOptions": "neverOpen"
}
```

### 调试当前TS文件

```json 
{
  "type": "node",
  "request": "launch",
  "name": "Debug Mocha Test",
  "program": "${workspaceFolder}/node_modules/mocha/bin/_mocha",
  "args": [
    "-r",
    "ts-node/register",
    "--timeout",
    "999999",
    "--colors",
    "${workspaceFolder}/${relativeFile}"
  ],
  "env": { "TS_NODE_COMPILER_OPTIONS": "{\"module\": \"commonjs\"}" },
  "envFile": "${workspaceFolder}/../.env",
  "protocol": "inspector"
}
```

### 调试当前JS文件

```json 
{
  "type": "node",
  "request": "launch",
  "name": "Debug Mocha Test",
  "program": "${workspaceFolder}/node_modules/mocha/bin/_mocha",
  "args": [
    "-r",
    "ts-node/register",
    "--timeout",
    "999999",
    "--colors",
    "${workspaceFolder}/${relativeFile}"
  ],
  "env": { "TS_NODE_COMPILER_OPTIONS": "{\"module\": \"commonjs\"}" },
  "envFile": "${workspaceFolder}/../.env",
  "protocol": "inspector"
}
```

### continuous update...
