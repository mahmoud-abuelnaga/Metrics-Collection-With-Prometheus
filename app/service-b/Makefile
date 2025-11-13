# ----------------- Variables ----------------- #
docker_hub_username ?= abuelnaga0
repository_name ?= metrics-exposing-app-service-b
tag ?= v1.0.0
image_name := $(docker_hub_username)/$(repository_name):$(tag)

# ----------------- Targets ----------------- #
.PHONY: all
all: build push

.PHONY: build
build:
	docker build --tag $(image_name) .

.PHONY: push
push:
	docker push $(image_name)