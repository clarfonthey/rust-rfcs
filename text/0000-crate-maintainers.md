- Feature Name: N/A
- Start Date: YYYY-MM-DD
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

## Summary
[summary]: #summary

Modernise the crate maintainers team and standardise what it means to be an "official Rust crate."

## Motivation
[motivation]: #motivation

Right now, the crate maintainers team exists to account for the small handful of crates that are officially under the rust-lang GitHub org. These crates are:

* [`cc-rs`]
* [`cmake-rs`]
* [`compiler-builtins`]
* [`ferris-says`]
* [`flate2-rs`]
* [`getopts`]
* [`glob`]
* [`hashbrown`]
* [`libc`]
* [`libz-sys`]
* [`log`]
* [`pkg-config-rs`]
* [`regex`]
* [`socket2`]

[`cc-rs`]: https://github.com/rust-lang/cc-rs
[`cmake-rs`]: https://github.com/rust-lang/cmake-rs
[`compiler-builtins`]: https://github.com/rust-lang/compiler-builtins
[`ferris-says`]: https://github.com/rust-lang/ferris-says
[`flate2-rs`]: https://github.com/rust-lang/flate2-rs
[`getopts`]: https://github.com/rust-lang/getopts
[`glob`]: https://github.com/rust-lang/glob
[`hashbrown`]: https://github.com/rust-lang/hashbrown
[`libc`]: https://github.com/rust-lang/libc
[`libz-sys`]: https://github.com/rust-lang/libz-sys
[`log`]: https://github.com/rust-lang/log
[`pkg-config-rs`]: https://github.com/rust-lang/pkg-config-rs
[`regex`]: https://github.com/rust-lang/regex
[`socket2`]: https://github.com/rust-lang/socket2

Currently, the team operates on a level of mutual trust where maybe one or two people actually maintain any given crate, but everyone on the team has access to merge changes for these crates and publish new releases if CI supports this. But, this already runs into a few issues:

* `hashbrown` is currently co-owned by the library team proper, who until recently did not share ownership with crate maintainers
* `regex` is categorised under its own subteam separate from crate maintainers
* Oh! Right…

It also turns out that the rust-lang org has *a lot* of official crates outside the purview of this team, published on crates.io. I'm going to shorten the list *a little* to only include things that have dedicated repositories, which aren't prefixed by names of projects like `rustc_` or `cargo_`:

* [`annotate-snippets-rs`] (cargo, compiler, wg-diagnostics)
* [`ar_archive_writer`] (compiler)
* [`cargo-bisect-rustc`] (compiler)
* [`chalk`] (types)
* [`datafrog`] (compiler, wg-polonius)
* [`dlmalloc-rs`] (compiler, libs)
* [`ena`] (compiler)
* [`futures-rs`] (wg-async)
* [`gccjit.rs`] (wg-gcc-backend)
* [`git2-rs`] (cargo)
* [`impl-trait-utils`] (wg-async)
* [`jobserver-rs`] (cargo, compiler)
* [`literal-escaper`] (compiler)
* [`measureme`] (compiler)
* [`polonius`] (wg-polonius)
* [`rust-bindgen`] (wg-bindgen)
* [`rustwide`] (docs.rs, infra)
* [`ssh2-rs`] (cargo)
* [`stacker`] (compiler)
* [`thorin`] (compiler)

[`annotate-snippets-rs`]: https://github.com/rust-lang/annotate-snippets-rs
[`ar_archive_writer`]: https://github.com/rust-lang/ar_archive_writer
[`cargo-bisect-rustc`]: https://github.com/rust-lang/cargo-bisect-rustc
[`chalk`]: https://github.com/rust-lang/chalk
[`datafrog`]: https://github.com/rust-lang/datafrog
[`dlmalloc-rs`]: https://github.com/rust-lang/dlmalloc-rs
[`ena`]: https://github.com/rust-lang/ena
[`futures-rs`]: https://github.com/rust-lang/futures-rs
[`gccjit.rs`]: https://github.com/rust-lang/gccjit.rs
[`git2-rs`]: https://github.com/rust-lang/git2-rs
[`impl-trait-utils`]: https://github.com/rust-lang/impl-trait-utils
[`jobserver-rs`]: https://github.com/rust-lang/jobserver-rs
[`literal-escaper`]: https://github.com/rust-lang/literal-escaper
[`measureme`]: https://github.com/rust-lang/measureme
[`polonius`]: https://github.com/rust-lang/polonius
[`rust-bindgen`]: https://github.com/rust-lang/rust-bindgen
[`rustwide`]: https://github.com/rust-lang/rustwide
[`ssh2-rs`]: https://github.com/rust-lang/ssh2-rs
[`stacker`]: https://github.com/rust-lang/stacker
[`thorin`]: https://github.com/rust-lang/thorin

And note that there are plenty of other crates which are published separately from the usual release process, but stored in various repositories for various teams.

And to be clear, *this probably isn't the full list*. Right now, it's extremely difficult to tell:

1. Who should be responding to issues and PRs for any given crate
2. Who should be handling publishing/yanking versions for any given crate
3. Which teams "own" various crates
4. What the "chain of command" is for various crates (if one maintainer is unresponsive, who do I ask next?)
5. Which crates are even maintained any more

And to be clear, these aren't all problems to be solved by this RFC, but this RFC plans to address at least one of these problems: the crate maintainers subteam is *not* a scalable solution for dealing with the overwhelming number of crates under the rust-lang org with various levels of maintenance.

## Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

> [!NOTE]
> Since the guide-level/reference-level split is a bit weird for this RFC, I'm going to treat the guide-level explanation as a description of the *endgame state* of the crate maintainers subteam, while the reference-level explanation will go into some of the specifics of how we get there.

### Subteams

The crate maintainers team will remain a subteam of the library team, however, it will additionally be split into four parts:

* Crate maintainers
* Crate owners
* Trusted crate maintainers
* Trusted crate owners

Note that for actual logistical reasons, the "trusted" status may be simply an additional role under the relevent teams, however, it's simpler for the RFC to explain them as four distinct teams.

#### Crate owners

Crate owners are the users entrusted with publish and yank permissions for crates. Ownership is granular to the repository level, meaning that in general, users will be given publish permissions to all crates within certain repositories.

In general, every repository owned by the team must have at least one crate owner, although for "low-risk" crates like `ferris-says` the option is available to instead treat everyone on the crate owners subteam as their owner.

#### Trusted crate owners

Trusted crate owners are the subset of crate owners capable of making decisions about the crate maintainers team via the FCP process. At least three and at most eight members of the crate maintainers subteam should be delegated as trusted crate owners and this subset should be shuffled regularly. (explained later)

They are also given blanket permissions to publish and yank all crates owned by the team as a backup in case of emergencies.

#### Crate maintainers

Crate maintainers are given permission to maintain crates owned by the crate maintainers team. This does not change from the existing crate maintainers team, with the exception of publish permissions.

Certain repositories may be limited to a strict subset of the crate maintainers for various reasons, and maintainers may be given separate permissions to maintain these repositories.

#### Trusted crate maintainers

Trusted crate maintainers are members of the crate maintainers team with permission to maintain *all* crates on the crate maintainers subteam, disregarding individual repositories' preferences. These members are aware of the unique circumstances of every crate owned by the team *or* are trusted to ask in the cases they aren't aware of.

Trusted crate owners are all trusted crate maintainers, but you don't need to be a trusted crate maintainer to be a crate owner.

### Crates

Crates owned by the crate maintainers subteam are split into three bikesheddable tiers. These tiers *do not* indicate anything about the crates other than their permission structure within the crate maintainers team, with higher tiers indicating a smaller subset of users able to work on them.

* Tier-3 crates can be maintained by anyone on the crate maintainers team and are owned by the entire crate owners team.
* Tier-2 crates can be maintained by anyone on the crate maintainers team and have specific crate owners.
* Tier-1 crates have specific crate maintainers and specific crate owners.

Note that in all cases, trusted maintainers and owners are able to act as maintainers and owners for any crate on the team, including tier-1 crates.

### Team ownership

Some crates are effectively maintained by entire teams and do not make sense to limit maintenance solely to the crate maintainers subteam. For these crates, the limits on publishing and yanking versions still apply, but additional teams may be added on the repositories as effective maintainers.

Whenever this occurs, the leads of the owning team are treated as effective owners of the crates, allowing for the case where no owner is otherwise assigned to the crates via the crate owners subteam. Note that this is purely additive, and there may be additional owners on the crate owners subteam separate from these teams.

## Reference-level explanation
[reference-level-explanation]: #reference-level-explanation


## Drawbacks
[drawbacks]: #drawbacks

## Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

## Prior art
[prior-art]: #prior-art

## Unresolved questions
[unresolved-questions]: #unresolved-questions

## Future possibilities
[future-possibilities]: #future-possibilities

