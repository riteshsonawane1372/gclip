# Makefile for gclip
# Author: Generated for cross-platform clipboard manager
# Version: 0.0.1

# Variables
BINARY_NAME=gclip
VERSION=$(shell git describe --tags --always --dirty 2>/dev/null || echo "dev")
BUILD_DATE=$(shell date -u +"%Y-%m-%dT%H:%M:%SZ")
GIT_COMMIT=$(shell git rev-parse --short HEAD 2>/dev/null || echo "unknown")
GO_VERSION=$(shell go version | awk '{print $$3}')

# Build flags
LDFLAGS=-ldflags "-s -w -X main.version=$(VERSION) -X main.buildDate=$(BUILD_DATE) -X main.gitCommit=$(GIT_COMMIT)"
BUILD_FLAGS=-trimpath -mod=readonly

# Directories
BUILD_DIR=build
DIST_DIR=dist
COVERAGE_DIR=coverage

# Go settings
export CGO_ENABLED=0
export GOPROXY=https://proxy.golang.org,direct
export GOSUMDB=sum.golang.org

# Platform targets
PLATFORMS=linux/amd64 linux/arm64 linux/386 darwin/amd64 darwin/arm64 windows/amd64 windows/386 windows/arm64

.DEFAULT_GOAL := help

## help: Show this help message
.PHONY: help
help:
	@echo "gclip $(VERSION) - Production Build System"
	@echo ""
	@echo "TARGETS:"
	@sed -n 's/^##//p' $(MAKEFILE_LIST) | column -t -s ':' | sed -e 's/^/ /'
	@echo ""
	@echo "VARIABLES:"
	@echo "  VERSION     = $(VERSION)"
	@echo "  BUILD_DATE  = $(BUILD_DATE)"
	@echo "  GIT_COMMIT  = $(GIT_COMMIT)"
	@echo "  GO_VERSION  = $(GO_VERSION)"

## init: Initialize project dependencies and tools
.PHONY: init
init:
	@echo "Initializing project..."
	go mod download
	go mod verify
	go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
	go install github.com/goreleaser/goreleaser@latest
	go install honnef.co/go/tools/cmd/staticcheck@latest
	@echo "Project initialized successfully!"

## tidy: Clean up go modules
.PHONY: tidy
tidy:
	@echo "Tidying go modules..."
	go mod tidy
	go mod verify

## fmt: Format Go code
.PHONY: fmt
fmt:
	@echo "Formatting code..."
	go fmt ./...
	gofmt -s -w .

## lint: Run linters
.PHONY: lint
lint:
	@echo "Running linters..."
	golangci-lint run ./...
	staticcheck ./...

## vet: Run go vet
.PHONY: vet
vet:
	@echo "Running go vet..."
	go vet ./...

## test: Run tests with coverage
.PHONY: test
test:
	@echo "Running tests..."
	@mkdir -p $(COVERAGE_DIR)
	go test -v -race -coverprofile=$(COVERAGE_DIR)/coverage.out ./...
	go tool cover -html=$(COVERAGE_DIR)/coverage.out -o $(COVERAGE_DIR)/coverage.html
	go tool cover -func=$(COVERAGE_DIR)/coverage.out | tail -1

## test-short: Run short tests only
.PHONY: test-short
test-short:
	@echo "Running short tests..."
	go test -short -v ./...

## bench: Run benchmarks
.PHONY: bench
bench:
	@echo "Running benchmarks..."
	go test -bench=. -benchmem ./...

## security: Run security checks
.PHONY: security
security:
	@echo "Running security checks..."
	@command -v gosec >/dev/null 2>&1 || go install github.com/securecodewarrior/gosec/v2/cmd/gosec@latest
	gosec ./...

## build: Build binary for current platform
.PHONY: build
build: clean
	@echo "Building $(BINARY_NAME) v$(VERSION)..."
	@mkdir -p $(BUILD_DIR)
	go build $(BUILD_FLAGS) $(LDFLAGS) -o $(BUILD_DIR)/$(BINARY_NAME) .
	@echo "Build complete: $(BUILD_DIR)/$(BINARY_NAME)"

## build-debug: Build with debug symbols
.PHONY: build-debug
build-debug: clean
	@echo "Building debug version..."
	@mkdir -p $(BUILD_DIR)
	go build -gcflags="all=-N -l" -o $(BUILD_DIR)/$(BINARY_NAME)-debug .

## cross-build: Build for all supported platforms
.PHONY: cross-build
cross-build: clean
	@echo "Cross-compiling for all platforms..."
	@mkdir -p $(DIST_DIR)
	@for platform in $(PLATFORMS); do \
		GOOS=$${platform%/*} GOARCH=$${platform#*/} ; \
		echo "Building for $$GOOS/$$GOARCH..." ; \
		output="$(DIST_DIR)/$(BINARY_NAME)-$$GOOS-$$GOARCH" ; \
		if [ "$$GOOS" = "windows" ]; then output="$$output.exe"; fi ; \
		go build $(BUILD_FLAGS) $(LDFLAGS) -o "$$output" . ; \
		if [ $$? -eq 0 ]; then \
			echo "  SUCCESS: $$output" ; \
		else \
			echo "  FAILED: $$output" ; \
		fi ; \
	done
	@echo "Cross-compilation complete!"
	@ls -la $(DIST_DIR)/

## package: Create release packages
.PHONY: package
package: cross-build
	@echo "Creating release packages..."
	@cd $(DIST_DIR) && for file in *; do \
		if [ -f "$$file" ]; then \
			if echo "$$file" | grep -q "windows"; then \
				zip "$${file%.*}.zip" "$$file" ; \
			else \
				tar -czf "$${file}.tar.gz" "$$file" ; \
			fi ; \
		fi ; \
	done
	@echo "Packages created in $(DIST_DIR)/"

## install: Install binary to system PATH
.PHONY: install
install: build
	@echo "Installing $(BINARY_NAME)..."
	@if [ "$(shell uname)" = "Darwin" ] || [ "$(shell uname)" = "Linux" ]; then \
		sudo cp $(BUILD_DIR)/$(BINARY_NAME) /usr/local/bin/ ; \
		sudo chmod +x /usr/local/bin/$(BINARY_NAME) ; \
		echo "Installed to /usr/local/bin/$(BINARY_NAME)" ; \
	else \
		echo "ERROR: Automatic install not supported on this platform" ; \
		echo "       Please manually copy $(BUILD_DIR)/$(BINARY_NAME) to your PATH" ; \
	fi

## uninstall: Remove binary from system PATH
.PHONY: uninstall
uninstall:
	@echo "Uninstalling $(BINARY_NAME)..."
	@if [ -f "/usr/local/bin/$(BINARY_NAME)" ]; then \
		sudo rm /usr/local/bin/$(BINARY_NAME) ; \
		echo "Uninstalled from /usr/local/bin/$(BINARY_NAME)" ; \
	else \
		echo "ERROR: $(BINARY_NAME) not found in /usr/local/bin/" ; \
	fi

## clean: Remove build artifacts
.PHONY: clean
clean:
	@echo "Cleaning build artifacts..."
	rm -rf $(BUILD_DIR) $(DIST_DIR) $(COVERAGE_DIR)
	go clean -cache -testcache -modcache
	@echo "Clean complete!"

## deps: Show dependency tree
.PHONY: deps
deps:
	@echo "Dependency tree:"
	go list -m all

## deps-update: Update all dependencies
.PHONY: deps-update
deps-update:
	@echo "Updating dependencies..."
	go get -u all
	go mod tidy

## deps-graph: Generate dependency graph
.PHONY: deps-graph
deps-graph:
	@echo "Generating dependency graph..."
	@command -v dot >/dev/null 2>&1 || { echo "ERROR: graphviz not installed"; exit 1; }
	go mod graph | modgraphviz | dot -Tsvg -o deps-graph.svg
	@echo "Dependency graph saved to deps-graph.svg"

## docker-build: Build Docker image
.PHONY: docker-build
docker-build:
	@echo "Building Docker image..."
	docker build -t $(BINARY_NAME):$(VERSION) .
	docker tag $(BINARY_NAME):$(VERSION) $(BINARY_NAME):latest

## run: Run the application with arguments
.PHONY: run
run: build
	@echo "Running $(BINARY_NAME)..."
	./$(BUILD_DIR)/$(BINARY_NAME) $(ARGS)

## run-dev: Run with go run for development
.PHONY: run-dev
run-dev:
	@echo "Running in development mode..."
	go run . $(ARGS)

## smoke-test: Run basic smoke tests
.PHONY: smoke-test
smoke-test: build
	@echo "Running smoke tests..."
	@echo "Testing help command..."
	@./$(BUILD_DIR)/$(BINARY_NAME) --help >/dev/null && echo "  PASS: Help command works"
	@echo "Testing version command..."
	@./$(BUILD_DIR)/$(BINARY_NAME) --version >/dev/null && echo "  PASS: Version command works"
	@echo "Testing basic copy/paste..."
	@echo "smoke test" | ./$(BUILD_DIR)/$(BINARY_NAME) --silent
	@./$(BUILD_DIR)/$(BINARY_NAME) --silent | grep -q "smoke test" && echo "  PASS: Copy/paste works"
	@./$(BUILD_DIR)/$(BINARY_NAME) clear --silent
	@echo "Smoke tests passed!"

## ci: Run continuous integration pipeline
.PHONY: ci
ci: fmt vet lint test security build smoke-test
	@echo "CI pipeline completed successfully!"

## release-check: Check if ready for release
.PHONY: release-check
release-check: ci
	@echo "Checking release readiness..."
	@git status --porcelain | grep -q . && { echo "ERROR: Working directory not clean"; exit 1; } || echo "  PASS: Working directory clean"
	@git describe --tags >/dev/null 2>&1 && echo "  PASS: Git tags present" || echo "  WARN: No git tags found"
	@test -f CHANGELOG.md && echo "  PASS: CHANGELOG.md exists" || echo "  WARN: CHANGELOG.md missing"
	@echo "Release check complete!"

## release: Create a new release (requires VERSION tag)
.PHONY: release
release: release-check cross-build package
	@echo "Creating release $(VERSION)..."
	@command -v goreleaser >/dev/null 2>&1 || { echo "ERROR: goreleaser not installed"; exit 1; }
	goreleaser release --clean
	@echo "Release $(VERSION) created!"

## info: Show build information
.PHONY: info
info:
	@echo "Build Information:"
	@echo "  Binary:     $(BINARY_NAME)"
	@echo "  Version:    $(VERSION)"
	@echo "  Build Date: $(BUILD_DATE)"
	@echo "  Git Commit: $(GIT_COMMIT)"
	@echo "  Go Version: $(GO_VERSION)"
	@echo "  Platforms:  $(PLATFORMS)"

# Development shortcuts
.PHONY: dev
dev: fmt vet test build

.PHONY: quick
quick: build smoke-test

# Print build size information
.PHONY: size
size: build
	@echo "Binary size information:"
	@ls -lah $(BUILD_DIR)/$(BINARY_NAME) | awk '{print "  Size: " $$5}'
	@file $(BUILD_DIR)/$(BINARY_NAME)

# Profile build
.PHONY: profile
profile:
	@echo "Building with profiling..."
	go build -gcflags="-m" . 2>&1 | head -20
