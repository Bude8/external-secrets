FROM golang:1.22.4-bookworm@sha256:aec47843e52fee4436bdd3ce931417fa980e9055658b5142140925eea3044bea as builder

ENV KUBECTL_VERSION="v1.28.3"
ENV HELM_VERSION="v3.13.1"

RUN go install github.com/onsi/ginkgo/v2/ginkgo@v2.1.6
RUN wget -q https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl -O /usr/local/bin/kubectl && \
    chmod +x /usr/local/bin/kubectl && \
    wget -q https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz -O - | tar -xzO linux-amd64/helm > /usr/local/bin/helm && \
    chmod +x /usr/local/bin/helm

WORKDIR /usr/src/app
COPY go.mod go.sum ./
RUN go mod download && go mod verify
COPY . .
WORKDIR /usr/src/app/e2e
RUN make e2e-bin

FROM alpine:3.20.0@sha256:77726ef6b57ddf65bb551896826ec38bc3e53f75cdde31354fbffb4f25238ebd
RUN apk add -U --no-cache \
    ca-certificates \
    bash \
    curl \
    tzdata \
    libc6-compat \
    openssl

COPY --from=builder /go/bin/ginkgo /usr/local/bin/
COPY --from=builder /usr/local/bin/kubectl /usr/local/bin/
COPY --from=builder /usr/local/bin/helm /usr/local/bin/

COPY --from=builder /usr/src/app/e2e/entrypoint.sh                   /entrypoint.sh
COPY --from=builder /usr/src/app/e2e/suites/provider/provider.test   /provider.test
COPY --from=builder /usr/src/app/e2e/suites/argocd/argocd.test       /argocd.test
COPY --from=builder /usr/src/app/e2e/suites/flux/flux.test           /flux.test
COPY --from=builder /usr/src/app/e2e/suites/generator/generator.test /generator.test
COPY --from=builder /usr/src/app/e2e/k8s                             /k8s

CMD [ "/entrypoint.sh" ]
