---
title: 'Guia do Desenvolvedor: Infraestrutura de Oraculos'
description: >-
  Aprenda como os Oraculos do Mezo fornecem feeds de dados seguros,
  descentralizados para suas aplicacoes.
topic: developers
---

O Mezo inclui um oraculo como parte de seus nos validadores. Oraculos de terceiros tambem estao disponiveis.

## Skip Connect

O Mezo utiliza o [Skip Connect](https://github.com/skip-mev/connect) como seu principal servico de oraculo. O Skip determina o preco de um par de ativos durante o consenso do bloco e o registra no estado on-chain do modulo Cosmos [x/oracle](https://github.com/skip-mev/connect/tree/main/x/oracle). Este modulo e fornecido pelo Skip e esta integrado ao cliente Mezo.

O sidecar e executado no mesmo sistema que o no validador, portanto a recuperacao e agregacao de dados sao concluidas no mesmo sistema e transmitidas ao no validador usando gRPC.

![Um diagrama mostrando o processo onde o Skip agrega dados de mercado de varias fontes e os validadores do Mezo executam o Skip Connect e o x/oracle para atualizar o estado on-chain com os valores mais recentes](/docs/images/oracle/mezo-oracle.avif)

Para uma descricao completa de como o Skip agrega dados, consulte a documentacao do [Skip Providers](https://github.com/skip-mev/connect/blob/main/cmd/constants/providers.go).

O Skip Connect inclui varios [provedores](https://github.com/skip-mev/connect/blob/main/providers/README.md) que podem ser configurados no sidecar. Voce pode encontrar uma lista completa dos provedores disponiveis na documentacao do Skip Connect:

- [Skip Providers (API)](https://github.com/skip-mev/connect/blob/main/providers/apis/README.md)
- [Skip Providers (Websocket)](https://github.com/skip-mev/connect/blob/main/providers/websockets/README.md)
- Referencias de Skip Providers e Market Map:
    - [Provedores](https://github.com/skip-mev/connect/blob/main/cmd/constants/providers.go)
    - [Mercados](https://github.com/skip-mev/connect/blob/main/cmd/constants/marketmaps/markets.go)

## Stork

Stork e um protocolo de oraculo que permite conexoes de latencia ultra baixa entre provedores de dados e aplicacoes on-chain e off-chain. O caso de uso mais comum do Stork e obter e consumir dados de mercado na forma de feeds de precos em tempo real para DeFi. O Stork [esta disponivel na Mezo Testnet](https://docs.stork.network/resources/contract-addresses/evm#mezo).

- **[Documentacao do Stork](https://docs.stork.network/)**
- **[Contratos Implantados na Mezo Testnet](https://docs.stork.network/resources/contract-addresses/evm#mezo)**

## Supra

Supra e uma rede de oraculos cross-chain projetada para alimentar dApps em ecossistemas blockchain com solucoes de dados rapidas, seguras, descentralizadas e escalaveis. O [Distributed Oracle Agreement (DORA)](https://docs.supra.com/oracles/data-feeds#distributed-oracle-agreement-dora) da Supra esta disponivel na Mezo Testnet. Consulte a pagina de [Redes Disponiveis](https://docs.supra.com/oracles/data-feeds/pull-oracle/networks) da Supra para encontrar os enderecos corretos do contrato pull e do contrato de armazenamento.

- **[Documentacao da Supra](https://docs.supra.com/)**

## Pyth

A [Pyth Network](https://pyth.network/) e uma das maiores redes de oraculos de primeira parte e fornece dados em tempo real em varias redes, incluindo o Mezo. O Pyth introduz um inovador design de [oraculo pull de baixa latencia](https://docs.pyth.network/documentation/pythnet-price-feeds/on-demand) onde os usuarios podem obter atualizacoes de precos on-chain quando necessario. Isso permite que todos no ambiente on-chain acessem pontos de dados de forma eficiente. A rede Pyth atualiza os precos a cada 400ms, tornando o Pyth um dos oraculos on-chain mais rapidos.

Contratos de oraculo do Pyth:
- Mezo Mainnet (proxy): [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)
- Mezo Testnet (proxy): [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

Consulte a [Documentacao do Pyth](https://docs.pyth.network/home) para aprender como usar o Pyth em seu dApp.
