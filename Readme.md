## Agenda

### Introduction - why we need clean, atomic and informative commit messages?
- ![intro](/images/git_msg_over_time.jpg)
- no longer misleading git blame -- screenshot from our project shown
    - git lg | grep "CR"
    - git lg | grep "review"
    - :(
- easier to review MRs
    - commit by commit

### Commit messages rules

- commit strucure
    - title, body, [links]
    - example:

```
    Summarize changes in around 50 characters or less

    More detailed explanatory text, if necessary. Wrap it to about 72
    characters or so. In some contexts, the first line is treated as the
    subject of the commit and the rest of the text as the body. The
    blank line separating the summary from the body is critical (unless
    you omit the body entirely); various tools like `log`, `shortlog`
    and `rebase` can get confused if you run the two together.

    Explain the problem that this commit is solving. Focus on why you
    are making this change as opposed to how (the code explains that).
    Are there side effects or other unintuitive consequences of this
    change? Here's the place to explain them.

    Further paragraphs come after blank lines.

     - Bullet points are okay, too

     - Typically a hyphen or asterisk is used for the bullet, preceded
       by a single space, with blank lines in between, but conventions
       vary here

    If you use an issue tracker, put references to them at the bottom,
    like this:

    Resolves: #123
    See also: #456, #789
```

- title standards
    - conventional commits https://www.conventionalcommits.org/en/v1.0.0/
        - ![example](https://miro.medium.com/max/1400/1*izVKF4AT1iDtv4fJO8oWWA.png)
    - (slightly different) semver commit messages https://gist.github.com/joshbuchea/6f47e86d2510bce28f8e7f42ae84c716
    - we don't use those in our projects, but you're invited to use:
        - "Add X" "Refactor X" "Fix X" ect.
        - summary written in imperative mood

### Context matters
- project level history
- merge request history
- your local trash / testing prod CI ;)

### Commitee rules for reviewers
- respect your reviewers -- their time and potential number of "WTF"s
- give context: description in the MR, issue link, sum-up changes
- nice commits structure
    - on the beggining it don't have to match final state -- eg. when you want to show missing test -- firstly do a test and make it fail, then do a fix in seaprate commit -- show both pipelines
- keep it small!
    - smaller psychological barrier to take a look -> faster feedback loop -> shorter branch lifetime -> less number of rebase
    - deeper care https://twitter.com/iamdevloper/status/397664295875805184
    - if too big -- your change can be devided to commits
    - or to separate MRs
    - in the case it can't be separated
        - rething -- maybe it can? Use feature flag
        - or see stacked MRs (not for this prez)

### My flow for Merge Requests commits

1. hack hack -- try separate commits from the begining -- it is easier later
    - try to extract refactors & fixes around main task goal
    - fixes / rebase done along the way
        - it is always easier to shuffle commits right now when you remember context, than next day
        - it doesn't mean it has to be done *right now* -- it may distract you from the coding flow
2. self-review
    - take position of the reviewer who knows "nothing" about your context
        - read from gitlab diff view
    - reorder, mix, mergem or reword commits with `rebase --interactive`
    - force-push'es are accepted at that point
3. fixes after 1. synchronous review
    - do `git ci --fixup <sha>` type of commits -- one per different issue addressed
        - if you have a problem with to find which commit the fixup belongs too -- it is either bad commits split from you or a new change that deserves normal commit
        - "!fixup" commits are signal for gitlab to mark MR as "Draft" -- this prevents merging
    - don't use force-push'es
4. rebase after first synchronous reviewer Approval
    - with squashing fixups
5. fixes after 2. (final) review
    - similar as after previous review
    - squash fixups AND other things that were splited for reviewer but are not important from the future readers / main branch point of view
    - don't include logic changes or they have to be re-reviewd by both reviewers
    - `git fetch origin master && git rebase origin/master`

### Misc Tip&Tricks

- git fixup/autosquash flow (when you want multiple commits to be merged on MR)
    - `git commit ...`
    - `git commit --fixup <sha>` (it force you to care more about commits content + it is easier to do it during the fix instead of one big rebase at the end)
    - `git rebase -i --autosquash`

- dealing with poetry.lock conflicts
    1. checkout existing version
        - `git checkout --ours poetry.lock`
        - or `git checkout origin/master -- poetry.lock`
        - ours vs theirs -- see excelent https://nitaym.github.io/ourstheirs/
    2. re-do your poetry changes eg. `poetry lock --no-update` and `git add poetry.lock` to mark as ready
        - this will allow resolver to use newer "cached" state and don't change lib versions from lock when it is not necessary

- order commits in a smart way to avoid conflicts
    - the rule of thumb -- keep those that are more risky be change-requested during review as more recent
        - usually it means the main feature/fix goes the last, because it is more often to see a MR comments around main feature rather changes requested for refactor, but it may depends on specific case
    - you don't have to worry about conflicts if your commits touch different code though

- "patch" commits as a nice tool to create multiple commits from your dirty working directory changes
    1. "hack hack"; oh, it seems I have much more stuff in working directory for the same file than fits in one atomic commit!
    2. `git commit -p -m "Refactor X"` to extract stuff for fist commit -- it shows you interactively what change to select
        - "--patch" works the same way for `add`, `stash` and some other git commands
    3. then remember to check tests! `git stash; make tests`
        - "every commit makes a green pipe" rule
        - this saves you from future fixups actually...
