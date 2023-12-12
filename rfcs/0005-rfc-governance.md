---
Status: Active
Champions: smalls
PR: {{ update_with_pr_number }}
---

# Governance

Update & clarify Lit's governance mode

## Objective

The Lit project works toward a better, lower-friction web platform by promoting
interoperable standards for existing and emergent developer needs, and
fostering an open and healthy ecosystem around these standards.

We'd like to apply these same values to our project. In this RFC, we propose
updating the Lit governance model (documented primarily in a new file,
`GOVERNANCE.md`) to make it clear that our community members have an important
role in deciding the future of Lit. While Google owns and operates the Lit
project repo, project roles are open to all contributors.

### Goals
- Specific goals of the RFC

TODO

### Non-Goals
- Specific non-goals of the RFC

TODO

## Motivation

Why is the RFC necessary? What background information is needed to understand why?

TODO

## Detailed Design

If this RFC is accepted, we will propose a PR for further iteration, using the
language here as a starting point. This RFC also includes initial lists of
Members and Maintainers (which will become `MEMBERS.md` and `MAINTAINERS.md` in
the PR).

### GOVERNANCE.md

### Mission, Vision, Values

We seek to build a better, lower-friction web platform by promoting
interoperable standards for existing and emergent developer needs. Whereas most
web frameworks ask "how might we best solve this developer need in userspace?",
on the Lit project we ask "how might this need be met by a new interoperable
standard, either as a new web platform feature or as a shared protocol
implementable by any framework?" As a corollary, our library development is
very tightly linked to developments in browser standards/implementation,
building on long-term relationships with Chrome, the W3C, community groups,
etc.

A key tenent of our decision-making framework involves applying this "ethos" of
standards-based interoperability to questions of features, roadmap, and
prioritization – and this tenent is both a core value and a key differentiator
of Lit. Decisions at all levels (from release planning to approving new
features) are judged through this lens.

We strive for transparency in our decision making and regularly look to the
project community for feedback. The Lit project repo is currently owned and
managed by Google; however project membership and Maintainer status is open to
all contributors of the project. Project roles and responsibilities are given
to the people, not their employer. This includes members of the Lit team at
Google; not all members of the team will be project Members or Maintainers, and
they will go through the process outlined here to achieve additional roles.

### How We Work

Lit strives to operate in the open, with full transparency to all members of
our community.

#### Issues and Tasks

These are tracked in Github and visible to all. We hold weekly engineering (see
our [calendar](https://lit.dev/community-calendar/) meetings where Members and
Maintainers review progress and discuss open topics. 

#### Features / Substantial Changes

Lit uses a Request for Comment process (RFC) to evaluate substantial changes to
the project, including governance.  Any community member may open an RFC, and
we welcome input from all. Approval of them is limited to Maintainers; an RFC
is approved when more than half of all Maintainers vote to approve the RFC.

#### Releases

The project uses a monorepo that is generally in a state such that it can be
released at any time. Releases are typically done at ad-hoc intervals based on
the need/desire to publish specific fixes and features. Major version changes
are rare and are done with large community awareness and outreach.

The release process is driven by the Maintainers of Lit.

#### Communication

Github is our “source of truth” for project discussions and decisions. At times
when a quick answer is needed, we have a Discord available for chat. We
recognize that chat is ephemeral and therefore you may be asked to create an
issue or RFC for the question you raise. This is so that all in the community
are able to be involved in the discussion, and decisions are not made in silos,
and so we have durable records of the discussions and decisions reached.  

### Contributors

In order to facilitate contributions at different levels, we have set up
different project roles at various levels of involvement. Community members
generally start at the first levels of the "ladder" and advance up it as their
responsibilities grow. A key attribute of contributors at each level is helping
others in the community - reviewing their code, helping them to use Lit,
recognizing when their contributions warrant a new role in the project.

We value transparency and openness in the Lit project, and as a result on the
rare occasion that exceptions must be made to this incremental progression
(e.g., a new maintainer is urgently needed or a new sub-project is accepted) we
will still follow the open process for joining at the required level. 

We do not expect this to be a static list of roles - as Lit continues to
evolve, so will our project needs. We will leverage our RFC process to document
these changes.

#### User

Our users are the most important role of Lit - without users, Lit is not
useful. Anyone can be a user and there are no special requirements.

We do encourage our users to interact with our broader community. Feedback
about successes is wonderful, as is hearing about how Lit could do better. Some
other ways that Users can support the project: By trying pre-releases and
giving us feedback Discussing Lit by word of mouth, in your communities, and
your projects Supporting others in the community By making new components, and
sharing them with the community Showing off apps that are built with Lit

We love to hear how our users are being successful with Lit. We have a special
[Showcase](https://discord.com/channels/1012791295170859069/1014310579852300439)
Discord channel especially for this purpose.

We ask that all of our users follow our [Code of
Conduct](https://github.com/lit/lit/blob/main/CODE_OF_CONDUCT.md) when
interacting with the project.

#### Contributor

A Contributor contributes directly to the project in some way. These
contributions don’t have to be code, but they can be - commenting on PRs,
filing issues, presenting at a community event are all contributions. Github
tracks code Contributors on a [contributors
list](https://github.com/lit/lit/graphs/contributors), but please keep in mind
this is not the exhaustive list of project contributors.

When making a contribution, please follow our [Contribution
Guide](https://github.com/lit/lit/blob/main/CONTRIBUTING.md).

#### Member

A Member of the Lit project is a Contributor with recognized, consistent
contributions to the project. They are actively pushing to the project, and
have enhanced permissions to enable this. They are likely to have an area of
focus (eg expertise with a Lit subsystem, our docs/lit.dev site, etc), or they
may have general expertise across the Lit codebase.

Members are eligible for enhanced GitHub permissions (GitHub Write and
[CODEOWNERS](https://github.com/lit/lit/blob/main/.github/CODEOWNERS)) for
their area of focus.

We ask that our Members continue to support and mentor our Contributors, to
help them grow into Members as well. This includes prioritizing reviews, and
providing good feedback on those reviews. When a Member believes that an
individual is ready to join the Members list, they should nominate them (by
creating a PR to amend `MEMBERS.md`); a majority of all Members is required to
approve that PR.


#### Maintainer

Maintainers are very established contributors who are responsible for the
entire project. 

They have the ability to approve PRs against any area of the project, and are
expected to participate in making decisions about the strategy and priorities
of the project. Maintainers are responsible for voting on RFCs, and may have
enhanced GitHub permissions (like GitHub Maintain) if needed.

We ask that our Maintainers support and mentor our Contributors and Members, to
help them grow into Maintainers as well. This includes prioritizing reviews,
and providing good feedback on proposals and RFCs. When a Member believes that
an individual is ready to join the Maintainers list, they should nominate them
(by creating a PR to amend `MAINTAINERS.md`). A majority of all Maintainers are
required to approve the PR adding the new Maintainer.

#### Stepping Down / Removal

If and when a Member or Maintainer’s commitment levels change, they can
consider stepping down (moving down the contributor ladder) or moving to
emeritus status (completely stepping away from the project).

Contact the Maintainers about changing to Emeritus status, or reducing your
contributor level.

#### Involuntary Removal or Demotion

Involuntary removal/demotion of a contributor happens when responsibilities and
requirements aren't being met. This may include repeated patterns of
inactivity, extended period of inactivity, a period of failing to meet the
requirements of your role, and/or a violation of the Code of Conduct. This
process is important because it protects the community and its deliverables
while also opens up opportunities for new contributors to step in.

Involuntary removal or demotion is handled through a vote by a majority of the
current Maintainers.

### Proposed `MEMBERS.md`

43081j
bencbradshaw
benjamind
bennypowers

### Proposed `MAINTAINERS.md`

AndrewJakubowicz
augustjk
bicknellr
dfreedm
e111077
graynorton
justinfagnani
kevinpschaaf
rictic
usergenic
sorvell

END END
END END
END END
END END
END END
END END
END END
END END
END END

## Implementation Considerations

### Implementation Plan

Is there anything important to note about implementation plan? Can it be done in a single PR or will it need to be staged out across several?

### Backward Compatibility

Backwards compatibility is extremely important to Lit, especially in the core libraries. Does this proposal break backwards compatibility? Are there workarounds?

### Testing Plan

How will this proposal be tested? Are unit tests sufficient, or do we need integration tests? Is any unique testing infrastructure required?

### Performance and Code Size Impact

What impact will this proposal have on performance and code size? What benchmarks should we create to evaluate the proposal?

### Interoperability

Is this proposal for a feature that could be interoperable across web components not written in Lit? Does it create a tight coupling between components written in Lit? Could it be a [Web Components Community Group](https://github.com/w3c/webcomponents-cg) [Community Protocol](https://github.com/webcomponents-cg/community-protocols)?

### Security Impact

What impact will this proposal have on security? Does the proposal require a security review? (We have a security team available for reviews)

We especially care about the handling of untrusted user input by library code so that we contnue to prevent XSS vectors.

### Documentation Plan

Do we need to create or update any documentation to complete this proposal? Does related documentation have a clear home in our docs outline? What playground examples or tutorials should be created?

## Downsides

Many proposals involve trade-offs. What are they for this proposal and what are the downsides of this approach?

## Alternatives

What alternatives were considered and rejected? Why?