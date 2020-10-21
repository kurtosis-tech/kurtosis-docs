Running in CI
=============
Running Kurtosis on your local machine is nice, but the real power of the platform comes when it's executed as part of CI. This guide will walk you through setting Kurtosis in your CI environment.

Machine-to-Machine Auth
-----------------------
While it's fine to prompt a user for their username and password when Kurtosis is run on the local machine, this method is insecure when executing Kurtosis on CI. Fortunately, our auth system [provides a system for handling this called "machine-to-machine auth"](https://auth0.com/docs/flows/client-credentials-flow). In the machine-to-machine flow, a client ID and a client secret are stored within the CI environment's secrets and passed in to every CI job. Kurtosis uses these credentials to retrieve a token from the auth provider, which allows Kurtosis execution to proceed.

The client ID and secret are created by the Kurtosis team on the backend, so if you don't have them already then shoot us a ping at [inquiries@kurtosistech.com](mailto:inquiries@kurtosistech.com) to discuss pricing!

**WARNING: Make sure you store your client ID and secret in a secure place! Anyone with the credentials could impersonate you to Kurtosis, which would use up any usage-based credits on your behalf.**

Using Client Credentials
------------------------
Now that you have client credentials, you'll need to pass them in to your CI environment as environment variables. The route for doing so will be particular to your CI server, so Google for "define YOUR_CI_TOOL secrets" for specifics. Once done, you'll need to pass the appropriate environment variables to Kurtosis. The Kurtosis initializer takes in special flags for receiving these, so we recommend:

1. [Visiting the Kurtosis documentation on running a testsuite](./testsuite-details.md##running-a-testsuite) to see what parameters to use and how to use them
2. Writing a wrapper script around `build_and_run.sh` that your CI will call to pass in the client ID and secret to Kurtosis

Client Credentials & Untrusted PRs
----------------------------------
Because the client credentials are secrets, they cannot be made available to builds of untrusted PRs (i.e. PRs from untrusted contributors, often submitted from forks) else there's a risk that the untrusted PR maliciously prints them. This means that CI builds for untrusted PRs will fail. This is a tough problem in the open-source community, but there is a workaround for those using Github:

1. Add nwtgck's "comment-run" Github Action to your repo [with these instructions](https://github.com/nwtgck/actions-comment-run#introduce-this-action) 
    * **NOTE 1:** Make sure to use the latest version, as these instructions hardcoded `v1.1` which is no longer the latest 
    * **NOTE 2:** As of 2020-10-21, [Github Actions seem to have a bug with assigning the OWNER role](https://github.community/t/github-actions-have-me-as-contributor-role-when-im-owner/138933) so you may need to fiddle with the `allowed-associations` section
1. Copy-paste [the PR merge preview comment-action](https://github.com/nwtgck/actions-comment-run#pr-merge-preview)
1. [Save it as one of your "Saved Replies" in Github](https://github.com/nwtgck/actions-comment-run#tips-saved-replies)

Then, whenever an untrusted PR is submitted to your repo:

1. Review the code carefully to make sure no secrets are being printed
1. Post a comment with your saved reply, which will trigger the Github Actions bot to create a copy of the untrusted PR on the repo under your name: because you're a trusted contributor, secrets will be passed to CI and the build will proceed
1. The outside contributor will be able to see the build and, if necessary, make changes
1. Review the new changes for any secret-leaking
1. Re-post the same saved reply to push their latest code to your copy branch and re-run CI

This allows you to require untrusted code to pass a review before it runs (thereby securing your secrets) while still preserving the safety of CI.
