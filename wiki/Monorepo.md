# Monorepo

## Pros

| Pro                      | Description |
| ------------------------ | ----------- |
| Cross component changes  | Much easier to perform cross-component changes, such as API changes. All components change at the same time.
| Dependency management    | Dependencies are naturally synched. The same submodule is used by the different components. The dependency needs to be built just once (e.g. Arrow).
| Atomic commits           | Easier to track down the origin of regressions using git bisect.
| Benefit from LLM         | Monorepos will make AI tools much more efficient and context aware. Cross-module refactor becomes much easier. 
| Improved visibility      | Everyone has the full picture. Easier to understand what changed.
| Unified tooling          | Monorepo allows for consolidating tooling, development procedures and CI.
| Cross team collaboration | Easier to contribute to different modules, reduces psychological boundaries

## Risks

| Risk                      | Description |
| ------------------------- | ----------- |
| Longer development cycle  | Pushing changes requires more rigorous testing and PR cycle becomes much longer. Long merge queues. Heavier load on CI machines.
| Lost flexibility          | "Quick and dirty", semi-tested patches are no longer allowed. Must follow strict protocols. Slower turnaround time.
| Ownership                 | Monorepo infra maintenance must be handled full time. Clear ownership of build and CI infra per component is needed.
| Cross-team standards      | It might be hard to define and enforce common standards that are accepted by everyone.

## Tooling

- **_Merge Queue_**
  * Problem: In a monorepo, linear history is required (squash merge strategy), for allowing efficient bisecting and clear version history.
  With linear history, the issue of stall PRs is significantly amplified. Each PR must be based on the latest HEAD.
  When a PR passes gater tests and is considered "ready for merge", merging it will require all other
  "ready for merge" PRs to rebase and retest.  
  * Solution: Using an automated merge queue - **GitHub Merge Queue** feature - automates the process.  

- **_Differential Testing_**
  * Problem: Each PR should be verified by the tests related to the changed component(s). We can't run the whole set of tests for each monorepo PR.  
  * Solution: Implement test selection:
    - Create a change-to-component map, for example via YAML:
      ```
      component-a:
        paths:
          - component-a/**
      component-b:
        paths:
          - component-b/**
      ```
    - Build a component dependency graph.
    - Write a test selector for running only relevant tests. Consider the files changes in the PR.  
    - **Advanced:** Test Result Memoization - cache test results and skip running tests when possible.
  * Assumption: The gating tests of each component offer sufficient testing coverage.
  * Caveat: need to carefully map hidden and non-trivial dependencies.  
    While not all of them can be mapped, need to enable requesting extra tests from the gater.  

## Steps towards Monorepo

- **Define ownership**  
  * A mono-repo might require full time maintenance, or at least "on-call" duty to handle any issue that rises.  
  * The ownership for build, CI and test infra per component must be clearly defined.  

- **Component testing coverage**  
  * Make sure every components is covered by adaquete gater tests. Map the gaps and extend the coverage as needed.

- **Component testing consistency**  
  * Make sure component gaters produce consistent results. Ideally, build and run tests in docker containers.  

- **Improve build efficiency**
  * Make sure the gater build process for each component is efficient and uses proper caching.  
  * For C++, use ccache and ctcache.  
  * For Java, use caching of maven artifacts. Avoid unneeded downloads.  
  * Smart caching of common artifacts, such as Arrow.  

- **Gradual transition - fake monorepo**
  * Keep components in separate repos.
  * Create a workspace repo that pulls each repo into a fixed layout (via script, not submodule); builds the system as if it were a monorepo; tests system integration end-to-end.

- **Infra and testing machines**
  * Define the minimum and desired requirements for CI resources. There should be enough machines to support build and integration flows with reasonable service times.  
  * Consider the mix between Github actions and Jenkins jobs.  

## Implementation considerations

- **Git history**
  * Git history of the projects should be maintained.  
  * Attempt to discard large unused objects while rewriting history. Check how history could be kept compact yet complete.  

- **Commit format**
  * Consider changing the commit format string to include the relevant components for better visibility.  
    Example from Arrow project: `GH-45185: [C++][Parquet] Raise an error for invalid repetition levels (#45186)`.
