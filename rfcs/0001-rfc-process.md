---
Status: Accepted
Champions: @justinfagnani
PR:
---

# Lit RFC Process

The Lit RFC ("request for comment") process is a way for "substantial" changes to Lit to be proposed, discussed, designed, refined, and ultimately reach consensus before being implemented or merged into the Lit core libraries.

## Objective

### Goals

* Bring increased visibility, community feedback, and deliberation to feature requests
* Allow our community to better observe and participate in the Lit team design processes
* Create more opportunities for community contributions: making and editing RFCs, feedback, implementation, etc.
* Raise the quality of feature requests
* Have a consistent and known process for evaluating feature requests
* Create a public record of change discussions, especially rationale for accepting or rejecting a feature request
* Raise the barrier _just a bit_ to making larger feature requests to help with volume, noise, and quality
* Move large and likely longer-to-be-open feature requests out of issues, helping us maintain a smaller and cleaner issue backlog

## What is an RFC?

RFC refers to the IETF's [Request For Comments](https://en.wikipedia.org/wiki/Request_for_Comments) process. An RFC is a document or set of documents that describe a feature in detail, and go through a process of review and approval.

For Lit, RFCs are markdown files in the lit/rfcs repository.

RFCs have a structure as defined in the [RFC template](./0000-rfc-template.md), including these metadata and sections (the RCF template will include more descriptions):

* Champion(s): The people making and maintaining the proposal. This can change over time.
* Stage: Accepted, Deprecated, Implemented
* Summary
* Objective (goals and non-goals)
* Motivation (background)
* Detailed Design, including these recommended sections:
    * Implementation Plan
    * Backward Compatibility
    * Testing Plan
    * Performance and Code Size Impact
    * Interoperability
    * Security Impact
    * Documentation Plan
* Downsides
* Alternatives

Accepted RFCs are not a guarantee that a feature will be developed and included in Lit. They are an indication that the team has consensus on a feature and design, and is open to accepting PRs to implement it. Things can change, and new information such as implementation feedback may lead the team to deprecate an RFC.

## When to use RFCs

RFCs must be created for new and "substantial" changes. "Substantial" is somewhat subjective, but some clear signs are:

* New packages
* New modules, classes, directives, etc. in an existing package
* Most new public-facing API
* Breaking changes

Changes that do _not_ require an RFC:

* Internal implementation changes
* Bug fixes
* Most dev mode changes: warnings, etc.

These criteria leave a category of small changes and experiments open to debate on whether they deserve or require an RFC. We do not wish to slow down development with too much process, so we will leave this area to the discretion of the core team. We will adjust the criteria for requiring RCFs over time.

#### Relationship to Lit Labs

The Lit project also has a labs process for pre-production experimentation and feedback. With the assumption that when we create a labs package we usually intend for it to eventually graduate, most labs packages should have (or be part of) an RFC.

## RFC Process

### Before opening an RFC

RFCs involve more work than an issue, both to open one and to review one. Before opening an RFC is it a good idea to get some indication of interest from the Lit core team or strong support from the community.

You may get this indication in several ways, from informal to formal:

* A feature request issue is closed with a request to open an RFC
* A core team member directly invites you to open an RFC from a community or support channel like GitHub Discussions, Slack, or Twitter
* You open a "[RRFC](https://github.com/lit/rfcs/issues/new)" (Requesting Request For Comment) issue in lit/rfcs and get positive feedback 

### Opening an RFC

1. Fork the lit/rfcs repo
2. Draft the RFC
    1. Add the RFC file to /rfcs in a markdown file named `NNNN-_descriptive-name_.md`
        1. Note the `NNNN` is literal. RFCs will not have an assigned RFC number when first opened.
3. Open an RFC pull request
    1. Optionally share the draft on [GH discussions](https://github.com/lit/lit/discussions) or [Slack](https://lit.dev/slack-invite/) before opening a pull request
    2. Once opened, update the PR with a link to the RFC PR in the metadata. This allows readers of the RFCs to find the review discussions.

### Discussion and Iteration

RFCs must be discussed and reviewed by the relevant maintainers, Lit project leadership, and other stakeholders before being accepted or rejected.

#### Reviewers

RFCs must be reviewed by Lit project leadership and by project area maintainers. Lit project leadership who can approve RFCs are listed in the lit/rfcs repo's [CODEOWNERS](../CODEOWNERS) file. RFCs must be approved by leadership before moving to accepted stage and being merged. Area maintainers are sometimes identifiable by who is active on certain projects in the lit/lit monorepo, but the Lit team can add the appropriate people as reviews if it's unclear.

#### Timeline

Many RFC processes have a prescribed timeline for discussion. We aren't yet sure what timelines for feedback are both most productive and attainable. For the time being we aim to have initial feedback within about 7 days and an accepted or rejected decision within about 30 days.

#### Acceptance

Once accepted, RFCs are assigned an RFC number and pull requests are updated and merged. This means that all RFCs in lit/rfcs start as accepted. Rejected RFC pull requests are closed.

#### RFC Meetings

RFC stakeholders may decide that RFCs need or could benefit from live community discussion. They will be added to the RFC meetings agenda list and discussed at the next opportunity.

### Updates and Amendments

After being accepted RFCs are intended to mostly stay unchanged.

Major changes proposed to an RFC must be done in a new RFC. If a new RFC deprecates a previous RFC, it must mark the previous RFC as deprecated with a link to the new RFC. Otherwise the new RFC should update the previous RFC with a link and callouts to the new RFCs in relevant places.

Small changes to RFCs can be made to:

* Add clarifications
* Update metadata and other non-design text (examples, documentation)
* Minor changes to implementation, possibly based on implementor feedback

### Implementation

As RFCs are implemented, they should be updated to indicate so. The status is updated to *Implemented* and relevant PRs linked.

## Other RFC Processes

The projects have served as inspiration for our process:

* [Ember](https://github.com/emberjs/rfcs)
* [npm](https://github.com/npm/rfcs)
* [Fuchsia](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/docs/contribute/governance/rfcs/rfc_process.md)
