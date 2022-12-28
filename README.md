# Branching strategy with semantic versioning
We propose the following three stable environments to achieve a fully-automated CICD flow:

- **develop**: feature integration environment.
- **release**: testing environment.
- **master**: productive environment.

On top of them, we add these temporal branches which will eventually be merged to their proper environment:

- **feature/\*\***: to develop new features branched from release environment (less bug-prone).
- **bugfix/\*\***: to fix bugs found on release environment.
- **hotfix/\*\***: to fix bugs found on productive environment.


![Branching strategy picture](./imgs/branching_stategy.png "Branching strategy")


# REVISAR INI
Finally, we suggest another temporal, auxiliary and environment-less branch:

- **staging**: feature bundling branch. Located before master branch. Once merged against, branches reach end-of-life and get deleted.

# REVISAR FIN

## Behavior

Based on the environment interacting with, developers will have less permissions as they get closer to production.


- Sprint's feature branches will reach end-of-life and will be deleted once merged to the staging branch. 
  
  - They will be promoted to a testing environment in an isolated fashion as soon as their implementation is finished so testing can be executed properly. 
  
  - They must be accumulated at ``release`` branch and bundled altogether to be promoted to a productive environment, that's when
    the get deleted.

---

### Development environment
As unrestricted as possible. No need for reviewers.

**Unique stage:** integrate new feature to development environment.

Create a Pull-Request from temporal branch ``feature/**`` against ``develop`` branch to trigger its pipeline.

- On pipe success approve and merge Pull-Request withou any code-review nor approval (speed-up integration process).

- On pipe failure fix errors/bugs and restart process. Merge won't be available.

---

### Release environment

A bit more restricted. Only reviewers can approve a Pull-Request.


**Stage 1**: promote a feature to testing environment.

Create a Pull-Request from temporal branch ``feature/**`` against ``release`` branch to trigger its pipeline.

- On pipe success, assign a reviewer and wait for the PR to be approved and merged.

    - ``release`` branch semver tagging will be automated.
    - ``feature/**`` temporal branch deletion will be automated.

- On pipe failure, fix error/bugs and restart process. Merge won't be available.

.

**Stage 2**: integrate a bugfix to testing environment.

Create a Pull-Request from temporal branch ``bugfix/**``against ``release`` branch to trigger its pipeline.

- On pipe success, assign a reviewer and wait for the PR to be approved and merged.

    - ``master`` branch semver tagging will be automated.
    - Downstream push of ``bugfix/**`` to ``develop`` branch will be automated. No pipe will be executed.
    - ``bugfix/**`` temporal branch deletion will be automated.

- On pipe failure, fix error/bugs and restart process. Merge won't be available.

--- 

### Productive environment

As restrictive as possible. Only a specific group of members (team leader or senior devs) can approve Pull-Requests.


**Stage 1**: promote testing environment to productive environment.

Create a Pull-Request from stable branch ``release``against ``master`` branch to trigger its pipeline.
- On pipe success, assign a reviewer and wait for the PR to be approved and merged.

    - ``master`` branch semver tagging will be automated.
    - ``feature/**`` branch deletion will be automated.

- On pipe failure, fix error/bugs and restart process. Merge won't be available.

.

**Stage 2**: integrate bugfix to production environment.

Create a Pull-Request from temporal branch ``hotfix/**`` against ``master`` branch to trigger its pipeline.

- On pipe success, assign a reviewer and wait for the PR to be approved and merged.

    - ``master`` branch semver tagging will be automated.
    - Downstream push of ``hotfix/**`` to ``release`` branch will be automated. No pipe will be executed.
    - Downstream push of ``hotfix/**`` to ``develop`` branch will be automated. No pipe will be executed.
    - ``hotfix/**`` branch deletion will be automated.

- On pipe failure, fix error/bugs and restart process. Merge won't be available.

***Every executed pipeline will add a comment in its Pull-Request per each executed step as a status report.***

![Branching strategy timeline](./imgs/branching_stategy-timeline.png "Branching strategy timeline")


## Branch protection rules
 
- All three stable branches -``develop``, ``release`` & ``master``- will be protected against write access. 
- The only way to modify these branches will be via Pull-Requests as explained above.
- Pull-Requests against ``develop`` branch will be mergeable only when: 

    - Pipeline's execution has been successful.
  
- Pull-Requests against ``release`` and ``master`` branches will be mergeable only when:
    
    - Pipeline's execution been successful.
    - Any reviewer has approved it.

## About merge conflicts

If there's a merge conflict, follow this steps:

1. Create a *locally* temporal branch, branched off the highest possible stable environment.
2. *Locally*, merge to that new branch the content of the branch that generated the conflict.
3. Push the branch with the solved conflict to the remote git repository.
4. Open a new Pull-Request against same environment using the conflict-solver branch.
