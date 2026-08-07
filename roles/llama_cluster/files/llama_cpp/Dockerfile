FROM ubuntu:24.04 AS builder

ARG LLAMA_CPP_REF=ddd4ec1428a6201e18975ea52b07c71e0f9aef26

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates cmake g++ make git curl && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
RUN git -c http.sslVerify=false clone --depth 1 --branch master https://github.com/ggerganov/llama.cpp.git /app/llama.cpp
WORKDIR /app/llama.cpp
RUN git -c http.sslVerify=false fetch --depth 1 origin "$LLAMA_CPP_REF" && git -c http.sslVerify=false checkout "$LLAMA_CPP_REF"

RUN cmake -S . -B build -DGGML_RPC=ON -DCMAKE_INSTALL_PREFIX=/out
RUN cmake --build build --config Release --target install -j"$(nproc)"

FROM ubuntu:24.04

ARG LLAMA_RUNTIME_PACKAGES
ARG LLAMA_ENTRYPOINT

RUN apt-get update && apt-get install -y --no-install-recommends ${LLAMA_RUNTIME_PACKAGES} && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /out /usr/local
RUN ldconfig

ENV LLAMA_ENTRYPOINT=${LLAMA_ENTRYPOINT}
ENTRYPOINT ["/bin/sh", "-c", "exec \"$LLAMA_ENTRYPOINT\" \"$@\"", "--"]
