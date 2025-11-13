# ----------------- Targets ----------------- #
.PHONY: all
all: build push

.PHONY: build
build:
	cd ./service-a && make build
	cd ./service-b && make build

.PHONY: push
push:
	cd ./service-a && make push
	cd ./service-b && make push