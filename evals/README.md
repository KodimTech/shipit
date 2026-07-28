# Evals

Cases for `claude plugin eval`. Each directory is one case: a `prompt.md` and one or
more rubrics in `graders/`.

```bash
claude plugin eval shipit                      # all cases, with a no-plugin baseline arm
claude plugin eval shipit --case init-*        # filter
claude plugin eval shipit --report report.html # scored HTML report
```

The baseline arm is what makes a score meaningful: it runs the same prompt with the
plugin disabled, so the number reported is a delta, not an absolute.

> **Format caveat.** The CLI accepts `evals/**/case.yaml` **or**
> `evals/**/prompt.md` + `graders/*.md`. These cases use the second form because its
> shape is self-evident from the files. The exact `case.yaml` keys were not verified
> against a running eval, so nothing here depends on them. If a run rejects this
> layout, that is the first thing to correct.

## What each case guards

| Case | Guards against |
| --- | --- |
| `init-ruby` | Writing a command that does not exist in the repo |
| `init-js` | Leaking one ecosystem's conventions into another |
| `plan-collision` | Two parallel worktrees silently planning the same files |
| `doctor-missing` | Reporting an optional tool as a failure, or installing without consent |

Every case targets a rule that, when broken, produces confidently wrong output
rather than an error. Those are the failures worth paying for a grader to catch.

## Adding a case

Pick a hard rule from a SKILL.md whose violation would be silent. Write the prompt
that tempts the model to break it, and a grader that only passes when it did not.

A case that merely checks the skill produced output is not worth its run cost.
