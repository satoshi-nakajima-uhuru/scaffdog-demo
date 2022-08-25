---
name: 'template'
root: '.'
output: '../../'
ignore: []
questions:
  name: 'Please enter a app name.'
---

# `{{ inputs.name | lower }}/specification.md`

```md
# 仕様

## 目的

## 方針

## 機能

{{ inputs.name | slice 0 15 }}

```
