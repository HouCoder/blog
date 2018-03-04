# Speed up your CI build process

When I was working on a project, we use Travis like this:

```
script:
  - vendor/bin/phpcs -n --standard=PSR2
    ./app/ ./bootstrap/ ./config/ ./resources/ ./tests/
    --extensions=php
    --ignore=cache
    --ignore=./tests/_*
    --ignore=./resources/third-party/*
  - npm run lintspace
  - npm run eslint
  - npm run stylelint
  - ./scripts/webdriverio.sh
```
Travis runs each task in sequence even though each task does not depend on each other, I did some research later and found a way to run independent tasks at the same time.

There is a package called parallel, it can build and execute command lines from a standard input in parallel https://launchpad.net/ubuntu/+source/parallel.

I tried it in that project and it worked great:

```
  - "parallel :::
      \"~/.composer/vendor/bin/phpcs -n --standard=PSR2 ./app/ ./bootstrap/ ./config/ ./resources/ ./tests/ --extensions=php --ignore=cache,./tests/_*,./resources/third-party/*\"
      \"npm run eslint\"
      \"npm run stylelint\"
      \"npm run lintspace\"
      \"./scripts/webdriverio.sh\""
```

I've also added it to another project and did some comparison, here is the result:


|                 | In Sequence  | In Parallel  |
|-----------------|--------------|--------------|
| Time Duration#1 | 2 min 30 sec | 1 min 3 sec  |
| Time Duration#2 | 1 min 47 sec | 1 min 38 sec |
| Time Duration#3 | 1 min 15 sec | 1 min 6 sec  |
| Time Duration#4 | 1 min 32 sec | 1 min 6 sec  |


It's a small time efficiency improvement in a fairly small project, maybe it will gain more performance benefits in a large project.
