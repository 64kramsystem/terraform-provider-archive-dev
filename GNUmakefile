TEST?=$$(go list ./... |grep -v 'vendor')
GOFMT_FILES?=$$(find . -name '*.go' |grep -v vendor)
WEBSITE_REPO=github.com/hashicorp/terraform-website
PKG_NAME=archive
BINARY_NAME=terraform-provider-archive
VERSION?=6.7.0

GOOS?=$(shell go env GOOS)
GOARCH?=$(shell go env GOARCH)

default: build

build: fmtcheck
	go install

build-local: fmtcheck
	@echo "Building for $(GOOS)/$(GOARCH)..."
	@mkdir -p bin
	GOOS=$(GOOS) GOARCH=$(GOARCH) go build -o bin/$(BINARY_NAME)_$(VERSION)_$(GOOS)_$(GOARCH)

build-all: fmtcheck build-darwin-arm64 build-linux-amd64
	@echo "All builds complete!"
	@ls -lh bin/

build-darwin-arm64: fmtcheck
	@echo "Building for darwin/arm64..."
	@mkdir -p bin
	GOOS=darwin GOARCH=arm64 go build -o bin/$(BINARY_NAME)_$(VERSION)_darwin_arm64

build-linux-amd64: fmtcheck
	@echo "Building for linux/amd64..."
	@mkdir -p bin
	GOOS=linux GOARCH=amd64 go build -o bin/$(BINARY_NAME)_$(VERSION)_linux_amd64

clean:
	@echo "Cleaning build artifacts..."
	@rm -rf bin/
	@echo "Clean complete!"

test: fmtcheck
	go test -i $(TEST) || exit 1
	echo $(TEST) | \
		xargs -t -n4 go test $(TESTARGS) -timeout=30s -parallel=4

testacc: fmtcheck
	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 120m

vet:
	@echo "go vet ."
	@go vet $$(go list ./... | grep -v vendor/) ; if [ $$? -eq 1 ]; then \
		echo ""; \
		echo "Vet found suspicious constructs. Please check the reported constructs"; \
		echo "and fix them if necessary before submitting the code for review."; \
		exit 1; \
	fi

fmt:
	gofmt -w $(GOFMT_FILES)

fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"

errcheck:
	@sh -c "'$(CURDIR)/scripts/errcheck.sh'"


test-compile:
	@if [ "$(TEST)" = "./..." ]; then \
		echo "ERROR: Set TEST to a specific package. For example,"; \
		echo "  make test-compile TEST=./$(PKG_NAME)"; \
		exit 1; \
	fi
	go test -c $(TEST) $(TESTARGS)

website:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

website-test:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider-test PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

.PHONY: build build-local build-all build-darwin-arm64 build-linux-amd64 clean test testacc vet fmt fmtcheck errcheck test-compile website website-test

