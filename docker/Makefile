VERSION=v2-zk3.9.1
PROJECT_ID=nirmata
REGISTRY=ghcr.io
PROJECT=${REGISTRY}/${PROJECT_ID}

build-and-push:
	docker buildx create --use
	docker buildx build --platform linux/amd64,linux/arm64 --push -t ${PROJECT}/kubernetes-zookeeper:${VERSION} .

.PHONY: build-and-push
