# Branching strategy with semantic versioning
We propose the following three environments to achieve a fully-automated CICD flow:
- **develop**: feature integration environment.
- **release**: testing environment.
- **master**: productive environment.

On top of them, we add these temporal branches which will eventually be merged to their proper environment:
- **feature/\*\***: to develop new features branched from release environment (less bug-prone).
- **bugfix/\*\***: to fix bugs found on release environment.
- **hotfix/\*\***: to fix bugs found on productive environment.

### Behavior
##### Development environment

**Stage 1:** integrate new feature to development environment.
Create a Pull-Request from temporal branch ``feature/**`` against ``develop`` branch to trigger its pipeline.
- On pipe success approve and merge Pull-Request withou any code-review nor approval (speed-up integration process).
- On pipe failure fix errors/bugs and restart process. Merge won't be available.

##### Release environment
**Stage 1**: promote feature to testing environment.    
Create a Pull-Request from temporal branch ``feature/**`` against ``release`` branch to trigger its pipeline.
- On pipe success, assign a reviewer and wait for the PR to be apprvoed and merged.
    - ``release`` branch semver tagging will be automated.
- On pipe failure, fix error/bugs and restart process. Merge won't be available.

**Stage 2**: integrate bugfix to testing environment.
Create a Pull-Request from temporal branch ``bugfix/**``against ``release`` branch to trigger its pipeline.
- On pipe success, assign a reviewer and wait for the PR to be approved and merged.
    - ``master`` branch semver tagging will be automated.
    - Downstream push of ``bugfix/**`` to ``develop`` branch will be automated. No pipe will be executed.
    - ``bugfix/**`` branch deletion will be automated.
- On pipe failure, fix error/bugs and restart process. Merge won't be available.

##### Productive environment
**Stage 1**: promote feature to production environment.
Create a Pull-Request from temporal branch ``feature/**``against ``master`` branch to trigger its pipeline.
- On pipe success, assign a reviewer and wait for the PR to be approved and merged.
    - ``master`` branch semver tagging will be automated.
    - ``feature/**`` branch deletion will be automated.
- On pipe failure, fix error/bugs and restart process. Merge won't be available.

**Stage 2**: integrate bugfix to production environment.
Create a Pull-Request from temporal branch ``hotfix/**`` against ``master`` branch to trigger its pipeline.
- On pipe success, assign a reviewer and wait for the PR to be approved and merged.
    - ``master`` branch semver tagging will be automated.
    - Downstream push of ``hotfix/**`` to ``release`` branch will be automated. No pipe will be executed.
    - Downstream push of ``hotfix/**`` to ``develop`` branch will be automated. No pipe will be executed.
    - ``hotfix/**`` branch deletion will be automated.
- On pipe failure, fix error/bugs and restart process. Merge won't be available.

> Every executed pipeline will add a comment in the PR per each executed step as a status report.

### Branch protection rules
- All three stable branches -``develop``, ``release`` & ``master``- will be protected against write access. 
- The only way to modify these branches will be via Pull-Requests as explained above.
- Pull-Requests against ``develop`` branch will be mergeable only when:
    - Pipeline's execution has been successful.
    - Any user has approved it.
- Pull-Requests against ``release`` and ``master`` branches will be mergeable only when:
    - Pipeline execution's been successful.
    - Any reviewer has approved it.

### About merge conflicts
If there's a merge conflict, follow this steps:
1. *Locally*, open another temporal branch branched off the highest possible stable environment.
2. *Locally*, merge to that new branch the content of the branch that created the conflict.
3. Push the branch with the solved conflict to the remote git repository.
4. Open a new Pull-Request against same environment using the conflict-solver branch.
