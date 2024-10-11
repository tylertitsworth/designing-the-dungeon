# Industrial Software Design in GitHub Actions

GitHub Actions is a powerful CI/CD tool built into GitHub. You can automate trivial tasks for day-to-day maintenance of your repository. However, the open-ended design of GitHub Actions leaves a lot to be desired. There is a definitive paradigm that one can follow to achieve a powerful automated pipeline in GitHub actions without the need to design around events and triggers.

## Architecture

This architecture is composed of 3 separate workflows, represented by the relevant subgraphs `Event`, `Workflow Call` and `Composite`.

[![](https://mermaid.ink/img/pako:eNpdUcFuwyAM_RXEufmBHHbpdts0aT1MqnLxwFlQAkaOWVtV_fcBUbNm74TfezwbfNWGLOpW9xOdzAAs6vWjCypjTl_fDHFQLz8YZOEKrGM04iiszgIPwu7czCgpqqZ5UmZAM1J6uLgGfhKPpZ_awzT96QX3WzUiT-N6MNJAkqEy7xEZau-NnuJEYLdJa7c9-UizE9zqBWvcVsLwkLUp_nWsUxyS98CXxZTdeqc9sgdn87deC91pGdBjp9t8tMBjp7twy778LjpcgtGtcMKdTtGC4LODPLm_kxHCkSiXPUxzrtE6IX5b1la3d_sF3l-L1A?type=png)](https://mermaid.live/edit#pako:eNpdUcFuwyAM_RXEufmBHHbpdts0aT1MqnLxwFlQAkaOWVtV_fcBUbNm74TfezwbfNWGLOpW9xOdzAAs6vWjCypjTl_fDHFQLz8YZOEKrGM04iiszgIPwu7czCgpqqZ5UmZAM1J6uLgGfhKPpZ_awzT96QX3WzUiT-N6MNJAkqEy7xEZau-NnuJEYLdJa7c9-UizE9zqBWvcVsLwkLUp_nWsUxyS98CXxZTdeqc9sgdn87deC91pGdBjp9t8tMBjp7twy778LjpcgtGtcMKdTtGC4LODPLm_kxHCkSiXPUxzrtE6IX5b1la3d_sF3l-L1A)


The `Event` workflow is triggered by a `push`, `pull_request`, `schedule`, or any kind of relevant event that happens during the contribution process.

The `Workflow Call` is a middleman that runs all necessary setup steps at the repository scope. These are tasks that usually involve setting up an environment that requires secrets.

The `Composite` workflow is the actual operation that needs to be done, and it's separated so that it can be called from any other repository without the need to set up the environment with the exact same secret configuration.

## Day 1 Actions vs. Day 100 Actions

You may look at this architecture and feel that it's overkill, and for most small automations you are absolutely right. You may not need this architecture today, but for large projects that require dynamic matricies and multi-repo workflows, this architecture will save you a lot of time in the long run.

The fundamentals of GitHub actions are simple, you begin with an event of kind, usually people start off with `workflow_dispatch`, because it gives you a little UI to trigger the jobs manually.

The workflow is typically the same 3 things: `checkout`, `install`, and `build/run`. However, therein lies the problem. Checkout is inherently tied to the repository's metadata and secrets because your token by default is scoped to that repository, likewise install steps usually involve knowledge about the runner label or the action environment. Your operation, or `build/run` in this case is not tied to any specific repository configuration, but instead a known environment configuration. Multiple repositories could call the same operation assuming they have the same environment configuration.

Finally, at some point when you start using events like `pull_request` and `schedule`, you will need metadata from that event in order to correctly trigger the desired event. Instead of pooling it all into one workflow, you will need separate workflows. Those separate workflows all need the same steps to reproduce the known environment configuration to run the operation.

## Simplicity is the Key to Success

