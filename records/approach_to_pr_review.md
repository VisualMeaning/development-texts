# Approach to Pull Request review

This document aims to set some basic standards around Pull Requests (PRs) and code review, both in terms of what is expected from the person making the PR and what is expected from the person reviewing the PR.

## VM's development model

The master copy of all Visual Meaning code lives in the default branch of each repository in the Visual Meaning GitHub organisation, which will typically be called `master` or `main`. Which out of `master`/`main` is the default branch will depend on the specific repository in question - if unsure, simply open the repository in GitHub and check which branch is selected as the default. This documentation will refer to "the default branch" in place of `master`/`main`.

When we develop new code, we do not do our work in the Visual Meaning repositories. Instead, we first make a [fork](https://docs.github.com/en/pull-requests/get-started/about-forks) of the repository. This is a completely independent copy of the repository that exists under your own personal GitHub account. When we start a new piece of work, we update the default branch in the fork repo with the most recent commits from the Visual Meaning ("upstream") repositories, and then we use that updated branch as the basis to create a working branch. This working branch is where we make any new commits.

This model isolates developer work from the canonical default branches in two ways.
- Developers work in copies of their own fork repositories, which are completely separate from Visual Meaning repositories.
- Developers work in working branches created using the default branch state, _not_ the default branch itself.

Once a working branch is ready to be proposed as a change to the default branch, it must be submitted for review via a [pull request](https://docs.github.com/en/pull-requests/get-started/about-pull-requests). Pull requests can be made across repositories, so you can submit your `username/repository-name/working-branch-name` branch to be merged into the `VisualMeaning/repository-name/master` branch. Once the pull request has been reviewed and approved, you can merge the changes from your working branch into the default branch, updating the default branch with your code changes.

## Making a PR

One very important thing to cover before getting to the process: **PR authors are responsible for the code they propose in their PRs**. We are experimenting with code that has been authored by AI agents, however agentic AI is a tool only. It cannot take responsibility for quality or outcomes. That is on _you_ as the PR author. If your PR incorporates AI-authored code, make sure you understand what it does and why. You will be asked questions about it.

### What you should do first

Most of our repositories will have some combination of linting checks and unittests that run when you submit a pull request. If any of these checks fail, your PR will be blocked from being merged even with an approving review until you fix them. To save yourself some pain, it is best practice to run the checks _first_ and fix them prior to making the PR. Test commands are usually packaged in the repository `Makefile` and invoked with some variant of `make check`/`make test`/`make lint` - check the `Makefile` in the root of the repository for the exact commands.

### Process and intent

To make a PR, commit your changes to a working branch in your local repository, push the working branch to your fork in GitHub, and propose the changes in that branch as a PR against the default branch in the source `VisualMeaning` repository.

Remember that the whole point of a PR is for another human to at least sanity check your code. They are not going to want to do this if the changes in your PR run to thousands and thousands of lines - PRs should ideally be scoped as tightly as possible, and not sprawl across multiple change topics. Sometimes long PRs are unavoidable, but as a general rule if your PR diff is approaching a thousand lines of new/changed code you should consider splitting it up into multiple smaller PRs.

### PR description

You will need to write a PR description. This should be a **human-written** summary of the changes proposed in the PR. It does not need to be an exhaustive summary of every line as that will be obvious from the code diff, just what the point of the change is and a broad overview of what has changed.

**Do not ask agentic AI to summarise the changes in your PR for you**. Writing the PR description yourself is good evidence that you have at least read and understood the code that you are proposing. A good pull request review is fundamentally a conversation between humans, and this conversation cannot happen unless both participants are working from the same base context of what the PR is about. The first thing the reviewer of your PR will do is read this description to get a sense of the context they should keep in mind when reviewing the actual code.

Planned code changes should be linked to work items in either Jira or Productive. Include a link to the relevant ticket in the PR description if one is available.

### Choosing a reviewer

You will also have to choose a reviewer. We have two groups set up in GitHub who are able to review:

- The `VisualMeaning/devs` group can be assigned as reviewers on all PRs across all repositories.
- The `VisualMeaning/pipeliners` group can be assigned as reviewers in specific repositories relevant to content generation.

As a rule of thumb, if the `pipeliners` group is available as a reviewer option in a repository, use that. If not, use `devs`.

## Reviewing a PR

The first thing to remember when reviewing a PR is: you do not need to go into it as an expert. You do not even need all of the context around the changes. You are reviewing something a human has submitted and taken responsibility for, and you should be able to have a conversation with that human.

The goal is to get to a point where you, as the reviewer, understand the logic behind the changes. 

As a corollary, this means that you should not be expected to review code written in languages you have no experience with. We do not expect people with zero exposure to TTL or SPARQL to review TTL or SPARQL unsupported, just like we don't expect people with zero exposure to Python or Javascript to review anything past very very simple and safe one-line changes. 

A reviewer should at least have a basic level of ability to read the code they're reviewing, just like someone reading a book written in English would need a basic level of English to understand it. They don't need to understand all of the words, they just need to understand enough to piece together what is going on. That is all.

This lets you get your foot in the door to start the review. Anything you don't understand from that point on, look it up. If it still doesn't make sense, ask the person who submitted the PR what their intent was. You don't know what you don't know; it's possible you don't have enough experience with the code to follow it completely (in which case asking about it is a valuable chance to learn), but it's also possible you've just found an issue with the PR.

If a PR is too long (rule of thumb is > 1000 lines being the point where it's difficult for a reviewer to hold everything in their head, although this will fluctuate depending on the nature of the changes) then you should think about rejecting the PR with the Request Changes status (see below) asking for the PR to split into multiple smaller PRs. 

### How to start a review

Open the PR in GitHub, go to the "Files Changed" tab, and start reading through the diff. Red lines are deletions/changes (previous state), green lines are additions/changes (new state).

Mousing over any line in either of the previous/changed states will cause a blue + symbol to appear. Clicking on the line allows you to leave comments on specific lines of code. Clicking and dragging up/down allows you to select whole code blocks to leave comments on.

Go through the changes proposed in the PR and leave comments on things that look like a problem (e.g. unclear function/variable naming, commented out code, hardcoded secrets/credentials), or which you don't understand.

When you are done, click the green "Submit Review" button in the top right-hand corner of the diff interface. This will allow you to make a final comment summarising your overall thoughts about the PR, along with the review outcome. The final comment does not need to be tremendously detailed, and if the code looked sane and you didn't leave any comments it's fine for it to be a simple one-word "Thanks!".

## Review outcome

A PR review can be submitted with one of the following three statuses.

**Request changes**. There are significant problems with the PR that must be addressed. These should be clearly communicated in comments during review, as well as called out in the summary comment when submitting the review. It is then the reponsibility of the PR author to address the comments through further code changes or discussion before requesting a re-review. 

An important thing to be aware of when requesting changes: submitting a review with this status will block the PR from being merged until the person who requested the changes submits a second, approving review. Approvals from other reviewers don't count; the original reviewer must remove this block by approving themselves.

**Comment**. A non-committal review that neither approves nor denies merging. This is typically only used by people who are only reviewing part of the PR and who want to have a discussion without taking responsibility for reviewing the whole thing, although we also sometimes use it to request follow-up information without explicitly rejecting the PR.

It is not to be expected that a reviewer who has Commented has exhaustively read the whole PR. They will likely have skim-read most of it, but the baseline assumption should be that they have _not_ read anything that they haven't explicitly commented on.

**Approval**. The code makes sense to the reviewer and does what the PR description says it does. There might be minor questions or code suggestions in the review comments, but these do not warrant a re-review.

## Responding to review

We trust that review feedback will be taken seriously. If a PR has been approved with comments, it is the responsbility of the PR author to make sure that the comments have been addressed prior to merging. This can be via further (minor) code changes or a reply to the comments explaining why the change will not be made.

Minor code changes which address comments can be pushed as additional commits to the same working branch without requiring a re-review.

Any significant code changes that go beyond responding to comments effectively invalidate the original PR review, as you now have code changes that the original reviewer did not review and is not expecting. These **do** require a re-review.

## Merging a PR

Once a PR has been approved the PR author can merge their working branch into the default branch at any time. (Assuming nobody has Requested Changes, as per above.)

We do not permit code to be merged to our default branches without at least one approving review from a human. Any CI (linting, automated tests) must also be in a passing state. Merging will not be possible if either of these conditions are not true.

When merging, we prefer to use the "Squash and Merge" option - this will combine all of the additional commits in your working branch down to a single commit which is then applied to the the default branch.

GitHub will surface a prompt asking if you want to delete your working branch after merging - you should click "Yes" since you don't need it any more. Any changes contained within are now part of the default branch, and so if you want to make further changes you can simply make a new working branch from the updated default branch state.
