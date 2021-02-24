---
title: DigitalBits Software
---
# Software

Pre-built software and services you can run on your own infrastructure, provided by Digitalbits.io.

## [DigitalBits Core](../digitalbits-core/software/admin.html)
DigitalBits Core is the backbone of the DigitalBits network and does the hard work of validating and agreeing on the status of every transaction with other instances of Core through the DigitalBits Consensus Protocol (DCP).

## [Frontier](https://github.com/xdbfoundation/go/tree/master/services/frontier)
Frontier is the client-facing API server for the DigitalBits ecosystem. It acts as the interface between DigitalBits Core and applications that want to access the DigitalBits network. If you are running DigitalBits Core, you will probably also want to run Frontier.

## [Federation Server](https://github.com/xdbfoundation/go/tree/master/services/federation)
A standalone Federation protocol server designed to be dropped in to your existing infrastructure. It can be configured to pull the data it needs out of your existing SQL database.

| Platform       | Latest Release                                                                         |
|----------------|------------------------------------------------------------------------------------------|
| Mac OSX 64-bit (amd64) | [federation-darwin-amd64](https://github.com/xdbfoundation/go/releases/download/federation-v0.2.0/federation-v0.2.0-darwin-amd64.tar.gz)      |
| Linux 64-bit (amd64)  | [federation-linux-amd64](https://github.com/xdbfoundation/go/releases/download/federation-v0.2.0/federation-v0.2.0-linux-amd64.tar.gz)       |
| Linux 32-bit (arm)  | [federation-arm-amd64](https://github.com/xdbfoundation/go/releases/download/federation-v0.2.0/federation-v0.2.0-linux-arm.tar.gz)       |
| Windows 64-bit (amd64) | [federation-windows-amd64.exe](https://github.com/xdbfoundation/go/releases/download/federation-v0.2.0/federation-v0.2.0-windows-amd64.zip) |

## [Bridge Server](https://github.com/xdbfoundation/bridge-server)
DigitalBits’s Bridge server provides a simple interface for the DigitalBits network. It is meant to simplify compliance operations and other more complicated integrations. Because it stores and manages keys and account information, access to it should be well protected. Unlike Frontier, it should never be exposed to the public internet.

## [Archivist](https://github.com/xdbfoundation/go/tree/master/tools/digitalbits-archivist)
This is a small tool, written in Go, for working with digitalbits-core history archives directly. It is a standalone tool that does not require digitalbits-core, or any other programs.


| Platform       | Latest Release                                                                        |
|----------------|------------------------------------------------------------------------------------------|
| Mac OSX 64-bit (amd64) | [digitalbits-archivist-darwin-amd64](https://github.com/xdbfoundation/go/releases/download/digitalbits-archivist-v0.1.0/digitalbits-archivist-v0.1.0-darwin-amd64.tar.gz)      |
| Linux 64-bit (amd64)   | [digitalbits-archivist-linux-amd64](https://github.com/xdbfoundation/go/releases/download/digitalbits-archivist-v0.1.0/digitalbits-archivist-v0.1.0-linux-amd64.tar.gz)       |
| Linux 32-bit (arm)   | [digitalbits-archivist-linux-arm](https://github.com/xdbfoundation/go/releases/download/digitalbits-archivist-v0.1.0/digitalbits-archivist-v0.1.0-linux-arm.tar.gz)       |
| Windows 64-bit (amd64) | [digitalbits-archivist-windows-amd64.exe](https://github.com/xdbfoundation/go/releases/download/digitalbits-archivist-v0.1.0/digitalbits-archivist-v0.1.0-windows-amd64.zip) |
