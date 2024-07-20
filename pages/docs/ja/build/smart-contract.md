---
title: "Smart Contracts"
---

<!-- import { RemoteCodeblock, permalinkFetch, Card, Cards } from '@components/index'
import { Callout } from 'nextra/components' -->

<!-- export async function getStaticProps() {
  return await permalinkFetch([
    'https://github.com/aptos-labs/aptos-core/blob/77e1d222ebc5e7294e115e0d090c001da1d0e072/aptos-move/move-examples/hello_blockchain/sources/hello_blockchain.move#L1-L59'
  ])
} -->

# Aptosのスマートコントラクト

AptosコントラクトはMoveを使用して記述されます。Moveは安全でサンドボックス化された、形式検証済みのプログラミングのための次世代言語であり、複数のチェーンで使用されています。
Moveを使用すると開発者は、資産への攻撃に対するセキュリティと保護を提供しながら、資産の柔軟な管理、転送をするプログラムを作成出来ます。

## 📖 Moveを学ぶ

<Cards>
  <Card href="smart-contracts/why-move">
    <Card.Title>　なぜMove?</Card.Title>
    <Card.Description>なぜAptosはMove言語を使うのか学ぶ</Card.Description>
  </Card>
  {/* <Card href="smart-contracts/prover">
    <Card.Title>Move Prover</Card.Title>
    <Card.Description>Formal specification and verification of Move contracts on Aptos</Card.Description>
  </Card> */}
  <Card href="smart-contracts/create-package">
    <Card.Title>パッケージを作成</Card.Title>
    <Card.Description>Moveパッケージの作成方法をまず学びましょう</Card.Description>
  </Card>
  <Card href="smart-contracts/objects">
    <Card.Title>オブジェクト</Card.Title>
    <Card.Description>Aptosのオブジェクト標準を使用して、オンチェーンで構成可能で柔軟なプリミティブを作成する方法を学ぶ</Card.Description>
  </Card>
</Cards>

## 👨‍💻 Move例

<Cards>
  <Card href="https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples">
    <Card.Title linkType="external">Aptos Move例</Card.Title>
    <Card.Description>AptosのMoveで開発する30+の例</Card.Description>
  </Card>
  <Card href="https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples/move-tutorial">
    <Card.Title linkType="external">Moveチュートリアル</Card.Title>
    <Card.Description>を使ったプログラミングの基礎をカバーします</Card.Description>
  </Card>
  <Card href="guides/first-move-module">
    <Card.Title>最初のMoveモジュール</Card.Title>
    <Card.Description>最初のmoveモジュールを公開する方法
    の例`hello_blockchain`</Card.Description>
  </Card>
</Cards>

Moveの`hello_blockchain`の例はこちら:

<RemoteCodeblock 
  permalink="https://github.com/aptos-labs/aptos-core/blob/77e1d222ebc5e7294e115e0d090c001da1d0e072/aptos-move/move-examples/hello_blockchain/sources/hello_blockchain.move#L1-L59" 
/>

```move filename="hello_blockchain.move"
module hello_blockchain::message {
    use std::error;
    use std::signer;
    use std::string;
    use aptos_framework::event;

    //:!:>resource
    struct MessageHolder has key {
        message: string::String,
    }
    //<:!:resource

    #[event]
    struct MessageChange has drop, store {
        account: address,
        from_message: string::String,
        to_message: string::String,
    }

    /// There is no message present
    const ENO_MESSAGE: u64 = 0;

    #[view]
    public fun get_message(addr: address): string::String acquires MessageHolder {
        assert!(exists<MessageHolder>(addr), error::not_found(ENO_MESSAGE));
        borrow_global<MessageHolder>(addr).message
    }

    public entry fun set_message(account: signer, message: string::String)
    acquires MessageHolder {
        let account_addr = signer::address_of(&account);
        if (!exists<MessageHolder>(account_addr)) {
            move_to(&account, MessageHolder {
                message,
            })
        } else {
            let old_message_holder = borrow_global_mut<MessageHolder>(account_addr);
            let from_message = old_message_holder.message;
            event::emit(MessageChange {
                account: account_addr,
                from_message,
                to_message: copy message,
            });
            old_message_holder.message = message;
        }
    }

    #[test(account = @0x1)]
    public entry fun sender_can_set_message(account: signer) acquires MessageHolder {
        let addr = signer::address_of(&account);
        aptos_framework::account::create_account_for_test(addr);
        set_message(account, string::utf8(b"Hello, Blockchain"));

        assert!(
            get_message(addr) == string::utf8(b"Hello, Blockchain"),
            ENO_MESSAGE
        );
    }
}
```

## ⚒️ 開発者リソース

### FAQとディスカッション

- MoveのQ&A[Aptos Devディスカッション](https://github.com/aptos-labs/aptos-developer-discussions/discussions)。

### Move IDEプラグイン

- Visual Studio用の[Aptos Move Analyzer](https://marketplace.visualstudio.com/items?itemName=MoveBit.aptos-move-analyzer)。
- [JetBrains IDEs用のMove言語プラグイン](https://plugins.jetbrains.com/plugin/14721-move-language): 構文の強調表示、コード ナビゲーション、名前の変更、書式設定、型チェック、コード生成をサポートします。

### 外部リソース

- [Aptos Moveの例](https://move-developers-dao.gitbook.io/aptos-move-by-example)
- [Teach yourself Move on Aptos](https://github.com/econia-labs/teach-yourself-move).
- [形式検証、Move言語、Moveプルーバー](https://www.certik.com/resources/blog/2wSOZ3mC55AB6CYol6Q2rP-formal-verification-the-move-language-and-the-move-prover)
- [IMCODING Moveチュートリアル](https://www.imcoding.online/tutorials?tag=Aptos)
- [Pontem Moveプレイグラウンド](https://playground.pontem.network/)
- [ネスト可能なMoveリソースのコレクション](https://github.com/taoheorg/taohe)

新しいMoveコンパイラと言語バージョンは現在、初期ベータテスト中です。試してみたい場合は[このページ](https://aptos.dev/en/build/smart-contracts/compiler_v2)を確認して下さい。

