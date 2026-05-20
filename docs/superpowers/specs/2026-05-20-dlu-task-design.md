# DLU Task Review and Add Design

## Summary
This design covers reviewing and adding a login task for Dalian University (大连大学) based on the current temp/default (4).json into the tasks index.

## Goals
- Add a DLU task with id "dlu" to tasks/ and both index files
- Keep existing step logic unchanged while ensuring required metadata and failure handling are present
- Comply with submit-task.md safety and formatting rules

## Non-goals
- No new automation logic or heuristics
- No device-type inference
- No success_conditions usage

## Inputs and constraints
- Source file: temp/default (4).json
- School: 大连大学
- Device: 未知 (do not include in description)
- Author: 2476449565
- id/file name: dlu (lowercase)

## Safety review
- Steps are only input/click/sleep; no eval/custom_js; no external data exfiltration

## Task JSON changes
- id: dlu
- name: 大连大学校园网登录
- description: 适用于大连大学校园网认证页面
- metadata:
  - author: 2476449565
  - school: 大连大学
  - device: 未知
- url: {{LOGIN_URL}}
- timeout: 30000
- steps: unchanged
- add on_success: {"message":"登录成功"}
- add on_failure: {"message":"登录失败，请检查账号密码或认证地址","screenshot": true}
- remove success_conditions (deprecated)

## File operations
- Create tasks/dlu.json from temp/default (4).json with the updates above
- Remove temp/default (4).json after promotion
- Append index entries to index.json and index.gitee.json:
  - id: dlu, name/description as above
  - tags: ["大连大学"]
  - author: 2476449565
  - version: 1.0.0
  - url: raw GitHub/Gitee paths to tasks/dlu.json

## Validation
- JSON parses and uses 2-space indentation
- id matches filename and index entry
- index.json and index.gitee.json remain valid JSON
