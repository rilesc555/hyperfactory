# Rules

## These rules apply to all files in this repository

### Context folder
* do not edit anything in the context folder. It is there just for reference.
* do not add any new files in the context folder. It is there just for reference.
* do not delete any files in the context folder. It is there just for reference.

### Documentation
* document all changes to the spec in the spec.md file.
* Keep the docs folder up to date with in-depth explanations of everything in the project.
* Document every change
* Before documenting a change, make sure there are no duplicates in the docs folder and make sure you are not adding duplicates. Look for related docs and update them instead of creating new ones.
* Organize the docs folder in to subfolders as needed.
* when editing markdown files, enforce a markdown lint
* document names are in kebab-case

### Spec
* The spec is the source of truth for the project
* The spec is the single source of truth for the project
* The spec is the only source of truth for the project
* The spec folder is named `spec`
* The spec folder has different files for different aspects of the project.
* Each file in the spec folder has a decided and an undecided section. The decided section contains the spec that has been decided on. The undecided section contains questions that still need to be answered.



### Development
* Development is happening in WSL, so use Linux commands
* Use Context7 when I need code generation, setup or configuration steps, or library/API documentation. This means you should automatically use the Context7 MCP tools to resolve library id and get library docs without me having to explicitly ask.

