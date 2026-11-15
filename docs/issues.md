# Issues: the suite standard

Issues are **roadmap items**: one issue per piece of work that ships on its own. Bugs use the same
shape. Taken from what already works in Heimdall and Huginn, made the same for every app.

## Where things go

| GitHub | Meaning | Rule |
|---|---|---|
| **Milestone** | A phase or step of the app's roadmap | Named as the app's own docs name it: `Phase 3: Breadth` (Heimdall), `Step 3: LAN` (Bifröst). Each issue has exactly one |
| **Label: type** | What kind of work | Exactly one of `feature`, `bug`, `security`, `testing`, `docs`, `ops`, `performance`, `accessibility` |
| **Label: `epic`** | Too big for one issue | Split into sub-issues when work starts |
| **Label: `blocked`** | Waits for another issue, often in another repo | The body names the issue it waits for |
| Phase labels (`phase-4`) | Not used | The milestone already says it. Huginn's stay until it next tidies up |

GitHub's default labels (`enhancement`, `question`, …) are not used for roadmap items.

## The title

What exists when the issue is closed, as a short noun phrase. Capital first letter, no full stop.

- Good: `Pairing by a compared 6-digit code`, `Architecture tests for the layer rules`
- Not: `Implement pairing`, `[Bifrost] pairing stuff`

## The body

```markdown
<One paragraph: what this is and why it matters. Who notices when it is missing.>

### Scope
- [ ] <a checkable piece>
- [ ] <another>

### Not in scope
- <what someone might expect here, and where it is instead>   (only when it helps)

### Depends on
- <repo>#<n>, or "nothing"   (only when there is a dependency)

### Spec
- [<doc> §<n>](<link to the section on the default branch>)

Done when: <one sentence a reviewer can check: the behaviour, the tests, the docs>.
```

- Link the spec. Do not copy it: the issue says what and why, the spec says how.
- A decision made while working goes in the spec or `decisions.md`, then the issue links it.
- Close with the commit or PR that finished it. Done work is not left open.

## Writing

Plain, short sentences, as in the suite docs. No emoji in titles.
