# Git Hooks

Git hooks can be used in your actual projects to save time.

**Installation:** Simply place the hook files into `.git/hooks` within your project directory and make them executable (e.g. `chmod +x .git/hooks/commit-msg`).

## commit-msg

This hook automates the use of “[smart commits](https://confluence.atlassian.com/display/AOD/Processing+JIRA+issues+with+commit+messages)” by extracting the issue key from the branch name and prepending it to every commit message (e.g. `[PROJECT-123] Solve important problem.`).

Instead of having to manually copy & paste the issue key into the commit message, simply use *feature branches* and follow the [Git Flow](http://nvie.com/posts/a-successful-git-branching-model/) naming convention (e.g. `feature/PROJECT-123-important-problem`). Feature branches created in JIRA will automatically do this.

Example:

    $ git branch
    * feature/PROJECT-123-important-problem
      master
    
    $ git commit -m "Solve important problem."
    [feature/PROJECT-123-important-problem 42f00ba7] [PROJECT-123] Solve important problem.
