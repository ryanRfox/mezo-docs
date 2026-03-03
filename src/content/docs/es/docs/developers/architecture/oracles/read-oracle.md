---
title: Leer Datos de Mercado con Oráculos
description: Cómo acceder e interpretar datos de mercado usando la infraestructura de oráculos de Mezo.
topic: developers
---

Mezo proporciona datos de mercado a través de dos sistemas de oráculos: [Skip Connect](https://docs.skip.build/connect/introduction) para BTC/USD y la [Red Pyth](https://docs.pyth.network/home) para feeds de precios adicionales.

## Descripción General

### Oráculo Skip

El oráculo Skip proporciona feeds de precios nativos de BTC/USD en Mezo a través de una interfaz de agregador compatible con Chainlink.

- **Par soportado:** Solo BTC/USD
- **Frecuencia de actualización:** Cada bloque
- **Interfaz:** [Chainlink Aggregator](https://github.com/smartcontractkit/libocr/blob/9e4afd8896f365b964bdf769ca28f373a3fb0300/contract/AccessControlledOffchainAggregator.sol)

### Oráculo Pyth

El oráculo Pyth proporciona múltiples feeds de precios más allá de BTC/USD.

**Frecuencia de actualización:** Depende del par de tokens.

- En Mezo consulta el [archivo de configuración](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler/configmap.yaml) para más detalles.
- En Ethereum consulta el [archivo de configuración](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler-ethereum/configmap.yaml) para más detalles.

El precio se actualiza cuando `price_deviation` se desvía más de un umbral establecido (en puntos porcentuales) o cada `time_difference` segundos, lo que ocurra primero.

## Mejores Prácticas

Al desarrollar dApps que consumen datos de oráculos, sigue estas pautas:

- **Valida la frescura:** Siempre verifica las marcas de tiempo y los números de bloque para detectar datos obsoletos. Establece umbrales de obsolescencia apropiados para tu caso de uso.
- **Establece límites de precio:** Implementa verificaciones de cordura en los valores de precios para detectar anomalías o intentos de manipulación.
- **Monitorea las condiciones del mercado:** Mantente atento a la volatilidad, liquidez y posibles eventos de manipulación que puedan requerir pausar tu aplicación.
- **Auditorías de seguridad:** Asegúrate de que tus contratos y dependencias cumplan con los estándares de seguridad. Audita el código que maneja datos de oráculos para prevenir exploits.

## Leer Feeds de Precios

### Usando el Oráculo Skip (BTC/USD)

El oráculo Skip implementa una interfaz compatible con Chainlink. Llama a `latestRoundData()` para obtener el precio más reciente:

**Contrato:** [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015)

**Valores de retorno:**

- `roundId`: El ID de la ronda cuando se actualizó el precio
- `answer`: El precio de BTC/USD (usa `decimals()` para obtener la precisión decimal)
- `startedAt`: Marca de tiempo Unix cuando comenzó la ronda
- `updatedAt`: Marca de tiempo Unix de la última actualización de la ronda

### Usando el Oráculo Pyth (Múltiples Feeds)

Pyth proporciona múltiples feeds de precios a través de sus contratos EVM. Usa `getPriceNoOlderThan(pair_id, maxAgeSeconds)` para obtener precios con verificaciones de obsolescencia integradas.

**Contrato Mezo Mainnet:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**Contrato Mezo Testnet:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**Contrato Ethereum Mainnet:** [0x4305FB66699C3B2702D4d05CF36551390A4c69C6](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6)

**Contrato Ethereum Sepolia:** [0xDd24F84d36BF92C65F92307595335bdFab5Bbd21](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21)

**Ejemplo para obtener el precio de un price id dado:**

```solidity
import "@pythnetwork/pyth-sdk-solidity/IPyth.sol";
import "@pythnetwork/pyth-sdk-solidity/PythStructs.sol";

contract ReadPythPrice {
    IPyth public immutable pyth;

    constructor(address pythContract) {
        pyth = IPyth(pythContract); // 0x2880aB155794e7179c9eE2e38200202908C17B43 on Mezo
    }

    function getPrice(bytes32 priceId, uint256 maxAgeSeconds)
        external
        view
        returns (int64 price, uint64 conf, int32 expo, uint256 publishTime)
    {
        PythStructs.Price memory priceData = pyth.getPriceNoOlderThan(priceId, maxAgeSeconds);
        return (priceData.price, priceData.conf, priceData.expo, priceData.publishTime);
    }

    // Example: Get MUSD/USD price
    function getMUSDPrice() external view returns (int64, uint256) {
        bytes32 musdPriceId = 0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de;
        PythStructs.Price memory price = pyth.getPriceNoOlderThan(musdPriceId, 3600); // Max 1 hour old
        return (price.price, price.publishTime);
    }
}
```

**Referencia:** Documentación de la API getPriceNoOlderThan [aquí](https://api-reference.pyth.network/price-feeds/evm/getPriceNoOlderThan)

## Datos de Precios Offchain

Puedes consultar feeds de precios y metadatos directamente de la Red Pyth sin interactuar con la blockchain:

- **API Hermes:** [https://hermes.pyth.network/docs/#/rest/price_feeds_metadata](https://hermes.pyth.network/docs/#/rest/price_feeds_metadata)
- **IDs de feeds de precios:** [https://docs.pyth.network/price-feeds/price-feeds#feed-ids](https://docs.pyth.network/price-feeds/price-feeds#feed-ids)

## Feeds de Precios Disponibles

### Feeds del Oráculo Skip

Disponibles tanto en Mezo mainnet como en testnet:

| Par | Dirección del Contrato | Red |
|------|-----------------|---------|
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015) | Mainnet |
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.test.mezo.org/address/0x7b7c000000000000000000000000000000000015) | Testnet |

**API del Nodo (Solo Testnet):** [http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD](http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD)

### Feeds del Oráculo Pyth

Disponibles en:

- Mezo [Mainnet](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) en `0x2880aB155794e7179c9eE2e38200202908C17B43`
- Mezo [Testnet](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) en `0x2880aB155794e7179c9eE2e38200202908C17B43`.
- Ethereum [Mainnet](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6) en `0x4305FB66699C3B2702D4d05CF36551390A4c69C6`
- Ethereum [Sepolia](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21) en `0xDd24F84d36BF92C65F92307595335bdFab5Bbd21`.

**IDs de feeds de precios soportados en Mezo:**

| Par | ID del Feed de Precios |
|------|---------------|
| SolvBTC/USD | `0xf253cf87dc7d5ed5aa14cba5a6e79aee8bcfaef885a0e1b807035a0bbecc36fa` |
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| BTC/USD | `0xe62df6c8b4a85fe1a67db44dc12de5db330f7ac66b72dc658afedf0f4a415b43` |
| cbBTC/USD | `0x2817d7bfe5c64b8ea956e9a26f573ef64e72e4d7891f2d6af9bcc93f7aff9a97` |
| USDC/USD | `0xeaa020c61cc479712813461ce153894a96a6c00b21ed0cfc2798d1f9a9e9c94a` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |
| T/USD | `0x7a072b799215196b0ecb6a58636ec312bce8461dcc33c28c3a046b1e636d121d` |

**IDs de feeds de precios soportados en Ethereum:**

| Par | ID del Feed de Precios |
|------|---------------|
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |

**Solicitar feeds de precios adicionales:** Para habilitar feeds de precios Pyth adicionales, consulta la lista completa de [IDs de feeds de precios](https://docs.pyth.network/price-feeds/price-feeds#feed-ids) disponibles y contacta al equipo de Mezo para solicitar la activación onchain.
