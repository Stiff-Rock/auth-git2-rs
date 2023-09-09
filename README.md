# auth-git2

Easy authentication for [`git2`].

Authentication with [`git2`] can be quite difficult to implement correctly.
This crate aims to make it easy.

## Features

* Small dependency tree.
* Query the SSH agent for private key authentication.
* Get SSH keys from files.
* Prompt the user for passwords for encrypted SSH keys.
** Only supported for OpenSSH private keys.
* Query the git credential helper for usernames and passwords.
* Use pre-provided plain usernames and passwords.
* Use the git askpass helper to ask the user for credentials.
* Fallback to prompting the user on the terminal if there is no askpass helper.

## Creating an authenticator and enabling authentication mechanisms

You can create use [`GitAuthenticator::new()`] (or [`default()`][`GitAuthenticator::default()`]) to create a ready-to-use authenticator.
Using one of these constructors will enable all supported authentication mechanisms.
You can still add more private key files from non-default locations to try if desired.

You can also use [`GitAuthenticator::new_empty()`] to create an authenticator without any authentication mechanism enabled.
Then you can selectively enable authentication mechanisms and add custom private key files.
and selectively enable authentication methods or add private key files.

## Using the authenticator

For the most flexibility, you can get a [`git2::Credentials`] callback using the [`GitAuthenticator::credentials()`] function.
You can use it with any git operation that requires authentication.
Doing this gives you full control to set other options and callbacks for the git operation.

If you don't need to set other options or callbacks, you can also use the convenience functions on [`GitAuthenticator`].
They wrap git operations with the credentials callback set:

* [`GitAuthenticator::clone_repo()`]
* [`GitAuthenticator::fetch()`]
* [`GitAuthenticator::push()`]

## Example: Clone a repository

```rust
use auth_git2::GitAuthenticator;
use std::path::Path;

let url = "https://github.com/de-vri-es/auth-git2-rs";
let into = Path::new("/tmp/dyfhxoaj/auth-git2-rs");

let auth = GitAuthenticator::default();
let mut repo = auth.clone_repo(url, into);
```

## Example: Clone a repository with full control over fetch options

```rust
use auth_git2::GitAuthenticator;
use std::path::Path;

let auth = GitAuthenticator::default();
let git_config = git2::Config::open_default()?;
let mut repo_builder = git2::build::RepoBuilder::new();
let mut fetch_options = git2::FetchOptions::new();
let mut remote_callbacks = git2::RemoteCallbacks::new();

remote_callbacks.credentials(auth.credentials(&git_config));
fetch_options.remote_callbacks(remote_callbacks);
repo_builder.fetch_options(fetch_options);

let url = "https://github.com/de-vri-es/auth-git2-rs";
let into = Path::new("/tmp/dyfhxoaj/auth-git2-rs");
let mut repo = repo_builder.clone(url, into);
```

[`git2`]: https://docs.rs/git2
[`GitAuthenticator`]: https://docs.rs/auth-git2/latest/auth_git2/struct.GitAuthenticator.html
[`GitAuthenticator::new()`]: https://docs.rs/auth-git2/latest/auth_git2/struct.GitAuthenticator.html#method.new
[`GitAuthenticator::default()`]: https://docs.rs/auth-git2/latest/auth_git2/struct.GitAuthenticator.html#method.default
[`GitAuthenticator::new_empty()`]: https://docs.rs/auth-git2/latest/auth_git2/struct.GitAuthenticator.html#method.new_empty
[`git2::Credentials`]: https://docs.rs/git2/latest/git2/type.Credentials.html
[`GitAuthenticator::credentials()`]: https://docs.rs/auth-git2/latest/auth_git2/struct.GitAuthenticator.html#method.credentials
[`GitAuthenticator::clone_repo()`]: https://docs.rs/auth-git2/latest/auth_git2/struct.GitAuthenticator.html#method.clone_repo
[`GitAuthenticator::fetch()`]: https://docs.rs/auth-git2/latest/auth_git2/struct.GitAuthenticator.html#method.fetch
[`GitAuthenticator::push()`]: https://docs.rs/auth-git2/latest/auth_git2/struct.GitAuthenticator.html#method.push
