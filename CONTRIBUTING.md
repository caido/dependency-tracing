# Contributing to Tracing

:balloon: Thanks for your help improving the project! We are so happy to have
you!

There are opportunities to contribute to Tracing at any level. It doesn't
matter if you are just getting started with Rust or are the most weathered
expert, we can use your help.

**No contribution is too small and all contributions are valued.**

This guide will help you get started. **Do not let this guide intimidate you**.
It should be considered a map to help you navigate the process.

You may also get help with contributing in the [dev channel][dev], please join
us!

Tracing is a part of the [Tokio][tokio] project, and follows the project's
guidelines for contributing. This document is based on the
[`CONTRIBUTING.md` file][tokio-contrib] in the `tokio-rs/tokio` repository.

[dev]: https://gitter.im/tokio-rs/dev
[tokio]: https://tokio.rs
[tokio-contrib]: https://github.com/tokio-rs/tokio/blob/master/CONTRIBUTING.md

## Conduct

The Tokio project adheres to the [Rust Code of Conduct][coc]. This describes
the _minimum_ behavior expected from all contributors.

[coc]: https://github.com/rust-lang/rust/blob/master/CODE_OF_CONDUCT.md

## Contributing in Issues

For any issue, there are fundamentally three ways an individual can contribute:

1. By opening the issue for discussion: For instance, if you believe that you
   have uncovered a bug in a Tracing crate, creating a new issue in the
   tokio-rs/tracing [issue tracker][issues] is the way to report it.

2. By helping to triage the issue: This can be done by providing
   supporting details (a test case that demonstrates a bug), providing
   suggestions on how to address the issue, or ensuring that the issue is tagged
   correctly.

3. By helping to resolve the issue: Typically this is done either in the form of
   demonstrating that the issue reported is not a problem after all, or more
   often, by opening a Pull Request that changes some bit of something in
   Tokio in a concrete and reviewable manner.

**Anybody can participate in any stage of contribution**. We urge you to
participate in the discussion around bugs and participate in reviewing PRs.

[issues]: https://github.com/tokio-rs/tracing/issues

### Asking for General Help

If you have reviewed existing documentation and still have questions or are
having problems, you can open an issue asking for help.

In exchange for receiving help, we ask that you contribute back a documentation
PR that helps others avoid the problems that you encountered.

### Submitting a Bug Report

When opening a new issue in the `tracing` issue tracker, users will
be presented with a [basic template][issue-template] that should be filled in. If you
believe that you have uncovered a bug, please fill out this form, following the
template to the best of your ability. Do not worry if you cannot answer every
detail, just fill in what you can.

The two most important pieces of information we need in order to properly
evaluate the report is a description of the behavior you are seeing and a simple
test case we can use to recreate the problem on our own. If we cannot recreate
the issue, it becomes impossible for us to fix.

In order to rule out the possibility of bugs introduced by userland code, test
cases should be limited, as much as possible, to using only Tokio APIs.

See [How to create a Minimal, Complete, and Verifiable example][mcve].

[mcve]: https://stackoverflow.com/help/mcve
[issue-template]: .github/ISSUE_TEMPLATE/bug_report.md

### Triaging a Bug Report

Once an issue has been opened, it is not uncommon for there to be discussion
around it. Some contributors may have differing opinions about the issue,
including whether the behavior being seen is a bug or a feature. This discussion
is part of the process and should be kept focused, helpful, and professional.

Short, clipped responses—that provide neither additional context nor supporting
detail—are not helpful or professional. To many, such responses are simply
annoying and unfriendly.

Contributors are encouraged to help one another make forward progress as much as
possible, empowering one another to solve issues collaboratively. If you choose
to comment on an issue that you feel either is not a problem that needs to be
fixed, or if you encounter information in an issue that you feel is incorrect,
explain why you feel that way with additional supporting context, and be willing
to be convinced that you may be wrong. By doing so, we can often reach the
correct outcome much faster.

### Resolving a Bug Report

In the majority of cases, issues are resolved by opening a Pull Request. The
process for opening and reviewing a Pull Request is similar to that of opening
and triaging issues, but carries with it a necessary review and approval
workflow that ensures that the proposed changes meet the minimal quality and
functional guidelines of the Tokio project.

## Pull Requests

Pull Requests are the way concrete changes are made to the code, documentation,
and dependencies in the `tracing` repository.

Even tiny pull requests (e.g., one character pull request fixing a typo in API
documentation) are greatly appreciated. Before making a large change, it is
usually a good idea to first open an issue describing the change to solicit
feedback and guidance. This will increase the likelihood of the PR getting
merged.

### Tests

If the change being proposed alters code (as opposed to only documentation for
example), it is either adding new functionality to a crate or it is fixing
existing, broken functionality. In both of these cases, the pull request should
include one or more tests to ensure that the crate does not regress in the future.
There are two ways to write tests: integration tests and documentation tests
(Tokio avoids unit tests as much as possible).

#### Integration tests

Integration tests go in the same crate as the code they are testing. Each sub
crate should have a `dev-dependency` on `tracing` itself. This makes all
`tracing` utilities available to use in tests, no matter the crate being
tested.

The best strategy for writing a new integration test is to look at existing
integration tests in the crate and follow the style.

#### Documentation tests

Ideally, every API has at least one [documentation test] that demonstrates how to
use the API. Documentation tests are run with `cargo test --doc`. This ensures
that the example is correct and provides additional test coverage.

The trick to documentation tests is striking a balance between being succinct
for a reader to understand and actually testing the API.

The type level example for `tokio_timer::Timeout` provides a good example of a
documentation test:

```
/// // import the `timeout` function, usually this is done
/// // with `use tokio::prelude::*`
/// use tokio::prelude::FutureExt;
/// use futures::Stream;
/// use futures::sync::mpsc;
/// use std::time::Duration;
///
/// # fn main() {
/// let (tx, rx) = mpsc::unbounded();
/// # tx.unbounded_send(()).unwrap();
/// # drop(tx);
///
/// let process = rx.for_each(|item| {
///     // do something with `item`
/// # drop(item);
/// # Ok(())
/// });
///
/// # tokio::runtime::current_thread::block_on_all(
/// // Wrap the future with a `Timeout` set to expire in 10 milliseconds.
/// process.timeout(Duration::from_millis(10))
/// # ).unwrap();
/// # }
```

Given that this is a *type* level documentation test and the primary way users
of `tokio` will create an instance of `Timeout` is by using
`FutureExt::timeout`, this is how the documentation test is structured.

Lines that start with `/// #` are removed when the documentation is generated.
They are only there to get the test to run. The `block_on_all` function is the
easiest way to execute a future from a test.

If this were a documentation test for the `Timeout::new` function, then the
example would explicitly use `Timeout::new`. For example:

```
/// use tokio::timer::Timeout;
/// use futures::Future;
/// use futures::sync::oneshot;
/// use std::time::Duration;
///
/// # fn main() {
/// let (tx, rx) = oneshot::channel();
/// # tx.send(()).unwrap();
///
/// # tokio::runtime::current_thread::block_on_all(
/// // Wrap the future with a `Timeout` set to expire in 10 milliseconds.
/// Timeout::new(rx, Duration::from_millis(10))
/// # ).unwrap();
/// # }
```

To reduce the effort required to review documentation-related changes,
`tracing`'s CI system generates preview websites containing the
`rustdoc` output for the entire repository for each PR. The preview can
be viewed by clicking the `details` link on the
`netlify/tracing-rs/deploy-preview check` on all pull requests.

### Building Documentation

Tracing's documentation uses nightly-only RustDoc features and lints, like
`doc(cfg)` and `broken_intra_doc_lints`. These features are enabled by
passing `--cfg docsrs` to RustDoc. Therefore, in order to build Tracing's
documentation the same way it would be built by docs.rs, it's necessary to
use the following command:

```bash
RUSTDOCFLAGS="--cfg docsrs" cargo +nightly doc --no-deps
```

### Commits

It is a recommended best practice to keep your changes as logically grouped as
possible within individual commits. There is no limit to the number of commits
any single Pull Request may have, and many contributors find it easier to review
changes that are split across multiple commits.

That said, if you have a number of commits that are "checkpoints" and don't
represent a single logical change, please squash those together.

Note that multiple commits often get squashed when they are landed (see the
notes about [commit squashing]).

#### Commit message guidelines

A good commit message should describe what changed and why.

1. The first line should:

  * contain a short description of the change (preferably 50 characters or less,
    and no more than 72 characters)
  * be entirely in lowercase with the exception of proper nouns, acronyms, and
    the words that refer to code, like function/variable names
  * be prefixed with the name of the crate being changed (without the
    `tracing` prefix) and start with an imperative verb.

  Examples:

  * fmt: add regex for parsing field filters
  * tower-http: add `Clone` impl for `Service` and `MakeService`

2. Keep the second line blank.
3. Wrap all other lines at 72 columns (except for long URLs).
4. If your patch fixes an open issue, you can add a reference to it at the end
   of the log. Use the `Fixes: #` prefix and the issue number. For other
   references use `Refs: #`. `Refs` may include multiple issues, separated by a
   comma.

   Examples:

   - `Fixes: #1337`
   - `Refs: #1234`

Sample complete commit message:

```txt
subcrate: explain the commit in one line

Body of commit message is a few lines of text, explaining things
in more detail, possibly giving some background about the issue
being fixed, etc.

The body of the commit message can be several paragraphs, and
please do proper word-wrap and keep columns shorter than about
72 characters or so. That way, `git log` will show things
nicely even when it is indented.

Fixes: #1337
Refs: #453, #154
```

### Opening the Pull Request

From within GitHub, opening a new Pull Request will present you with a
[pull-request-template] that should be filled out. Please try to do your best at filling out
the details, but feel free to skip parts if you're not sure what to put.

[pull-request-template]: .github/PULL_REQUEST_TEMPLATE.md

### Discuss and update

You will probably get feedback or requests for changes to your Pull Request.
This is a big part of the submission process so don't be discouraged! Some
contributors may sign off on the Pull Request right away, others may have
more detailed comments or feedback. This is a necessary part of the process
in order to evaluate whether the changes are correct and necessary.

**Any community member can review a PR and you might get conflicting feedback**.
Keep an eye out for comments from code owners to provide guidance on conflicting
feedback.

**Once the PR is open, do not rebase the commits**. See [Commit Squashing] for
more details.

### Commit Squashing

In most cases, **do not squash commits that you add to your Pull Request during
the review process**. When the commits in your Pull Request land, they may be
squashed into one commit per logical change. Metadata will be added to the
commit message (including links to the Pull Request, links to relevant issues,
and the names of the reviewers). The commit history of your Pull Request,
however, will stay intact on the Pull Request page.

## Reviewing Pull Requests

**Any Tokio community member is welcome to review any pull request**.

All Tokio contributors who choose to review and provide feedback on Pull
Requests have a responsibility to both the project and the individual making the
contribution. Reviews and feedback must be helpful, insightful, and geared
towards improving the contribution as opposed to simply blocking it. If there
are reasons why you feel the PR should not land, explain what those are. Do not
expect to be able to block a Pull Request from advancing simply because you say
"No" without giving an explanation. Be open to having your mind changed. Be open
to working with the contributor to make the Pull Request better.

Reviews that are dismissive or disrespectful of the contributor or any other
reviewers are strictly counter to the Code of Conduct.

When reviewing a Pull Request, the primary goals are for the codebase to improve
and for the person submitting the request to succeed. **Even if a Pull Request
does not land, the submitters should come away from the experience feeling like
their effort was not wasted or unappreciated**. Every Pull Request from a new
contributor is an opportunity to grow the community.

### Review a bit at a time.

Do not overwhelm new contributors.

It is tempting to micro-optimize and make everything about relative performance,
perfect grammar, or exact style matches. Do not succumb to that temptation.

Focus first on the most significant aspects of the change:

1. Does this change make sense for Tokio?
2. Does this change make Tokio better, even if only incrementally?
3. Are there clear bugs or larger scale issues that need attending to?
4. Is the commit message readable and correct? If it contains a breaking change
   is it clear enough?

Note that only **incremental** improvement is needed to land a PR. This means
that the PR does not need to be perfect, only better than the status quo. Follow
up PRs may be opened to continue iterating.

When changes are necessary, *request* them, do not *demand* them, and **do not
assume that the submitter already knows how to add a test or run a benchmark**.

Specific performance optimization techniques, coding styles and conventions
change over time. The first impression you give to a new contributor never does.

Nits (requests for small changes that are not essential) are fine, but try to
avoid stalling the Pull Request. Most nits can typically be fixed by the Tokio
Collaborator landing the Pull Request but they can also be an opportunity for
the contributor to learn a bit more about the project.

It is always good to clearly indicate nits when you comment: e.g.
`Nit: change foo() to bar(). But this is not blocking.`

If your comments were addressed but were not folded automatically after new
commits or if they proved to be mistaken, please, [hide them][hiding-a-comment]
with the appropriate reason to keep the conversation flow concise and relevant.

### Be aware of the person behind the code

Be aware that *how* you communicate requests and reviews in your feedback can
have a significant impact on the success of the Pull Request. Yes, we may land
a particular change that makes `tracing` better, but the individual might
just not want to have anything to do with `tracing` ever again. The goal is
not just having good code.

### Abandoned or Stalled Pull Requests

If a Pull Request appears to be abandoned or stalled, it is polite to first
check with the contributor to see if they intend to continue the work before
checking if they would mind if you took it over (especially if it just has nits
left). When doing so, it is courteous to give the original contributor credit
for the work they started (either by preserving their name and email address in
the commit log, or by using an `Author: ` meta-data tag in the commit.

_Adapted from the [Node.js contributing guide][node]_.

[node]: https://github.com/nodejs/node/blob/master/CONTRIBUTING.md
[hiding-a-comment]: https://help.github.com/articles/managing-disruptive-comments/#hiding-a-comment
[documentation test]: https://doc.rust-lang.org/rustdoc/documentation-tests.html

## Releasing

Since the Tracing project consists of a number of crates, many of which depend on
each other, releasing new versions to crates.io can involve some complexities.
When releasing a new version of a crate, follow these steps:

1. **Ensure that the release crate has no path dependencies.** When the HEAD
   version of a Tracing crate requires unreleased changes in another Tracing crate,
   the crates.io dependency on the second crate will be replaced with a path
   dependency. Crates with path dependencies cannot be published, so before
   publishing the dependent crate, any path dependencies must also be published.
   This should be done through a form of depth-first tree traversal:

   1. Starting with the first path dependency in the crate to be released,
      inspect the `Cargo.toml` for the dependency. If the dependency has any
      path dependencies of its own, repeat this step with the first such
      dependency.
   2. Begin the release process for the path dependency.
   3. Once the path dependency has been published to crates.io, update the
      dependent crate to depend on the crates.io version.
   4. When all path dependencies have been published, the dependent crate may
      be published.

   To verify that a crate is ready to publish, run:

   ```bash
   bin/publish --dry-run <CRATE NAME> <CRATE VERSION>
   ```

2. **Update Cargo metadata.** After releasing any path dependencies, update the
   `version` field in `Cargo.toml` to the new version, and the `documentation`
   field to the docs.rs URL of the new version.
3. **Update the "Documentation" link in the crate's
   `README.md` to point to the docs.rs URL of the new version.
4. **Update the changelog for the crate.** Each crate in the Tokio repository
   has its own `CHANGELOG.md` in that crate's subdirectory. Any changes to that
   crate since the last release should be added to the changelog. Change
   descriptions may be taken from the Git history, but should be edited to
   ensure a consistent format, based on [Keep A Changelog][keep-a-changelog].
   Other entries in that crate's changelog may also be used for reference.
5. **Perform a final audit for breaking changes.** Compare the HEAD version of
   crate with the Git tag for the most recent release version. If there are any
   breaking API changes, determine if those changes can be made without breaking
   existing APIs. If so, resolve those issues. Otherwise, if it is necessary to
   make a breaking release, update the version numbers to reflect this.
6. **Open a pull request with your changes.** Once that pull request has been
   approved by a maintainer and the pull request has been merged, continue to
   the next step.
7. **Release the crate.** Run the following command:

   ```bash
   bin/publish <NAME OF CRATE> <VERSION>
   ```

   Your editor and prompt you to edit a message for the tag. Copy the changelog
   entry for that release version into your editor and close the window.

[keep-a-changelog]: https://github.com/olivierlacan/keep-a-changelog/blob/master/CHANGELOG.md
