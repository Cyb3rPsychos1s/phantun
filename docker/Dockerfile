#
# 1. 准备阶段 (Planner)：分析项目依赖，生成依赖配方 (recipe.json)
#
FROM lukemathwalker/cargo-chef:latest-rust-latest AS chef
WORKDIR /usr/src/phantun

FROM chef AS planner
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

#
# 2. 构建阶段 (Builder)：先按配方编译依赖（这一层会被缓存），再编译源码
#
FROM chef AS builder
COPY --from=planner /usr/src/phantun/recipe.json recipe.json

# 这一步只编译依赖项！只要依赖没变，Docker 就会直接使用这里的缓存
RUN cargo chef cook --release --recipe-path recipe.json

# 依赖编译完后，再拷贝真正的源码进来
COPY . .

# 编译你自己的业务代码（速度极快）
RUN cargo build --release && \
    strip target/release/server target/release/client && \
    install target/release/server /usr/local/bin/phantun-server && \
    install target/release/client /usr/local/bin/phantun-client

#
# 3. 运行阶段 (Runtime)：精简运行环境，自带 socat
#
FROM debian:latest

RUN apt-get update && apt-get install -y \
    socat \
    procps \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /usr/local/bin/phantun-server /usr/local/bin/
COPY --from=builder /usr/local/bin/phantun-client /usr/local/bin/

ENV RUST_LOG=INFO

ENTRYPOINT ["phantun-server"]