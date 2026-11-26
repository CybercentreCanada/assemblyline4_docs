# Contribution Guidelines

This defines some guidelines for managing repositories that are part of the Assemblyline project. Some information here may only be relevant to Assemblyline team members, but much of it will be broadly useful for contributing.

## Git

Our git repositories are generally managed with concurrent and persistent stable and development branches. The stable branch should be the `main` branch (`master` in legacy repos) and the development branch should be named `dev`.

These branches contain the code that will be used whenever a new release is created. A release should be created soon after these branches are modified to prevent subsequent builds from including unexpected changes. The actual deployable releases from either branch are represented by tags created by our build environment. This implies that when no release is being prepared the head of these branches should be a tagged commit representing the code running in a corresponding up to date system.

A feature or patch development generally follows this lifecycle:
- A feature branch is forked off the **stable branch**.
- Development and local testing is done within the feature branch.
- Pull requests into the stable and development branches are created concurrently when initial development is complete or to begin tracking of the branch against issues.
- The feature branch should be merged into the development branch whenever needed for integration testing and testing in our staging environment.
- A stable pull request can be considered ready when reviews and integration testing in the staging environment are complete.
- A stable pull request that is ready can be merged when a release that includes it is being produced.

Triggering and deploying a stable builds should be considered whenever a stable pull request is ready to merge. This is to favour many smaller changes in our production environment rather than larger more complex deployment changes with harder to review change surfaces.

Note that branches are forked off of stable, then merged back into both development and stable branches. Stable can always be merged into a feature branch or the development branch and a feature branch can always be rebased on stable, but to maintain this **the development branch may never be merged into a feature branch or the stable branch**. If conflicts cause github to recommend doing this, _do not_. Instead either create a temporary branch merging your feature branch with development to resolve conflicts, or merge the branch into development manually resolving the merge conflicts within the development branch.

### Maintaining the development branch

If every feature developed to the point of merging into the development branch eventually merges into stable the two should always come to be the same over time. This isn't always the case in practice, for example when a branch is abandoned and not reverted, or when a reversion is incomplete. To manage this the development branch can be occasionally recreated from the stable branch and merging in relevant feature branches.

For some repos with a small change frequency a development branch may be omitted and only created when there is the intention to maintain separate stable and development builds.

### Other Requests
- When a feature is developed across multiple repos, use the same branch name whenever possible.
- Keep your branch up to date with the stable head.

### Proposed changes to consider
- Squash merge PRs into stable.
- Use the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) format.
