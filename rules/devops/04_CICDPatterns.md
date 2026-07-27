---
trigger: always_on
---

# CI/CD Patterns (DevOps)

## 1. GitLab CI Pipeline

### Standard 3-Stage Pipeline

```yaml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_TLS_CERTDIR: "/certs"
  REGISTRY: registry.unicon.site

# ── Test Stage ──────────────────────────────────
test:
  stage: test
  image: golang:1.25
  script:
    - golangci-lint run
    - go test ./... -v -coverprofile=coverage.out
    - gosec ./...
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.out

# ── Build Stage ─────────────────────────────────
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $REGISTRY
    - docker build -t $REGISTRY/$APP_NAME:$CI_ENVIRONMENT_NAME-$CI_COMMIT_SHORT_SHA .
    - docker push $REGISTRY/$APP_NAME:$CI_ENVIRONMENT_NAME-$CI_COMMIT_SHORT_SHA
  only:
    - develop
    - staging
    - main

# ── Deploy Stage ────────────────────────────────
deploy-dev:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: dev
  script:
    - kubectl config use-context dev
    - cd k8s/environments/dev/$APP_NAME
    - yq e ".spec.template.spec.containers[0].image = \"$REGISTRY/$APP_NAME:dev-$CI_COMMIT_SHORT_SHA\"" -i deployment.yaml
    - kubectl apply -f .
    - kubectl rollout status deployment/$APP_NAME-dev --timeout=120s
  only:
    - develop
  when: auto

deploy-staging:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: staging
  script:
    - kubectl config use-context staging
    - cd k8s/environments/staging/$APP_NAME
    - yq e ".spec.template.spec.containers[0].image = \"$REGISTRY/$APP_NAME:staging-$CI_COMMIT_SHORT_SHA\"" -i deployment.yaml
    - kubectl apply -f .
    - kubectl rollout status deployment/$APP_NAME-staging --timeout=120s
  only:
    - staging
  when: manual

deploy-production:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: production
  script:
    - kubectl config use-context production
    - cd k8s/environments/production/$APP_NAME
    - yq e ".spec.template.spec.containers[0].image = \"$REGISTRY/$APP_NAME:prod-$CI_COMMIT_SHORT_SHA\"" -i deployment.yaml
    - kubectl apply -f .
    - kubectl rollout status deployment/$APP_NAME-prod --timeout=180s
  only:
    - main
  when: manual
```

## 2. Branch Strategy

| Branch | Environment | Deploy |
|---|---|---|
| `develop` | dev | Auto |
| `staging` | staging | Manual |
| `main` | production | Manual |

## 3. Image Tagging

```
{env}-{commitSHA}
{env}-latest
```

Examples:
- `dev-abc1234`
- `staging-def5678`
- `prod-ghi9012`

## 4. Health Check After Deploy

```yaml
# In deploy script
- |
  for i in $(seq 1 10); do
    if curl -sf "https://$APP_URL/health" > /dev/null; then
      echo "Health check passed"
      exit 0
    fi
    echo "Attempt $i/10 failed, waiting 10s..."
    sleep 10
  done
  echo "Health check failed"
  exit 1
```

## 5. Rollback

```bash
# Quick rollback
kubectl rollout undo deployment/$APP_NAME-$ENV

# Rollback to specific revision
kubectl rollout undo deployment/$APP_NAME-$ENV --to-revision=2
```

## 6. Required CI Variables

| Variable | Description |
|---|---|
| `KUBECONFIG` | Base64-encoded kubeconfig |
| `GITLAB_TOKEN` | GitLab API token |
| `CI_REGISTRY_USER` | Registry username |
| `CI_REGISTRY_PASSWORD` | Registry password |
