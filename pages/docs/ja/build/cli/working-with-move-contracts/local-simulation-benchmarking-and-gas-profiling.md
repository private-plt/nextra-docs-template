import { Callout, FileTree } from "nextra/components";

# ローカルシミュレーション、ベンチマーキング、ガスプロファイリング

## 概要
前のチュートリアルでは、さまざまなCLIコマンドを使用してMoveコントラクトをデプロイおよび相互作用する方法を示しました。

デフォルトでは、これらのコマンドはリモートフルノードへトランザクションを送信し、シミュレーションと実行を行います。この動作をオーバーライドし、ローカルでトランザクションをシミュレートするには、以下のいずれかのコマンドラインオプションを追加します。
- `--local`: 追加の測定や分析を行わずに、トランザクションをローカルでシミュレートします。
- `--benchmark`: トランザクションをベンチマークし、実行時間を報告します。
- `--profile-gas`: 詳細なガス使用量のトランザクションをプロファイルします。

これらの追加オプションは、以下のCLIコマンドと組み合わせて使用​​出来ます。

- `aptos move run`
- `aptos move run-script`
- `aptos move publish`

あるいは、過去のトランザクションの再生への興味がある場合は、[このチュートリアル](../replay-past-transactions.mdx)を御覧下さい。

<Callout type="info" emoji="ℹ️">
Local simulations do not result in any to the on-chain state.
</Callout>

## サンプルコントラクトのデプロイ

デモンストレーションの目的で、引き続き[`hello_blockchain`](https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples/hello_blockchain)パッケージを例として使用します。

まずパッケージをdevネットかテストネットへ公開します(まだ公開していない場合)。

パッケージディレクトリへ変更します。

```bash filename="ターミナル"
cd aptos-move/move-examples/hello_blockchain
```

そして、以下のコマンドを使用してパッケージを公開します。

```bash filename="ターミナル"
aptos move publish --named-addresses hello_blockchain=default --assume-yes
```

<details>
<summary>出力</summary>
```bash
{
  "Result": {
    "transaction_hash": "0xe4ae0ec4ea3474b2123838885b04d7f4b046c174d14d7dc1c56916f2eb553bcf",
    "gas_used": 1118,
    "gas_unit_price": 100,
    "sender": "dbcbe741d003a7369d87ec8717afb5df425977106497052f96f4e236372f7dd5",
    "sequence_number": 5,
    "success": true,
    "timestamp_us": 1713914742422749,
    "version": 1033819503,
    "vm_status": "Executed successfully"
  }
}
```
</details>

注意:CLIプロファイルを適切に設定し、名前付きアドレスを正しくバインドする必要が有ります。詳細は[CLI構成](../setup-cli.mdx)を参照して下さい。

<Callout type="info" emoji="ℹ️">
注意: devネット/テストネットへパッケージを公開する事はローカルシミュレーションのステージのひとつの方法に過ぎず、唯一の方法ではありません。

あるいはローカルノードの使用もしくは最初にコードを公開する必要のないトランザクションのシミュレートが出来ます。（スクリプト、自身のトランザクションのパッケージの公開のような）

</Callout>

## ローカルシミュレーション

次は、追加のコマンドラインオプション`--local`の使用を有効にし、ローカルシミュレーションでエントリー関数message::set_messageを実行します。これは新たな計測や分析の導入をせずローカルでトランザクションを実行します。

```bash filename="ターミナル"
aptos move run --function-id 'default::message::set_message' --args 'string:abc' --local
```

<details>
<summary>出力</summary>
```bash
Simulating transaction locally...
{
  "Result": {
    "transaction_hash": "0x5aab20980688185eed2c9a27bab624c84b8b8117241cd4a367ba2a012069f57b",
    "gas_used": 441,
    "gas_unit_price": 100,
    "sender": "dbcbe741d003a7369d87ec8717afb5df425977106497052f96f4e236372f7dd5",
    "success": true,
    "version": 1033887414,
    "vm_status": "status EXECUTED of type Execution"
  }
}
```
</details>
<Callout type="info" emoji="ℹ️">
ローカル及びリモートシミュレーションで同一の結果が生成されます。
</Callout>

## ベンチマーク
トランザクションの実行時間(秒)の計測では`--benchmark`オプションを使用します。

```bash filename="　ターミナル"
aptos move run --function-id 'default::message::set_message' --args 'string:abc' --benchmark
```

<details>
<summary>出力</summary>
```bash
Benchmarking transaction locally...
Running time (cold code cache): 985.141µs
Running time (warm code cache): 848.159µs
{
  "Result": {
    "transaction_hash": "0xa2fe548d37f12ee79df13e70fdd8212e37074c1b080b89b7d92e82550684ecdb",
    "gas_used": 441,
    "gas_unit_price": 100,
    "sender": "dbcbe741d003a7369d87ec8717afb5df425977106497052f96f4e236372f7dd5",
    "success": true,
    "version": 1033936831,
    "vm_status": "status EXECUTED of type Execution"
  }
}
```
</details>


注意すべきは、これらの実行時間が情報の参考としてのみ提供される事です。それらはローカルマシンの仕様を条件として、ノイズやその他のランダム要素の影響を受けるかもしれません。

**コントラクトの最適化を目指す場合はガスプロファイリング結果を基準として決定すべきです。**

<Callout type="info" emoji="ℹ️">
測定誤差を最小化するため、ベンチマークハーネスは同じトランザクションを複数回実行します。そのため、ベンチマークタスクの完了まで少し時間がかかります。
</Callout>

## ガスプロファイリング

AptosガスプロファイラーはAptosトランザクションのガス使用量の理解で役立つ強力なツールです。一度有効となると、機械化されたVMを使用しトランザクションをシユレーションし、webベースのリポートを生成します。

リポートは完全な実行トレースも含んでいるため、ガスプロファイラーはデバッガーとしても機能します。

### ガスプロファイラーを使用する

`--profile-gas`オプションの添加でガスプロファイラーを呼び出す事が出来ます。

```bash filename="ターミナル"
aptos move run --function-id 'default::message::set_message' --args 'string:abc' --profile-gas
```

<details>
<summary>出力</summary>
```bash
Simulating transaction locally using the gas profiler...
Gas report saved to gas-profiling/txn-d0bc3422-0xdbcb-message-set_message.
{
  "Result": {
    "transaction_hash": "0xd0bc342232f14a6a7d2d45251719aee45373bdb53f68403cfc6dc6062c74fa9e",
    "gas_used": 441,
    "gas_unit_price": 100,
    "sender": "dbcbe741d003a7369d87ec8717afb5df425977106497052f96f4e236372f7dd5",
    "success": true,
    "version": 1034003962,
    "vm_status": "status EXECUTED of type Execution"
  }
}
```
</details>

生成されたガスリポートは、`gas-profiling`ディレクトリ内で見つける事が出来ます。

<FileTree>
  <FileTree.Folder name="hello_blockchain" defaultOpen>
    <FileTree.File name="Move.toml" />
    <FileTree.Folder name="sources" />
    <FileTree.Folder name="gas-profiling" defaultOpen>
      <FileTree.Folder name="txn-XXXXXXXX-0xXXXX-message-set_message" defaultOpen>
        <FileTree.Folder name="assets" defaultOpen />
        <FileTree.File name="index.html" />
      </FileTree.Folder>
    </FileTree.Folder>
  </FileTree.Folder>
</FileTree>

`index.html`はリポートのメインページで、webブラウザで表示出来ます。 
[サンプルリポート](/gas-profiling/sample-report/index.html)


### ガスリポートを理解する

ガスレポートは3つのセクションで構成されていて、異なる視点でガス使用量を理解するのに役立ちます。

#### フレームグラフ

最初のセクションはガス使用量を2つのフレームグラフの形式で可視化したもので、ひとつは実行とIO用、もうひとつはストレージ用です。2つのグラフが必要なのは、これらが異なる単位で計測されるためです。ひとつはガス単位、もうひとつはAPTで計測されます。

グラフの中の種々の要素は相互作用出来ます。アイテムの上へカーソルを置くと正確な費用と割合が表示されます。
![gas-profiling-flamegraph-0.png](/docs/gas-profiling-flamegraph-0.png)

アイテムをクリックすると拡大され、子アイテムを鮮明に表示出来ます。左上隅の「ズームをリセット」ボタンをクリックすると、ビューをリセットできます。
![gas-profiling-flamegraph-1.png](/docs/gas-profiling-flamegraph-1.png)

右上隅には「検索」ボタンもあり、特定の項目を一致させて強調表示する事が出来ます。
![gas-profiling-flamegraph-2.png](/docs/gas-profiling-flamegraph-2.png)

#### 費用の内訳

２つ目のセクションは、全てのガスコストの詳細な内訳です​​。このセクションで表示されるデータは、分類、集計、並べ替えされています。どの数字を見るべきか分かっている場合は、特に役立ちます。

例えば以下の表は、全てのMoveバイトコード命令/操作の実行コストを示しています。ここでのパーセンテージは、所属カテゴリ(この場合 Exec + IO)の合計費用の相対的な物です。

![gas-profiling-cost-break-down-table.png](/docs/gas-profiling-cost-break-down-table.png)

#### 全実行の追跡
<!-- full execution trace  -->
ガスレポートの最後のセクションは、以下のようなトランザクションの完全な実行トレースです。

```text filename="実行トレースの例"
    intrinsic                                                     2.76        85.12%
    dependencies                                                  0.0607      1.87%
        0xdbcb..::message                                         0.0607      1.87%
    0xdbcb..::message::set_message                                0.32416     10.00%
        create_ty                                                 0.0004      0.01%
        create_ty                                                 0.0004      0.01%
        create_ty                                                 0.0004      0.01%
        create_ty                                                 0.0004      0.01%
        create_ty                                                 0.0008      0.02%
        imm_borrow_loc                                            0.00022     0.01%
        call                                                      0.00441     0.14%
        0x1::signer::address_of                                   0.007534    0.23%
            create_ty                                             0.0008      0.02%
            move_loc                                              0.000441    0.01%
            call                                                  0.004043    0.12%
            0x1::signer::borrow_address                           0.000735    0.02%
            read_ref                                              0.001295    0.04%
            ret                                                   0.00022     0.01%
        st_loc                                                    0.000441    0.01%
        copy_loc                                                  0.000854    0.03%
        load<0xdbcb..::0xdbcb..::message::MessageHolder>          0.302385    9.33%
        exists_generic                                            0.000919    0.03%
        not                                                       0.000588    0.02%
        br_false                                                  0.000441    0.01%
        imm_borrow_loc                                            0.00022     0.01%
        move_loc                                                  0.000441    0.01%
        pack                                                      0.000955    0.03%
        move_to_generic                                           0.001838    0.06%
        branch                                                    0.000294    0.01%
        @28
        ret                                                       0.00022     0.01%
    ledger writes                                                 0.097756    3.01%
        transaction
        events
        state write ops                                           0.097756    3.01%
            create<0xdbcb..::0xdbcb..::message::MessageHolder>    0.097756    3.01%
```

左の列には、実行中の全てのMove命令と操作がリストされ、各インデントレベルは関数呼び出しを示します。

中央の列は、操作と関連するガスコストを表します。

バイトコード内の特定の場所へのジャンプを表す特別な表記法`@number`もあります。(上記のスニペット内の`@28`) これは純粋な情報提供であり、制御フローの理解で役立ちます。
