.PHONY: tools gen fmt lint test build clean help

# Change GOBIN to prevent tools from overwriting system wide versions
CACHE := $(PWD)/.cache
export GOBIN := $(CACHE)/bin
PKGS := $(shell go list ./... | grep -v -E '/vendor')

default: help

tools: ## Install tools from tools.go
	@go mod tidy
	@go mod verify
	@cat tools/tools.go | grep _ | awk -F'"' '{print $$2}' | xargs -tI % go install %

gen: ## Run go generate.
	@go generate ./...

fmt: tools ## Format code and docs.
	@go fmt $(PKGS)
	@$(GOBIN)/goimports -w -l .
	@$(GOBIN)/impi --local . --scheme stdThirdPartyLocal $(PKGS)
	@find . -type f -name "*.md" -exec $(GOBIN)/terrafmt fmt -v -f {} \;

lint: tools ## Runs all lint.
	@go vet $(PKGS)
	@$(GOBIN)/golangci-lint run -E unused,gosimple,staticcheck
	@$(GOBIN)/tflint

test: ## Run tests.
	@TF_ACC=1 go test ./... -v -race -count=1 -timeout 120m

build: tools ## Build binary with goreleaser.
	@$(GOBIN)/goreleaser release --skip-publish

clean: ## Clean this directory
	@rm -fr $(CACHE) dist/ || true

help: ## Display this help.
	@awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m<target>\033[0m\n"} /^[a-zA-Z_0-9-]+:.*?##/ { printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) } ' $(MAKEFILE_LIST)
