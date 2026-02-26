# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`tcplink` is a Go-based TCP networking tool that provides various proxy and relay modes. It supports cleartext and encrypted TCP relays, reverse relays, HTTP/SOCKS proxies, and combinations thereof.

## Build Commands

Build for current platform:
```bash
go build -o tcplink
```

Build for all platforms (Windows, FreeBSD, macOS, Linux):
```bash
bash make.bash
```

On Windows:
```cmd
make.cmd
```

## Running

Run with default config (`config.json` in same directory as executable):
```bash
./tcplink
```

Run with specific config file:
```bash
./tcplink /path/to/config.json
```

## Project Structure

All source files are in the root directory as a single `main` package:
- `main.go` - Entry point, config parsing, mode dispatch
- `link.go` - Core relay logic, encryption/decryption, protocol units
- `relay.go`, `inner.go`, `outer.go` - Relay modes
- `finder.go`, `mapper.go` - Cleartext reverse relay
- `broker.go`, `router.go` - Encrypted reverse relay
- `http.go`, `sock.go` - Cleartext HTTP/SOCKS5 proxy
- `https.go`, `socks.go`, `agent.go` - Encrypted HTTP/SOCKS5 proxy

## Configuration

Configuration is JSON array of rule objects. Each rule has:
- `mode`: one of `relay`, `inner`, `outer`, `finder`, `mapper`, `broker`, `router`, `http`, `sock`, `https`, `socks`, `agent`
- `args`: mode-specific arguments (`listen`, `target`, `secret`)

Example `config.json`:
```json
[
    {"mode": "relay", "args": {"listen": ":4001", "target": "127.0.0.1:14001"}},
    {"mode": "inner", "args": {"secret": "key", "listen": ":4002", "target": "127.0.0.1:14002"}},
    {"mode": "outer", "args": {"secret": "key", "listen": ":14002", "target": "127.0.0.1:24002"}}
]
```

## Architecture

### Mode Pairings
Modes work in pairs for complete data paths:
- `relay` - standalone cleartext relay
- `inner` + `outer` - encrypted relay (inner encrypts, outer decrypts)
- `finder` + `mapper` - cleartext reverse relay (finder connects to mapper)
- `broker` + `router` - encrypted reverse relay (broker connects to router)
- `http`/`sock` - standalone cleartext proxies
- `https`/`socks` + `agent` - encrypted proxies (https/socks connect to agent)

### Encryption Protocol (`link.go`)
Encrypted modes use a custom protocol with:
- AES-CBC encryption with 16-byte secret key
- SHA-256 HMAC authentication
- Protocol unit types: `unitKindData`, `unitKindLink`, `unitKindOK`, `unitKindErr`, `unitKindPing`, `unitKindAuth`
- HTTP-like framing headers (`POST / HTTP/1.1` and `HTTP/1.1 200 OK`)

### Concurrency Model
Each mode spawns a goroutine per accepted connection. The main goroutine reads config and launches mode handlers, then blocks on `select {}`.

## Go Version

Requires Go 1.14 or later (defined in `go.mod`). No external dependencies.
