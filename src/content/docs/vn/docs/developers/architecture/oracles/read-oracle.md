---
title: Đọc Dữ liệu Thị trường với Oracle
description: Cách truy cập và diễn giải dữ liệu thị trường bằng hạ tầng oracle của Mezo.
topic: developers
---

Mezo cung cấp dữ liệu thị trường thông qua hai hệ thống oracle: [Skip Connect](https://docs.skip.build/connect/introduction) cho BTC/USD và [Pyth Network](https://docs.pyth.network/home) cho các nguồn cấp giá bổ sung.

## Tổng quan

### Oracle Skip

Oracle Skip cung cấp nguồn cấp giá BTC/USD gốc trên Mezo thông qua giao diện aggregator tương thích Chainlink.

- **Cặp được hỗ trợ:** Chỉ BTC/USD
- **Tần suất cập nhật:** Mỗi block
- **Giao diện:** [Chainlink Aggregator](https://github.com/smartcontractkit/libocr/blob/9e4afd8896f365b964bdf769ca28f373a3fb0300/contract/AccessControlledOffchainAggregator.sol)

### Oracle Pyth

Oracle Pyth cung cấp nhiều nguồn cấp giá ngoài BTC/USD.

**Tần suất cập nhật:** Phụ thuộc vào cặp token.

- Trên Mezo xem [tệp cấu hình](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler/configmap.yaml) để biết chi tiết.
- Trên Ethereum xem [tệp cấu hình](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler-ethereum/configmap.yaml) để biết chi tiết.

Giá được cập nhật khi `price_deviation` vượt quá ngưỡng đã đặt (tính bằng điểm phần trăm) hoặc mỗi `time_difference` giây, tùy điều kiện nào đến trước.

## Các Phương pháp Tốt nhất

Khi xây dựng dApp sử dụng dữ liệu oracle, hãy tuân thủ các hướng dẫn sau:

- **Xác minh tính mới:** Luôn kiểm tra timestamp và số block để phát hiện dữ liệu cũ. Đặt ngưỡng thời gian hết hạn phù hợp cho trường hợp sử dụng của bạn.
- **Đặt giới hạn giá:** Triển khai kiểm tra tính hợp lý trên giá trị giá để phát hiện bất thường hoặc nỗ lực thao túng.
- **Giám sát điều kiện thị trường:** Chú ý đến biến động, thanh khoản và các sự kiện thao túng tiềm năng có thể yêu cầu tạm dừng ứng dụng của bạn.
- **Kiểm toán bảo mật:** Đảm bảo hợp đồng và các phụ thuộc đáp ứng tiêu chuẩn bảo mật. Kiểm toán mã xử lý dữ liệu oracle để ngăn chặn khai thác.

## Đọc Nguồn cấp Giá

### Sử dụng Oracle Skip (BTC/USD)

Oracle Skip triển khai giao diện tương thích Chainlink. Gọi `latestRoundData()` để lấy giá mới nhất:

**Hợp đồng:** [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015)

**Giá trị trả về:**

- `roundId`: ID vòng khi giá được cập nhật
- `answer`: Giá BTC/USD (sử dụng `decimals()` để lấy độ chính xác thập phân)
- `startedAt`: Timestamp Unix khi vòng bắt đầu
- `updatedAt`: Timestamp Unix khi vòng được cập nhật lần cuối

### Sử dụng Oracle Pyth (Nhiều Nguồn cấp)

Pyth cung cấp nhiều nguồn cấp giá thông qua hợp đồng EVM. Sử dụng `getPriceNoOlderThan(pair_id, maxAgeSeconds)` để lấy giá với kiểm tra thời gian hết hạn tích hợp.

**Hợp đồng Mezo Mainnet:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**Hợp đồng Mezo Testnet:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**Hợp đồng Ethereum Mainnet:** [0x4305FB66699C3B2702D4d05CF36551390A4c69C6](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6)

**Hợp đồng Ethereum Sepolia:** [0xDd24F84d36BF92C65F92307595335bdFab5Bbd21](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21)

**Ví dụ lấy giá cho một price ID:**

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

**Tham khảo:** API getPriceNoOlderThan [tài liệu](https://api-reference.pyth.network/price-feeds/evm/getPriceNoOlderThan)

## Dữ liệu Giá Offchain

Bạn có thể truy vấn nguồn cấp giá và metadata trực tiếp từ Pyth Network mà không cần tương tác với blockchain:

- **Hermes API:** [https://hermes.pyth.network/docs/#/rest/price_feeds_metadata](https://hermes.pyth.network/docs/#/rest/price_feeds_metadata)
- **ID nguồn cấp giá:** [https://docs.pyth.network/price-feeds/price-feeds#feed-ids](https://docs.pyth.network/price-feeds/price-feeds#feed-ids)

## Nguồn cấp Giá Có sẵn

### Nguồn cấp Oracle Skip

Có sẵn trên cả Mezo mainnet và testnet:

| Cặp | Địa chỉ Hợp đồng | Mạng |
|------|-----------------|---------|
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015) | Mainnet |
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.test.mezo.org/address/0x7b7c000000000000000000000000000000000015) | Testnet |

**API Node (Chỉ Testnet):** [http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD](http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD)

### Nguồn cấp Oracle Pyth

Có sẵn trên:

- Mezo [Mainnet](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) tại `0x2880aB155794e7179c9eE2e38200202908C17B43`
- Mezo [Testnet](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) tại `0x2880aB155794e7179c9eE2e38200202908C17B43`
- Ethereum [Mainnet](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6) tại `0x4305FB66699C3B2702D4d05CF36551390A4c69C6`
- Ethereum [Sepolia](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21) tại `0xDd24F84d36BF92C65F92307595335bdFab5Bbd21`

**ID nguồn cấp giá được hỗ trợ trên Mezo:**

| Cặp | ID Nguồn cấp Giá |
|------|---------------|
| SolvBTC/USD | `0xf253cf87dc7d5ed5aa14cba5a6e79aee8bcfaef885a0e1b807035a0bbecc36fa` |
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| BTC/USD | `0xe62df6c8b4a85fe1a67db44dc12de5db330f7ac66b72dc658afedf0f4a415b43` |
| cbBTC/USD | `0x2817d7bfe5c64b8ea956e9a26f573ef64e72e4d7891f2d6af9bcc93f7aff9a97` |
| USDC/USD | `0xeaa020c61cc479712813461ce153894a96a6c00b21ed0cfc2798d1f9a9e9c94a` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |
| T/USD | `0x7a072b799215196b0ecb6a58636ec312bce8461dcc33c28c3a046b1e636d121d` |

**ID nguồn cấp giá được hỗ trợ trên Ethereum:**

| Cặp | ID Nguồn cấp Giá |
|------|---------------|
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |

**Yêu cầu thêm nguồn cấp giá:** Để kích hoạt thêm nguồn cấp giá Pyth, hãy duyệt danh sách đầy đủ các [ID nguồn cấp giá](https://docs.pyth.network/price-feeds/price-feeds#feed-ids) có sẵn và liên hệ với nhóm Mezo để yêu cầu kích hoạt onchain.
