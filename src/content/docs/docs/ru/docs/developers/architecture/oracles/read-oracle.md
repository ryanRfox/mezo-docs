---
title: Чтение рыночных данных с помощью оракулов
description: >-
  Как получать и интерпретировать рыночные данные с помощью инфраструктуры
  оракулов Mezo.
topic: developers
---

Mezo предоставляет рыночные данные через две системы оракулов: [Skip Connect](https://docs.skip.build/connect/introduction) для BTC/USD и [Pyth Network](https://docs.pyth.network/home) для дополнительных ценовых потоков.

## Обзор

### Оракул Skip

Оракул Skip предоставляет нативные ценовые потоки BTC/USD на Mezo через интерфейс агрегатора, совместимого с Chainlink.

- **Поддерживаемая пара:** только BTC/USD
- **Частота обновления:** каждый блок
- **Интерфейс:** [Chainlink Aggregator](https://github.com/smartcontractkit/libocr/blob/9e4afd8896f365b964bdf769ca28f373a3fb0300/contract/AccessControlledOffchainAggregator.sol)

### Оракул Pyth

Оракул Pyth предоставляет множество ценовых потоков помимо BTC/USD.

**Частота обновления:** зависит от пары токенов.

- На Mezo смотрите [файл конфигурации](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler/configmap.yaml) для подробностей.
- На Ethereum смотрите [файл конфигурации](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler-ethereum/configmap.yaml) для подробностей.

Цена обновляется, когда `price_deviation` отклоняется более чем на установленный порог (в процентных пунктах) или каждые `time_difference` секунд, в зависимости от того, что наступит раньше.

## Лучшие практики

При создании dApps, потребляющих данные оракулов, следуйте этим рекомендациям:

- **Проверяйте актуальность:** всегда проверяйте временные метки и номера блоков для обнаружения устаревших данных. Установите подходящие пороги устаревания для вашего случая использования.
- **Устанавливайте ценовые границы:** реализуйте проверки разумности ценовых значений для обнаружения аномалий или попыток манипулирования.
- **Мониторьте рыночные условия:** учитывайте волатильность, ликвидность и потенциальные события манипулирования, которые могут потребовать приостановки вашего приложения.
- **Аудит безопасности:** убедитесь, что ваши контракты и зависимости соответствуют стандартам безопасности. Проводите аудит кода, обрабатывающего данные оракулов, для предотвращения эксплойтов.

## Чтение ценовых потоков

### Использование оракула Skip (BTC/USD)

Оракул Skip реализует интерфейс, совместимый с Chainlink. Вызовите `latestRoundData()` для получения последней цены:

**Контракт:** [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015)

**Возвращаемые значения:**

- `roundId`: ID раунда, когда цена была обновлена
- `answer`: цена BTC/USD (используйте `decimals()` для получения десятичной точности)
- `startedAt`: временная метка Unix начала раунда
- `updatedAt`: временная метка Unix последнего обновления раунда

### Использование оракула Pyth (множество потоков)

Pyth предоставляет множество ценовых потоков через свои EVM-контракты. Используйте `getPriceNoOlderThan(pair_id, maxAgeSeconds)` для получения цен со встроенными проверками устаревания.

**Контракт Mezo основная сеть:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**Контракт Mezo тестовая сеть:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**Контракт Ethereum основная сеть:** [0x4305FB66699C3B2702D4d05CF36551390A4c69C6](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6)

**Контракт Ethereum Sepolia:** [0xDd24F84d36BF92C65F92307595335bdFab5Bbd21](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21)

**Пример получения цены для заданного ID цены:**

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

**Справка:** getPriceNoOlderThan API [документация](https://api-reference.pyth.network/price-feeds/evm/getPriceNoOlderThan)

## Внецепочечные ценовые данные

Вы можете запрашивать ценовые потоки и метаданные напрямую из Pyth Network без взаимодействия с блокчейном:

- **Hermes API:** [https://hermes.pyth.network/docs/#/rest/price_feeds_metadata](https://hermes.pyth.network/docs/#/rest/price_feeds_metadata)
- **ID ценовых потоков:** [https://docs.pyth.network/price-feeds/price-feeds#feed-ids](https://docs.pyth.network/price-feeds/price-feeds#feed-ids)

## Доступные ценовые потоки

### Потоки оракула Skip

Доступны как в основной, так и в тестовой сети Mezo:

| Пара | Адрес контракта | Сеть |
|------|-----------------|---------|
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015) | Основная сеть |
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.test.mezo.org/address/0x7b7c000000000000000000000000000000000015) | Тестовая сеть |

**API узла (только тестовая сеть):** [http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD](http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD)

### Потоки оракула Pyth

Доступны на:

- Mezo [основная сеть](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) по адресу `0x2880aB155794e7179c9eE2e38200202908C17B43`
- Mezo [тестовая сеть](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) по адресу `0x2880aB155794e7179c9eE2e38200202908C17B43`
- Ethereum [основная сеть](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6) по адресу `0x4305FB66699C3B2702D4d05CF36551390A4c69C6`
- Ethereum [Sepolia](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21) по адресу `0xDd24F84d36BF92C65F92307595335bdFab5Bbd21`

**Поддерживаемые ID ценовых потоков Mezo:**

| Пара | ID ценового потока |
|------|---------------|
| SolvBTC/USD | `0xf253cf87dc7d5ed5aa14cba5a6e79aee8bcfaef885a0e1b807035a0bbecc36fa` |
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| BTC/USD | `0xe62df6c8b4a85fe1a67db44dc12de5db330f7ac66b72dc658afedf0f4a415b43` |
| cbBTC/USD | `0x2817d7bfe5c64b8ea956e9a26f573ef64e72e4d7891f2d6af9bcc93f7aff9a97` |
| USDC/USD | `0xeaa020c61cc479712813461ce153894a96a6c00b21ed0cfc2798d1f9a9e9c94a` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |
| T/USD | `0x7a072b799215196b0ecb6a58636ec312bce8461dcc33c28c3a046b1e636d121d` |

**Поддерживаемые ID ценовых потоков Ethereum:**

| Пара | ID ценового потока |
|------|---------------|
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |

**Запрос дополнительных ценовых потоков:** чтобы включить дополнительные ценовые потоки Pyth, просмотрите полный список доступных [ID ценовых потоков](https://docs.pyth.network/price-feeds/price-feeds#feed-ids) и свяжитесь с командой Mezo для запроса активации в сети.
