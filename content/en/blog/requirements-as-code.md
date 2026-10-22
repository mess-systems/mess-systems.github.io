---
title: "Requirements as code"
date: 2026-10-22
weight: 2
description: "Why keep requirements in Markdown next to the code, check them in CI and trace them to tests, and how the mess-systems/requirements template does it."
---

Requirements usually live apart from the code: in a wiki, a spreadsheet, a task tracker. While the team is small and remembers everything, that works. Then questions appear that are hard to answer: which requirement is implemented, where exactly, is there a test for it, who changed the wording and when.

When AI agents join the work, there are more questions. An agent writes code quickly, but it needs to know which requirement it is closing, and it must not be the one deciding that a requirement is done.

We published the [mess-systems/requirements](https://github.com/mess-systems/requirements) template, where requirements are kept the same way as code. Below is how it works and why it is built this way. Its examples are written for the fictional product `x` of the company mess.systems.

## One requirement, one file

Each requirement is a Markdown file with a YAML header. The file name matches the ID, for example `X-OPS-001.md`, and the directory matches the scope: `requirements/x-platform/`. The header holds the type, status, priority, owner, source, links to ADRs and the version. The owner is a role, for example `role-platform-engineers`, not a person's name.

The body has three required parts: the requirement itself, the rationale and the acceptance criteria. A requirement without acceptance criteria fails the check. That way "the system must be reliable" does not get into the register: it has to be turned into something that can be verified.

Why files and not a spreadsheet. A file in git has history, review through PRs, and an author for every change. A requirement can be discussed where the code is discussed, and changed together with the code in one PR.

## Rules in config, not in scripts

ID prefixes, requirement types, scopes, statuses and priorities are set in one file, `reqs.config.yaml`. The scripts do not know a single ID in advance. To adapt the template to your project you change the config, not the code.

`validate.py` does the checking: the header schema (`schema/requirement.schema.json`), file name matching the ID, prefix, type, directory, duplicates, presence of acceptance criteria and the tracing marker, and existence of the ADRs a requirement refers to. `generate.py` builds the register: the files become `REGISTER.md` and `register.json`. The register is never edited by hand, and CI checks that it is up to date.

## Strictness is switched on step by step

A new check cannot be turned on as an error right away: it would break the main branch on old files. So every rule has its own mode. First the rule runs as `warn`: CI prints the violations and attaches a report, but does not fail. Then a separate PR fixes the files from the report. When the report is empty, the rule is switched to `error` in a small PR.

CI is written for GitHub Actions and for Forgejo Actions. In the simple case it all comes down to one command: `make check`.

## A human sets the status

The requirement lifecycle is `draft`, `proposed`, `approved`, `implemented`, `verified`, and from any status a requirement can move to `deprecated`. The file is not deleted.

The main rule: the `approved` and `verified` statuses are set by a human through an approved PR. An agent can create a draft, suggest wording, link a requirement to an ADR, tag code. But it cannot decide that a requirement is approved or done. This is written into the process and into the agent instructions.

## Tracing to code and tests

Tracing uses [OpenFastTrace](https://github.com/mess-systems/tech-radar/blob/main/technologies/openfasttrace.md). A requirement carries the marker `req~X-OPS-001~1` and the line `Needs: impl, itest`, meaning it expects an implementation and an integration test. The implementation code gets the tag `[impl->req~X-OPS-001~1]`, the test gets `[itest->req~X-OPS-001~1]`.

`make trace` builds a report: which requirements are covered by code and tests and which are not. The number at the end of the marker is the version. If the wording of an approved requirement changes significantly, the version goes up, and tracing shows that the old tags in the code need review. By default the report does not fail the build, because the examples cover only some requirements. With `TRACE_STRICT=1` any tracing defect becomes an error.

The template includes an example implementation repository with such tags, so you can see how it looks in practice.

## Decisions next to requirements

A requirement answers "what is needed", an ADR answers "why we decided this way". The template has an ADR template and two examples: on secrets and on backups. The link goes both ways: the requirement has a `links.adr` field, the ADR has a list of related requirements. The validator checks that the ADRs a requirement refers to exist.

## A site for people who do not read git

`make site` builds a [MkDocs](https://github.com/mess-systems/tech-radar/blob/main/technologies/mkdocs.md) site from the requirements, the register, the ADRs and the process docs. The build runs in strict mode, so a broken link breaks it. The site is for people who read requirements but do not work with git.

## Rules for agents

Agents work by the same rules as people. The template has `AGENTS.md` for any agent, Cursor rules in `.cursor/rules/`, and eight ready prompts: create a requirement, link it to an ADR, tag code, review changes, check coverage, bootstrap the template in a new project, fix violations from a report, write acceptance criteria.

The limits for agents are short: do not raise statuses, do not bypass red CI, check that an ID number is free before a PR, do not delete requirements but move them to `deprecated`, and never add secrets, personal data or internal addresses.

## How it connects to the rest

Names of products, domains, environments and owners in requirements come from the [naming standard](https://github.com/mess-systems/naming-standard). The template's tools (OpenFastTrace, MkDocs, GitHub Actions, Forgejo Actions) are described in the [tech radar](https://github.com/mess-systems/tech-radar). OpenFastTrace and MkDocs are in the trial ring there: we are trying them but do not yet treat them as the default choice. Requirements for the reference architectures in [blueprints](https://github.com/mess-systems/blueprints) can be kept with the same template.

## Where to start

The repository is marked as a template on GitHub: you can create your own from it and follow the short quick start in the README. Change the prefix and scopes in the config, run `make check`, replace the examples with your own requirements, keep the CI you need and turn on branch protection.

What I do not know yet: how this approach behaves with hundreds of requirements and several teams, each with its own rhythm. The template is licensed under CC BY 4.0 for texts and Apache-2.0 for code. If you try it, [tell us](/about/) where it got in the way.
