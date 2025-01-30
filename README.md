


```sh
vi app/src/app.rs
change something
and the change will be reflected into the browser
http://localhost:8080/

```


```sh
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
```


```yml
# docker compose up
services:
  server3:
    build:
      context: .
      target: development
    user: appuser
    ports:
      - 8080:8080
    volumes:
      - ./app:/app      
#    develop:
#      watch:
#        - action: rebuild
#          path: ./app
#          target: /app
```

-------------------

-------------------


```sh
new-app:
	cargo leptos new -g leptos-rs/start-axum -n app 
	curl -o Dockerfile https://raw.githubusercontent.com/maximilianou/weekly70/refs/heads/main/web44-docker/Dockerfile
	curl -o Dockerfile.prod https://raw.githubusercontent.com/maximilianou/weekly70/refs/heads/main/web44-docker/Dockerfile.prod
	curl -o compose.yml https://raw.githubusercontent.com/maximilianou/weekly70/refs/heads/main/web44-docker/compose.yml

dev-app:
	docker compose up --remove-orphans

run-app:
	docker build -f Dockerfile.prod -t rust-app-prod .

del-app:
	rm -r app
	rm Dockerfile Dockerfile.prod compose.yml

clean:
	cd app && cargo clean
	docker container prune
	docker-compose up -d --remove-orphans
	# docker image ls
	# docker image rm rust-app-watch
	
create-bookstore-app db-01:	
	cargo new bookstore	

db-02:	
	cd bookstore && cargo add tokio --features=full
	cd bookstore && cargo add sqlx --features=postgres,runtime-tokio-rustls
	cd bookstore && cargo check

db-03:	
	cd bookstore && cargo run

db-04:
	cd bookstore && mkdir migrations

db-05:
	cargo install sqlx-cli

db-06:
	~/.cargo/bin/sqlx migrate build-script
```


```sh
mkdir web50-island
cd web50-island
make new-app

```

- Cargo.toml
```yaml
[dependencies]
leptos = { version = "0.7.0", features = ["nightly","islands"] }
```



```rust
// lib.rs
pub mod app;

#[cfg(feature = "hydrate")]
#[wasm_bindgen::prelude::wasm_bindgen]
pub fn hydrate() {
    use crate::app::*;
    console_error_panic_hook::set_once();
//    leptos::mount::hydrate_body(App);
    leptos::mount::hydrate_islands();
}
```

```rust
// src/app.rs
                <AutoReload options=options.clone() />
                <HydrationScripts options islands=true />
                <MetaTags/>

```