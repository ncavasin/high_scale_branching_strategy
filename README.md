# Branching strategy with semantic versioning

This branching strategy is a way to manage the development, testing, and deployment of software applications in a 
coordinated and organized manner. It is particularly well-suited for medium to large-sized applications that have 
multiple concurrent developers working on them.  It can be implemented on any continuous integration and continuous
deployment (CICD) platform and cloud provider. The strategy is also intended to work with any issue tracking system.

Our strategy proposes the following three stable environments to achieve a fully-automated CICD flow:

- **develop**: feature development environment.
- **sit**: integration and testing (manual or automated) environment.
- **master**: productive environment.

This allows for a clear separation of concerns, with the ``develop`` environment serving as a sandbox for developers to 
build and test new features, the ``sit`` environment serving as a staging ground for integrating and testing those features 
before they go live, and the ``master`` environment representing the version of the software that is currently in production
and being used by end users.

In addition to these stable environments, the strategy also proposes the use of several temporal branches which will
eventually be merged to their proper **parent environment** and their complete name *should* be given by the ticket 
number in the issue tracking system describing what needs to be achieved:

- **feature/\*\***: to develop new features branched off sit environment, which is less prone to bugs.
- **bugfix/\*\***: to fix bugs found on sit environment.
- **hotfix/\*\***: to fix bugs found on productive environment.

These temporal branches allow developers to focus on specific tasks and then merge their work back into the appropriate 
stable environment when they are finished.

Finally, the strategy suggests the use of an environment-less amd stable branch:

- **release**: for bundling features and fixes together before they are merged into ``master`` branch. Generates a 
new tag for each finished Sprint that includes all the new content from that Sprint.

  <img src="https://bitbucket.org/cavasinnicolas/gitflow/raw/554702b8f88f43da1b320b97041c45973a41b4ae/imgs/branching_strategy.png" alt="Branching strategy" width="10" height="10;"/>


Overall, this branching strategy provides a structured and organized approach for managing the development, testing, and
deployment of software applications. It allows for a clear separation of concerns and encourages a focus on specific 
tasks, while also providing mechanisms for testing and integrating changes before they go live. By using this strategy, 
organizations can improve the quality and reliability of their software, and better meet the needs of their users.


## Branch usages and restrictions

Based on environment's branch interacting with, developers will have less permissions as they get closer to production.


- Sprint's ``feature/**`` branches will be promoted to a testing environment in an isolated fashion as soon as their implementation is finished so 
  tests can be executed properly.
 
  - Once testing has finished successfully, a QA team member will merge the branch to ``release`` using a Pull-Request.  
  
  - When merged to ``release``, those branches will reach end-of-life and will be deleted. 
  
  - Finally, they will be held at ``release`` creating a bundle and waiting to be promoted to ``master`` branch via Pull-Request 
  once Sprint's finished.
  
- ``release`` branch will be promoted to ``master`` via Pull-Request when Sprint's over, creating a new semver tag.

---

### Develop
As unrestricted as possible. Permissions for everybody. No need for reviewers. CommitHashId+Timestamp tagging.

#### Unique scenario: integrate new features to development environment.

Create a Pull-Request from temporal branch ``feature/**`` against ``develop`` branch to trigger its test pipeline.

On pipe failure fix errors/bugs and restart process. Merge won't be available.

On pipe success approve and merge Pull-Request without approval to speed-up integration process.

The merge will trigger ``develop``'s deploy pipeline:

  - On pipe failure, a rollback to the previous version (if any) will be automatically performed and the pipeline logs 
  must be checked to discover what went wrong and solve it.
  - On pipe success, new development version will be deployed to the environment.
    - ``develop`` branch tagging will be automated.

---

### SIT

A bit more restricted. Permissions for everybody. Only reviewers can approve a Pull-Request. Semver tagging.


#### Scenario 1: promote a feature to testing environment.

Create a Pull-Request from temporal branch ``feature/**`` against ``sit`` branch to trigger its pipeline.

On pipe failure, fix error/bugs and restart process. Merge won't be available.

On pipe success, assign a reviewer and wait for the PR to be approved and merged.

.                                                     

#### Scenario 2: integrate a bugfix to testing environment.

Create a Pull-Request from temporal branch ``bugfix/**``against ``sit`` branch to trigger its pipeline.

On pipe failure, fix error/bugs and restart process. Merge won't be available.

On pipe success, assign a reviewer and wait for the PR to be approved and merged.

  - Downstream push of ``bugfix/**`` to ``develop`` branch will be automated and no pipe will be triggered.

.

#### Despite the scenario

Any merge will trigger ``sit``'s deploy pipeline:

  - On pipe failure, a rollback to the previous version (if any) will be automatically performed and pipe's logs 
  must be checked to discover what went wrong and solve it.
  - On pipe success, new testing version will be deployed to the environment.
    - ``sit`` branch tagging will be automated.


### Release

**Not an environment**. It's a **bundling branch** used to create a release candidate per sprint. 

As restrictive as possible. Permissions only for QA role. Semver tagging.


#### Unique Stage: promote features and bug/hot fixes. 

As soon as a feature's testing has been finished, QA member creates a Pull-Request from temporal branch ``feature/**`` 
or ``bugfix/**`` or ``hotfix/**`` against ``release`` branch to trigger it's unique testing pipeline.

On pipe failure, fix error/bugs and restart process. Merge won't be available.

On pipe success, assign a reviewer and wait for the PR to be approved and merged.

Any merge will trigger ``release``'s tagging pipeline:

  - A new release candidate tag following semver convention with the `-rc` suffix will be automatically generated.

---

### Master

As restrictive as possible. Permissions only for Team Leader or DevOps role. Semver tagging. 


#### Scenario 1: promote a release candidate to productive environment.

Create a Pull-Request from release candidate against ``master`` branch to trigger its pipeline.

On pipe failure, fix error/bugs and restart process. Merge won't be available.

On pipe success, assign a reviewer and wait for the PR to be approved and merged.

.

#### Scenario 2: integrate bugfix to productive environment.

Create a Pull-Request from temporal branch ``hotfix/**`` against ``master`` branch to trigger its pipeline.

On pipe failure, fix error/bugs and restart process. Merge won't be available.

On pipe success, assign a reviewer and wait for the PR to be approved and merged.

  - Downstream push of ``bugfix/**`` to ``release`` branch will be automated and no pipe will be triggered.
  - Downstream push of ``bugfix/**`` to ``sit`` branch will be automated and no pipe will be triggered.
  - Downstream push of ``bugfix/**`` to ``develop`` branch will be automated and no pipe will be triggered.

#### Despite the scenario

Any merge will trigger ``master``'s deploy pipeline:

  - On pipe failure, a rollback to the previous version (if any) will be automatically performed and pipe's logs 
  must be checked to discover what went wrong and solve it.
  - On pipe success, new testing version will be deployed to the environment.
    - ``master`` branch semver tagging will be automated.

  
<img src="https://bitbucket.org/cavasinnicolas/gitflow/raw/554702b8f88f43da1b320b97041c45973a41b4ae/imgs/branching_strategy-demo.png" alt="Branching strategy demo" width="10" height="10;"/>


## Branch protection rules
 
- All four stable branches -``develop``, ``sit``, ``release`` & ``master``- will be protected against write access. 
- The only way to modify these branches will be via Pull-Requests as explained above.
- Pull-Requests against ``develop`` branch will be mergeable only when: 

    - Pipeline's execution has been successful.
  
- Pull-Requests against ``sit`` and ``master`` branches will be mergeable only when:
    
    - Pipeline's execution been successful.
    - Its proper reviewer has approved it.

## About merge conflicts

If there's a merge conflict, follow this steps:

1. Create a *locally* temporal branch, branched off the highest possible stable environment.
2. *Locally*, merge to that new branch the content of the branch that generated the conflict.
3. Push the branch with the solved conflict to the remote git repository.
4. Open a new Pull-Request against same environment using the conflict-solver branch.




## References

[0] - [GitOps and GitFlow and GitHub Flow](https://elisabethirgens.github.io/notes/2021/06/gitops/)

[1] - [GitOps](https://www.atlassian.com/git/tutorials/gitops)

[2] - [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)

[3] - [GitHub flow](https://docs.github.com/en/get-started/quickstart/github-flow)

[4] - [Trunk-based development](https://trunkbaseddevelopment.com/)

[5] - [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)

[6] - [QA Test - What is DEV, SIT, UAT & PROD?](https://medium.com/@buttertechn/qa-testing-what-is-dev-sit-uat-prod-ac97965ce4f)
