# sue445/heroku-cli
Dockerfile for heroku deployment

[![CircleCI](https://circleci.com/gh/sue445/dockerfile-heroku-cli.svg?style=svg)](https://circleci.com/gh/sue445/dockerfile-heroku-cli)

https://hub.docker.com/r/sue445/heroku-cli/

## Build
```bash
docker build --rm -t heroku-cli .
```

## Running
```bash
docker run -it --rm sue445/heroku-cli bash
```

## Example
### CircleCI
c.f. https://circleci.com/docs/2.0/deployment-integrations/#heroku

```yml
# .circleci/config.yml
version: 2

jobs:
  deploy:
    docker:
      - image: sue445/heroku-cli
    working_directory: /home/circleci/app

    steps:
      - run:
          name: Setup Heroku
          command: |-
            cat > ~/.netrc << EOF
            machine api.heroku.com
              login $HEROKU_LOGIN
              password $HEROKU_API_KEY
            EOF

            mkdir -m 700 -p ~/.ssh/
            cat >> ~/.ssh/config << EOF
            StrictHostKeyChecking no
            EOF

      - checkout

      - add_ssh_keys:
          fingerprints:
            - "xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx"

      - run:
          name: Deploy to Heroku
          command: |-
            heroku config:add BUNDLE_WITHOUT="test:development" --app $HEROKU_APP_NAME

            git push git@heroku.com:$HEROKU_APP_NAME.git $CIRCLE_SHA1:refs/heads/master

            heroku run rake db:migrate --app $HEROKU_APP_NAME

workflows:
  version: 2

  build-and-deploy:
    jobs:
      - deploy:
          filters:
            branches:
              only: master
```
