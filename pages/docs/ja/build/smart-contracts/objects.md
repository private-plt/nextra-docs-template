---
title: "Building with Objects"
---

# Moveオブジェクト
Moveでは、オブジェクトのリソースをグループ化出来るので、チェーン上で単一のエンティティとして扱う事が出来ます。

オブジェクトには独自のアドレスがあり、アカウントと同様のリソースを所有出来ます。
オブジェクトはエントリ関数で直接使用出来、一度にひとつのリソースではなく完全なパッケージとして転送出来るため、チェーン上でより複雑なデータ型を表すのに役立ちます。

オブジェクトを作成して転送する例はこちら。

```move filename="Example.move"
module my_addr::object_playground {
  use std::signer;
  use std::string::{Self, String};
  use aptos_framework::object::{Self, ObjectCore};
  
  struct MyStruct1 {
    message: String,
  }
  
  struct MyStruct2 {
    message: String,
  }
 
  entry fun create_and_transfer(caller: &signer, destination: address) {
    // Create object
    let caller_address = signer::address_of(caller);
    let constructor_ref = object::create_object(caller_address);
    let object_signer = object::generate_signer(constructor_ref);
    
    // オブジェクト内の２つのリソースを作成してオブジェクトを設定します
    move_to(&object_signer, MyStruct1 {
      message: string::utf8(b"hello")
    });
    move_to(&object_signer, MyStruct2 {
      message: string::utf8(b"world")
    });
 
    // 目的地へ転送します
    let object = object::object_from_constructor_ref<ObjectCore>(
      &constructor_ref
    );
    object::transfer(caller, object, destination);
  }
}
```

構築中に、オブジェクトは転送可能、バーン可能、​​拡張可能となるよう構成出来ます。

例えば、オブジェクトを使用して、オブジェクトを1回だけ転送可能にし、ソウルバウンドNFTを表す事が出来ます。この場合、画像リンクとメタデータ用の独自のリソースを持つ事が出来ます。

オブジェクトは他のオブジェクトを所有する事も出来るため、ソウルバウンドNFTをいくつかオブジェクトへ転送する事で、自前のNFTコレクションオブジェクトを実装出来ます。

## 方法を学ぶ

- [新しいオブジェクトを作成して構成します。](objects/creating-objects.mdx)
- [作成したオブジェクトを使用します。](objects/using-objects.mdx)

## オブジェクトコントラクトの例

- [デジタル資産マーケットプレイスの例](https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples/marketplace)
- [デジタル資産の例](https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples/token_objects)
- [代替資産の例](https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples/fungible_asset)