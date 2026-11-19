# The trust layer

[Español](capa-de-confianza.es.md)

Torbyn's own summary of its direction for teams of coding agents. It is a
design: almost everything it describes **is not built**. The end says what
exists today.

## The problem

Several coding agents working at the same time on one project do so without
guarantees:

1. **They overwrite each other.** Two agents edit the same file and the last
   one to write wins. Nobody notices until something stops working.
2. **Nobody knows who did what.** In a shared folder, git sees changes, not
   authors.
3. **Undoing is manual work.** If an agent breaks something, going back
   without losing the others' work requires knowing git, and whatever wasn't
   saved in a commit is lost.
4. **Putting the work together is done blind.** With one branch per agent,
   joining three branches is still the same old manual task.
5. **“Done” doesn't mean “works”.** The agent says it finished; nobody ran
   the tests.
6. **When a tool runs out, the work is left half done.** Continuing in
   another one means explaining everything again.

## The idea

Torbyn doesn't compete with the agents or direct them. It puts **the
guarantees around them**:

- who can touch what;
- how to go back;
- what gets put together;
- what is verified;
- how to continue when a tool goes down.

The person stays in charge: they turn on, set up or run each guarantee.

## The rules that don't change

- **Torbyn doesn't read what happens in the terminals.** Its signals come
  from git, from whether a process is still open and from its exit code,
  never from a terminal's text.
- **The person decides.** Nothing runs on Torbyn's or an agent's initiative:
  undoing, putting work together, verifying or handing the work to another
  tool are their clicks.
- **Nothing leaves the computer.** Everything is local git and local files.

## The five pillars

### 1. Who can touch what

An open format, in the project's `.torbyn/` folder, for the person to assign
paths to each agent and for every tool to read.

- With the tools that allow checking an edit before it happens, Torbyn stops
  it if the path belongs to another.
- With the rest, the rule reaches them as an instruction, and Torbyn warns
  afterwards if someone stepped out. For that, each agent has to work in its
  own folder with git.

### 2. Restore points and undo

Before each agent opens, a restore point of its folder. It includes what
isn't in a commit yet and the new files, and it touches nothing of the
person's: neither their branches nor their index. What is ignored, such as
`.env` or `node_modules`, doesn't go in.

- **Undoing always leaves another point first**, so undoing can be undone
  too.
- With an agent in its own folder, it is undone whole and exactly.
- In a shared folder, it is undone file by file, and the person chooses
  which ones.

### 3. Review and merge

One view per project with what each agent changed. It warns of clashes
**before** they happen, while the agents are still working (“these two
agents would clash in this file”), without touching any folder.

Putting an agent's work together is only offered if:

- it is in a commit;
- it is verified;
- it doesn't touch another's paths;
- it doesn't clash.

There is always a restore point first, and Torbyn **doesn't resolve clashes
on its own**: it says which files clash and with whom.

### 4. Verification

The person sets the commands that mean “done” for their project: their
tests, the type checker, the linter. Torbyn runs them in the agent's folder.

- **The verdict is the exit code**, never a reading of the text.
- **It is tied to the exact state of the files.** If something changed
  afterwards, it goes back to “not verified”.
- **Only the person sets the commands**, never the repository.
- **They run with a minimal environment** and a time limit, with no more
  permissions than the agent's own terminal.

### 5. Continuity

“Continue in…” hands an agent's task to another tool, in the same folder and
on the same branch. The person presses it; it is never automatic.

The new agent gets **only references**: which branch, from which commit and
what to look at in git. Never text written by another AI. If the previous
agent left a note, it is told where it is and reads it itself.

## What exists today

**Built:**

- **Assigned paths.** The person assigns parts of the project to an agent or
  to themselves, in the launcher or while they work.
  - With Claude Code, Torbyn stops an agent from editing or creating files in
    another's path, also if the path was assigned after it opened. That needs
    Git for Windows.
  - With the other tools it is an instruction, and Torbyn warns if they
    stepped out when each agent works in its own folder with git.
- **The work contract.** Each project has three lists: the paths reserved for
  an agent, the read-only ones (those the person assigns to themselves) and,
  if wanted, the paths an agent is allowed to write in.
  - Claude Code keeps to it with the same block as the assigned paths.
  - The other tools get it as an instruction.
  - Torbyn checks it with git against the commit the agent's folder had when
    it was launched, and each cell says “Contract: met”, “Contract: 1 breach”
    (or more) or “Contract: not checked” with the reason. In a shared folder
    it always says “Contract: not checked”: there, nobody knows who changed
    what.
- **A folder per agent.** Each agent chooses where it works; with a
  repository with commits, each one in its own worktree and on its own
  branch.
- **Changes per agent.** What each one changed, file by file, read with git
  without running anything from the repository.

**Not covered:** what an agent changes with terminal commands (Bash).

**Designed, not built:** restore points, review and merge, verification and
continuity.
