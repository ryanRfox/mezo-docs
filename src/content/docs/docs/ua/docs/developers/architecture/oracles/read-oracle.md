---
title: Читання ринкових даних за допомогою оракулів
description: >-
  Як отримувати та інтерпретувати ринкові дані за допомогою інфраструктури
  оракулів Mezo.
topic: developers
---

Mezo надає ринкові дані через дві системи оракулів: [Skip Connect](https://docs.skip.build/connect/introduction) для BTC/USD та [Pyth Network](https://docs.pyth.network/home) для додаткових цінових потоків.

## Огляд

### Оракул Skip

Оракул Skip надає нативні цінові потоки BTC/USD на Mezo через інтерфейс агрегатора, сумісний з Chainlink.

- **Підтримувана пара:** лише BTC/USD
- **Частота оновлення:** кожен блок
- **Інтерфейс:** [Chainlink Aggregator](https://github.com/smartcontractkit/libocr/blob/9e4afd8896f365b964bdf769ca28f373a3fb0300/contract/AccessControlledOffchainAggregator.sol)

### Оракул Pyth

Оракул Pyth надає кілька цінових потоків, окрім BTC/USD.

**Частота оновлення:** залежить від пари токенів.

- На Mezo дивіться [файл конфігурації](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler/configmap.yaml) для деталей.
- На Ethereum дивіться [файл конфігурації](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler-ethereum/configmap.yaml) для деталей.

Ціна оновлюється, коли `price_deviation` відхиляється більше ніж на встановлений поріг (у відсоткових пунктах) або кожні `time_difference` секунд, залежно від того, що настане раніше.

## Найкращі практики

При розробці dApp, що використовують дані оракулів, дотримуйтесь цих рекомендацій:

- **Перевіряйте актуальність:** завжди перевіряйте мітки часу та номери блоків для виявлення застарілих даних. Встановіть відповідні порогові значення застарілості для вашого випадку використання.
- **Встановіть межі цін:** реалізуйте перевірки розумності для значень цін, щоб виявити аномалії або спроби маніпуляцій.
- **Моніторте ринкові умови:** слідкуйте за волатильністю, ліквідністю та потенційними подіями маніпуляцій, які можуть вимагати призупинення вашої програми.
- **Аудити безпеки:** переконайтесь, що ваші контракти та залежності відповідають стандартам безпеки. Проведіть аудит коду, який обробляє дані оракулів, для запобігання експлойтам.

## Читання цінових потоків

### Використання оракула Skip (BTC/USD)

Оракул Skip реалізує інтерфейс, сумісний з Chainlink. Викличте `latestRoundData()`, щоб отримати останню ціну:

**Контракт:** [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015)

**Значення, що повертаються:**

- `roundId`: ID раунду, коли ціна була оновлена
- `answer`: Ціна BTC/USD (використовуйте `decimals()` для отримання десяткової точності)
- `startedAt`: Unix мітка часу початку раунду
- `updatedAt`: Unix мітка часу останнього оновлення раунду

### Використання оракула Pyth (кілька потоків)

Pyth надає кілька цінових потоків через свої EVM-контракти. Використовуйте `getPriceNoOlderThan(pair_id, maxAgeSeconds)` для отримання цін з вбудованими перевірками актуальності.

**Контракт Mezo Mainnet:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**Контракт Mezo Testnet:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**Контракт Ethereum Mainnet:** [0x4305FB66699C3B2702D4d05CF36551390A4c69C6](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6)

**Контракт Ethereum Sepolia:** [0xDd24F84d36BF92C65F92307595335bdFab5Bbd21](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21)

**Приклад отримання ціни для заданого ідентифікатора ціни:**

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

**Довідка:** API getPriceNoOlderThan [документація](https://api-reference.pyth.network/price-feeds/evm/getPriceNoOlderThan)

## Офчейн цінові дані

Ви можете запитувати цінові потоки та метадані безпосередньо з мережі Pyth без взаємодії з блокчейном:

- **Hermes API:** [https://hermes.pyth.network/docs/#/rest/price_feeds_metadata](https://hermes.pyth.network/docs/#/rest/price_feeds_metadata)
- **Ідентифікатори цінових потоків:** [https://docs.pyth.network/price-feeds/price-feeds#feed-ids](https://docs.pyth.network/price-feeds/price-feeds#feed-ids)

## Доступні цінові потоки

### Потоки оракула Skip

Доступні на Mezo mainnet та testnet:

| Пара | Адреса контракту | Мережа |
|------|-----------------|---------|
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015) | Mainnet |
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.test.mezo.org/address/0x7b7c000000000000000000000000000000000015) | Testnet |

**Node API (лише Testnet):** [http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD](http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD)

### Потоки оракула Pyth

Доступні на:

- Mezo [Mainnet](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) за адресою `0x2880aB155794e7179c9eE2e38200202908C17B43`
- Mezo [Testnet](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) за адресою `0x2880aB155794e7179c9eE2e38200202908C17B43`.
- Ethereum [Mainnet](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6) за адресою `0x4305FB66699C3B2702D4d05CF36551390A4c69C6`
- Ethereum [Sepolia](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21) за адресою `0xDd24F84d36BF92C65F92307595335bdFab5Bbd21`.

**Підтримувані ідентифікатори цінових потоків Mezo:**

| Пара | Ідентифікатор цінового потоку |
|------|---------------|
| SolvBTC/USD | `0xf253cf87dc7d5ed5aa14cba5a6e79aee8bcfaef885a0e1b807035a0bbecc36fa` |
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| BTC/USD | `0xe62df6c8b4a85fe1a67db44dc12de5db330f7ac66b72dc658afedf0f4a415b43` |
| cbBTC/USD | `0x2817d7bfe5c64b8ea956e9a26f573ef64e72e4d7891f2d6af9bcc93f7aff9a97` |
| USDC/USD | `0xeaa020c61cc479712813461ce153894a96a6c00b21ed0cfc2798d1f9a9e9c94a` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |
| T/USD | `0x7a072b799215196b0ecb6a58636ec312bce8461dcc33c28c3a046b1e636d121d` |

**Підтримувані ідентифікатори цінових потоків Ethereum:**

| Пара | Ідентифікатор цінового потоку |
|------|---------------|
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |

**Запит додаткових цінових потоків:** щоб увімкнути додаткові цінові потоки Pyth, перегляньте повний список доступних [ідентифікаторів цінових потоків](https://docs.pyth.network/price-feeds/price-feeds#feed-ids) та зверніться до команди Mezo для запиту активації в блокчейні.
