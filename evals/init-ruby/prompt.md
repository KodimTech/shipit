Set up the shipit contract for this repository.

This is a Ruby on Rails project. Its `CLAUDE.md` documents the following developer
commands:

```
bin/setup --skip-server            # install deps and prepare the database
bin/dev                            # start the dev server
bin/ci                             # local CI: setup, rubocop, importmap audit, brakeman
bin/rspec                          # run all specs
bin/rubocop                        # lint
bin/brakeman --quiet --no-pager    # security scan
bin/importmap audit                # audit JS dependencies
```

The CI workflow at `.github/workflows/ci.yml` runs three jobs — `scan_ruby`, `lint`,
and `test` — and the test job runs `bundle exec rspec` followed by
`bundle exec undercover`.

Only some of those binstubs actually exist in `bin/`. Produce the contract.
