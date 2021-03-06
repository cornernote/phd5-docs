### GitLab CI

- :blue_book: http://docs.gitlab.com/ce/ci/yaml/README.html


#### Setup

Add variable `REGISTRY_HOST`



### Setup

Set **Variables**

-	`PHP_IMAGE_NAME` registry.example.com/namespace/project_php
-	`GITHUB_API_TOKEN` abcd1234

-----


### Travis

Push example

```
after_success: |
  docker login -u="$REGISTRY_USER" -p="$REGISTRY_PASS" $REGISTRY_HOST
  if [ "${TRAVIS_BRANCH}" = "release" ]; then
    docker tag ${STACK_PHP_IMAGE} ${STACK_PHP_IMAGE_LATEST}
    docker-compose push || exit 1
    docker push ${STACK_PHP_IMAGE_LATEST} || exit 1
  fi
  docker logout
```

---

Example trigger for [`stacks-staging`](https://git.hrzg.de/dangerzone/stacks-staging)

```
deploy:latest:
  stage: deploy
  script:
    - curl -X POST -F token=${PROJECT_TOKEN} -F ref=${PROJECT_REF} -F "variables[REDEPLOY_STACK_DIR]=${STACK_DIR}"  https://git.hrzg.de/api/v3/projects/256/trigger/builds
  only:
    - latest
```