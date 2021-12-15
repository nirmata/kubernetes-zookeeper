VERSION=v3-zk3.6.3
PROJECT_ID=nirmata
REGISTRY=ghcr.io
PROJECT=${REGISTRY}/${PROJECT_ID}

all: build

build:
	docker build -t ${PROJECT}/kubernetes-zookeeper:${VERSION} .

push: build
	docker push ${PROJECT}/kubernetes-zookeeper:${VERSION}

.PHONY: all build push
