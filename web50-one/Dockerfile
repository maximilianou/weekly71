# ./Dockerfile
# Get started with a build env with Rust nightly
FROM rustlang/rust:nightly-bookworm AS development

RUN groupadd -g 1000 appgroup && \
    useradd -m -u 1000 -g appgroup appuser 
RUN mkdir /app && chown appuser:appgroup /app
USER appuser
WORKDIR /app

# Install cargo-binstall, which makes it easier to install other
# cargo extensions like cargo-leptos
RUN wget https://github.com/cargo-bins/cargo-binstall/releases/latest/download/cargo-binstall-x86_64-unknown-linux-musl.tgz
RUN tar -xvf cargo-binstall-x86_64-unknown-linux-musl.tgz
RUN cp cargo-binstall /usr/local/cargo/bin

# Install cargo-leptos
RUN cargo binstall cargo-leptos -y

# Add the WASM target
RUN rustup target add wasm32-unknown-unknown

# Make an /app dir, which everything will eventually live in
RUN mkdir -p /app
WORKDIR /app
COPY ./app .

# Set any required env variables and
ENV RUST_LOG="debug"
ENV LEPTOS_SITE_ADDR="0.0.0.0:8080"
ENV LEPTOS_SITE_ROOT="site"
EXPOSE 8080
CMD [ "cargo", "leptos", "watch" ]