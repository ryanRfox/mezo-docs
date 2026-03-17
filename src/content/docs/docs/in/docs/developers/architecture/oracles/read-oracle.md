---
title: ऑरेकल के साथ बाजार डेटा पढ़ना
description: >-
  Mezo के ऑरेकल इन्फ्रास्ट्रक्चर का उपयोग करके बाजार डेटा तक पहुंचने और व्याख्या
  करने का तरीका।
topic: developers
---

Mezo दो ऑरेकल सिस्टम के माध्यम से बाजार डेटा प्रदान करता है: BTC/USD के लिए [Skip Connect](https://docs.skip.build/connect/introduction) और अतिरिक्त मूल्य फ़ीड के लिए [Pyth Network](https://docs.pyth.network/home)।

## अवलोकन

### Skip ऑरेकल

Skip ऑरेकल Chainlink-संगत एग्रीगेटर इंटरफेस के माध्यम से Mezo पर नेटिव BTC/USD मूल्य फ़ीड प्रदान करता है।

- **समर्थित जोड़ी:** केवल BTC/USD
- **अपडेट आवृत्ति:** प्रत्येक ब्लॉक
- **इंटरफेस:** [Chainlink Aggregator](https://github.com/smartcontractkit/libocr/blob/9e4afd8896f365b964bdf769ca28f373a3fb0300/contract/AccessControlledOffchainAggregator.sol)

### Pyth ऑरेकल

Pyth ऑरेकल BTC/USD के अलावा कई मूल्य फ़ीड प्रदान करता है।

**अपडेट आवृत्ति:** टोकन जोड़ी पर निर्भर करता है।

- Mezo पर विवरण के लिए [कॉन्फ़िगरेशन फ़ाइल](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler/configmap.yaml) देखें।
- Ethereum पर विवरण के लिए [कॉन्फ़िगरेशन फ़ाइल](https://github.com/mezo-org/mezod/blob/main/infrastructure/kubernetes/common/pyth-scheduler-ethereum/configmap.yaml) देखें।

जब `price_deviation` एक निर्धारित सीमा (प्रतिशत अंकों में) से अधिक विचलित होता है या प्रत्येक `time_difference` सेकंड में, जो भी पहले हो, मूल्य अपडेट किया जाता है।

## सर्वोत्तम प्रथाएं

ऑरेकल डेटा का उपभोग करने वाले dApps बनाते समय, इन दिशानिर्देशों का पालन करें:

- **ताजगी सत्यापित करें:** पुराने डेटा का पता लगाने के लिए हमेशा टाइमस्टैम्प और ब्लॉक नंबर जांचें। अपने उपयोग के मामले के लिए उचित पुरानेपन की सीमाएं निर्धारित करें।
- **मूल्य सीमाएं निर्धारित करें:** विसंगतियों या हेरफेर के प्रयासों का पता लगाने के लिए मूल्य मानों पर विवेक जांच लागू करें।
- **बाजार स्थितियों की निगरानी करें:** अस्थिरता, तरलता और संभावित हेरफेर घटनाओं से अवगत रहें जिनके लिए आपके एप्लिकेशन को रोकने की आवश्यकता हो सकती है।
- **सुरक्षा ऑडिट:** सुनिश्चित करें कि आपके अनुबंध और निर्भरताएं सुरक्षा मानकों को पूरा करती हैं। शोषण को रोकने के लिए ऑरेकल डेटा को संभालने वाले कोड का ऑडिट करें।

## मूल्य फ़ीड पढ़ना

### Skip ऑरेकल का उपयोग (BTC/USD)

Skip ऑरेकल Chainlink-संगत इंटरफेस को लागू करता है। नवीनतम मूल्य प्राप्त करने के लिए `latestRoundData()` कॉल करें:

**अनुबंध:** [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015)

**रिटर्न मान:**

- `roundId`: मूल्य अपडेट होने पर राउंड ID
- `answer`: BTC/USD मूल्य (दशमलव सटीकता प्राप्त करने के लिए `decimals()` का उपयोग करें)
- `startedAt`: राउंड शुरू होने का Unix टाइमस्टैम्प
- `updatedAt`: राउंड के अंतिम अपडेट का Unix टाइमस्टैम्प

### Pyth ऑरेकल का उपयोग (एकाधिक फ़ीड)

Pyth अपने EVM अनुबंधों के माध्यम से कई मूल्य फ़ीड प्रदान करता है। अंतर्निहित पुरानेपन जांच के साथ मूल्य प्राप्त करने के लिए `getPriceNoOlderThan(pair_id, maxAgeSeconds)` का उपयोग करें।

**अनुबंध Mezo मेननेट:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**अनुबंध Mezo टेस्टनेट:** [0x2880aB155794e7179c9eE2e38200202908C17B43](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43)

**अनुबंध Ethereum मेननेट:** [0x4305FB66699C3B2702D4d05CF36551390A4c69C6](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6)

**अनुबंध Ethereum Sepolia:** [0xDd24F84d36BF92C65F92307595335bdFab5Bbd21](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21)

**दिए गए मूल्य ID के लिए मूल्य प्राप्त करने का उदाहरण:**

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

**संदर्भ:** getPriceNoOlderThan API [दस्तावेज़ीकरण](https://api-reference.pyth.network/price-feeds/evm/getPriceNoOlderThan)

## ऑफचेन मूल्य डेटा

आप ब्लॉकचेन के साथ इंटरैक्ट किए बिना सीधे Pyth Network से मूल्य फ़ीड और मेटाडेटा क्वेरी कर सकते हैं:

- **Hermes API:** [https://hermes.pyth.network/docs/#/rest/price_feeds_metadata](https://hermes.pyth.network/docs/#/rest/price_feeds_metadata)
- **मूल्य फ़ीड ID:** [https://docs.pyth.network/price-feeds/price-feeds#feed-ids](https://docs.pyth.network/price-feeds/price-feeds#feed-ids)

## उपलब्ध मूल्य फ़ीड

### Skip ऑरेकल फ़ीड

Mezo मेननेट और टेस्टनेट दोनों पर उपलब्ध:

| जोड़ी | अनुबंध पता | नेटवर्क |
|------|-----------------|---------|
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.mezo.org/address/0x7b7c000000000000000000000000000000000015) | मेननेट |
| BTC/USD | [0x7b7c000000000000000000000000000000000015](https://explorer.test.mezo.org/address/0x7b7c000000000000000000000000000000000015) | टेस्टनेट |

**नोड API (केवल टेस्टनेट):** [http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD](http://mezo-node-0.test.mezo.org:1317/connect/oracle/v2/get_price?currency_pair=BTC/USD)

### Pyth ऑरेकल फ़ीड

उपलब्ध:

- Mezo [मेननेट](https://explorer.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) पर `0x2880aB155794e7179c9eE2e38200202908C17B43`
- Mezo [टेस्टनेट](https://explorer.test.mezo.org/address/0x2880aB155794e7179c9eE2e38200202908C17B43) पर `0x2880aB155794e7179c9eE2e38200202908C17B43`
- Ethereum [मेननेट](https://etherscan.io/address/0x4305FB66699C3B2702D4d05CF36551390A4c69C6) पर `0x4305FB66699C3B2702D4d05CF36551390A4c69C6`
- Ethereum [Sepolia](https://sepolia.etherscan.io/address/0xDd24F84d36BF92C65F92307595335bdFab5Bbd21) पर `0xDd24F84d36BF92C65F92307595335bdFab5Bbd21`

**Mezo समर्थित मूल्य फ़ीड ID:**

| जोड़ी | मूल्य फ़ीड ID |
|------|---------------|
| SolvBTC/USD | `0xf253cf87dc7d5ed5aa14cba5a6e79aee8bcfaef885a0e1b807035a0bbecc36fa` |
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| BTC/USD | `0xe62df6c8b4a85fe1a67db44dc12de5db330f7ac66b72dc658afedf0f4a415b43` |
| cbBTC/USD | `0x2817d7bfe5c64b8ea956e9a26f573ef64e72e4d7891f2d6af9bcc93f7aff9a97` |
| USDC/USD | `0xeaa020c61cc479712813461ce153894a96a6c00b21ed0cfc2798d1f9a9e9c94a` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |
| T/USD | `0x7a072b799215196b0ecb6a58636ec312bce8461dcc33c28c3a046b1e636d121d` |

**Ethereum समर्थित मूल्य फ़ीड ID:**

| जोड़ी | मूल्य फ़ीड ID |
|------|---------------|
| MUSD/USD | `0x0617a9b725011a126a2b9fd53563f4236501f32cf76d877644b943394606c6de` |
| USDT/USD | `0x2b89b9dc8fdf9f34709a5b106b472f0f39bb6ca9ce04b0fd7f2e971688e2e53b` |

**अतिरिक्त मूल्य फ़ीड का अनुरोध:** अतिरिक्त Pyth मूल्य फ़ीड सक्षम करने के लिए, उपलब्ध [मूल्य फ़ीड ID](https://docs.pyth.network/price-feeds/price-feeds#feed-ids) की पूरी सूची ब्राउज़ करें और ऑनचेन सक्रियण का अनुरोध करने के लिए Mezo टीम से संपर्क करें।
