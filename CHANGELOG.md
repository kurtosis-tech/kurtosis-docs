(in order of most recent to least recent)

* Update the "Running in CI" instructions to point users to the `mieubrisse/actions-comment-run@allowed-users-for-orgs` fork, which has the necessary fix for running untrusted PRs with org-owned repos
* Replace list of initializer parameters with a pointer to use the `SHOW_HELP` initializer flag
* Update quickstart prerequisites with a Kurtosis login
* Don't run the validation job on `master` branch (should already have had it before PR is merged)
* Added changelog check to CircleCI
* Added instructions for running in CI
* Refactored entire onboarding flow to make it much, much simpler to quickstart!
* Added docs on the `CLIENT_ID` and `CLIENT_SECRET` Docker environment variables that can be passed to Kurtosis
* Updated docs with information about needing to bind-mount `$HOME/.kurtosis` -> `/kurtosis` for the initializer
* Added an initial version of the docs
