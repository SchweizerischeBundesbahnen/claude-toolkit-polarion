# Changelog

## [2.1.0](https://github.com/ariwk/claude-toolkit-polarion/compare/v2.0.0...v2.1.0) (2026-03-18)


### Features

* **core:** add claude-md skill to core plugin ([#17](https://github.com/ariwk/claude-toolkit-polarion/issues/17)) ([71ea785](https://github.com/ariwk/claude-toolkit-polarion/commit/71ea785f9b21497e3f3650f2d5d013051df0a56f)), closes [#16](https://github.com/ariwk/claude-toolkit-polarion/issues/16)

## [2.0.0](https://github.com/ariwk/claude-toolkit-polarion/compare/v1.1.0...v2.0.0) (2026-02-26)


### ⚠ BREAKING CHANGES

* **core:** Commands directory removed. Use skills instead (/precommit, /save-todos, /tox).

### Features

* **agents:** adopt minimal hazard-focused agent design across all plugins ([30c94fd](https://github.com/ariwk/claude-toolkit-polarion/commit/30c94fdecacb5a5be5be74e1e8340e10567f76a3))
* **core:** adopt Agent Skills pattern for commands ([35b67c3](https://github.com/ariwk/claude-toolkit-polarion/commit/35b67c3189cb5bde27214efdd08bd56e3089e8c6)), closes [#4](https://github.com/ariwk/claude-toolkit-polarion/issues/4)

## [1.1.0](https://github.com/ariwk/claude-toolkit-polarion/compare/v1.0.0...v1.1.0) (2026-02-03)


### Features

* **core:** redistribute CLAUDE.md in core plugin ([3dcdac1](https://github.com/ariwk/claude-toolkit-polarion/commit/3dcdac1867e618fcfbde769f87020d77edac2235))

## 1.0.0 (2026-02-01)


### Features

* add Context7 tools to deep-research-analyst agent ([837fa77](https://github.com/ariwk/claude-toolkit-polarion/commit/837fa77069d9657665cd8b36cbba975e68b3abda))
* add plugin with 12 agents, 3 commands, hooks, skills ([88e1dec](https://github.com/ariwk/claude-toolkit-polarion/commit/88e1dec9954033aac13dee72963acb47711523e5))
* add pre-commit configuration ([a74027e](https://github.com/ariwk/claude-toolkit-polarion/commit/a74027e8732826b4657d24fcb0b413a674259d62))
* add release-please workflow for automated releases ([44a0a71](https://github.com/ariwk/claude-toolkit-polarion/commit/44a0a71f61fb2b5219e3455cfe5178b7a511961d))


### Bug Fixes

* correct Context7 MCP tool names in all agents ([b34d9e2](https://github.com/ariwk/claude-toolkit-polarion/commit/b34d9e25f5da4cbbb3d8a5672f84b567e8287ea9))
* correct marketplace.json schema (add owner, fix source format) ([edb4859](https://github.com/ariwk/claude-toolkit-polarion/commit/edb4859376906fe40ee0146cfc8e5926efcbc5e6))
* include full operational guidance in CLAUDE.md ([e7936e7](https://github.com/ariwk/claude-toolkit-polarion/commit/e7936e74472fed15a85a8dbcca37a500b98e3193))
* move engineering principles from skill to CLAUDE.md ([88816b6](https://github.com/ariwk/claude-toolkit-polarion/commit/88816b6de9dcf69a4901e61031508ae1e93557a5))
