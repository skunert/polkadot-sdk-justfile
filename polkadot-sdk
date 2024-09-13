time :=`date +"%Y-%m-%dT%H:%M:%S"`
fixed_debug_dir := "~/work/zombienet/current_debug"
default_install_profile := "testnet"

export RUST_BACKTRACE := "1"

default:
  @just --choose

alias i := install-polkadot-parachain
alias it := install-test-parachain
alias is := install-substrate
alias ip := install-polkadot
alias d := generate-docs
alias do := generate-docs-open
alias zsi := zombie-spawn-interactive
alias zti := zombie-test-interactive

[private]
clean-debug:
	mkdir -p {{fixed_debug_dir}};
	rm -rf {{fixed_debug_dir}}/*;

[group("install")]
install PATH FEATURES="":
	cargo install --path {{PATH}} --locked --profile {{default_install_profile}} {{ if FEATURES != "" { "--features " + FEATURES } else { "" } }}
[group("install")]
install-chain-spec-builder: (install "./substrate/bin/utils/chain-spec-builder")
[group("install")]
install-polkadot-parachain FEATURES="runtime-benchmarks": (install "./cumulus/polkadot-parachain" FEATURES)
[group("install")]
install-polkadot FEATURES="runtime-benchmarks": (install "./polkadot" FEATURES)
[group("install")]
install-test-parachain FEATURES="": (install "./cumulus/test/service/" FEATURES)
[group("install")]
install-substrate FEATURES="": (install "./substrate/bin/node/cli" FEATURES)


[group("zombienet")]
zombie-spawn-interactive NAME="": clean-debug
    zombienet -l text --dir {{fixed_debug_dir}} -f --provider native spawn `fd -e toml --exclude "Cargo.toml" | fzf -q "{{NAME}}" -1`
[group("zombienet")]
zombie-test-interactive NAME="": clean-debug
    zombienet -l text --dir {{fixed_debug_dir}} -f --provider native test `fd -e zndsl | fzf -q "{{NAME}}" -1`

[group("zombienet")]
zombie-small-network:
	zombienet -l text --dir {{fixed_debug_dir}} -f --provider native spawn ./cumulus/zombienet/examples/small_network.toml

[group("utility")]
visit-logs:
	lnav {{fixed_debug_dir}}/*.log
[group("utility")]
start-asset-hub-kusama-node:
	polkadot-parachain --chain asset-hub-kusama --tmp --sync warp --relay-chain-rpc-urls wss://kusama-rpc.polkadot.io -- --chain kusama --tmp;
[group("utility")]
taplo:
	taplo format --config .config/taplo.toml
[group("utility")]
clippy $SKIP_WASM_BUILD="1":
	cargo clippy --all-targets --locked --workspace
	cargo clippy --locked --workspace
[group("utility")]
zepter:
	zepter run --config .config/zepter.yaml
[group("utility")]
fmt-check:
	cargo +nightly fmt -- --check
[group("utility")]
fmt:
	cargo +nightly fmt
[group("utility")]
ci: taplo zepter fmt clippy
[group("utility")]
ui-tests:
	RUN_UI_TESTS=1 cargo nextest run --locked -p frame-support-test --features=frame-feature-testing,no-metadata-docs,try-runtime,experimental

[group("docs")]
generate-docs-open $SKIP_WASM_BUILD="1" $RUSTDOCFLAGS="--html-in-header $(pwd)/docs/sdk/assets/header.html --extend-css $(pwd)/docs/sdk/assets/theme.css --default-theme=ayu":
	cargo doc -p polkadot-sdk-docs --open
[group("docs")]
generate-docs $SKIP_WASM_BUILD="1" $RUSTDOCFLAGS="--html-in-header $(pwd)/docs/sdk/assets/header.html --extend-css $(pwd)/docs/sdk/assets/theme.css --default-theme=ayu":
	cargo doc -p polkadot-sdk-docs

[group("test")]
test-crate NAME="" CRATE="" $SKIP_WASM_BUILD="1" :
    cargo nextest run -p `cargo ws list | fzf -q "{{CRATE}}" -1` --nocapture --retries 0 {{NAME}}
[group("test")]
test-crate-wasm NAME="" CRATE="": (test-crate NAME CRATE "0")

[group("build")]
build-crate NAME="" FEATURES="":
    cargo build --release -p `cargo ws list | fzf -q "{{NAME}}" -1` {{ if FEATURES != "" { "--features " + FEATURES } else { "" } }}
[group("build")]
build-crate-bench NAME="": (build-crate NAME "runtime-benchmarks")

[group("build")]
clippy-crate NAME="":
    cargo clippy -p `cargo ws list | fzf -q "{{NAME}}" -1`

[group("benchmark")]
run-benchmark-omni $RUST_LOG="info":
	frame-omni-bencher v1 benchmark pallet --runtime=`fd -I -e compressed.wasm "" target | fzf` --wasm-execution=compiled --pallet=cumulus_pallet_parachain_system --no-storage-info --no-median-slopes --no-min-squares --extrinsic=set_claim_queue_offset --steps=50 --repeat=20 --json
[group("benchmark")]
run-benchmark-overhead-interactive $RUST_LOG="info":
	polkadot-parachain benchmark overhead --chain `fd -I -e json | fzf` --para-id 1000;

run-benchmark $RUST_LOG="info":
	polkadot benchmark pallet --disable-proof-recording --genesis-builder=runtime --runtime=`fd -I -e compressed.wasm "" target | fzf` --wasm-execution=compiled --pallet=frame_system --no-storage-info --no-median-slopes --no-min-squares --extrinsic=cumulus_pallet_parachain_system::set_claim_queue_offset --steps=50 --repeat=20 --json --output=./cumulus/parachains/runtimes/assets/asset-hub-rococo/src/weights/
run-benchmark-overhead $RUST_LOG="info":
	polkadot-parachain benchmark overhead --enable-proof-size --para-id 1000;
overhead $RUST_LOG="info":
    just i runtime-benchmarks;
    polkadot-parachain benchmark overhead --enable-proof-size --para-id 1000;
