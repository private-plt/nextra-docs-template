---
title: "Using objects"
---

import { Callout } from 'nextra/components'

# オブジェクトの使用

オブジェクトを作成したら、それを Move エントリ関数や構造体で使用したり、転送したり、[オブジェクトの構築中](creating-objects.mdx)生成した参照を使用して変更出来ます。以下は、Moveでオブジェクトを利用、管理、および相互運用する方法です。


## エントリ関数の引数としてオブジェクトを使用する

Move関数内のオブジェクトの型は`Object<T>`で、`T`はオブジェクトが所有するリソースの型です。全てのオブジェクトには、オブジェクトのメタデータを含む`ObjectCore`型があります。

オブジェクトパラメータを使用するには、ユーザーはオブジェクトのアドレスまたはオブジェクトへの参照を渡す事が出来ます。

実行時、コントラクトはそのアドレスにオブジェクトが存在し、関数の実行前にT型のリソースを持っている事を確認します。

```move filename="Example.move"
module my_addr::object_playground {
  use aptos_framework::object::Object;

  struct MyAwesomeStruct has key {}

  /// This will fail if the object doesn't have MyAwesomeStruct stored
  entry fun do_something(object: Object<MyAwesomeStruct>) {
    // ...
  }
  
  /// All Objects have ObjectCore, so this will only fail if the 
  /// address is not an object
  entry fun do_something_to_object_core(object: Object<ObjectCore>) {
    // ...
  }
}
```

エントリ関数のユーザーがリソースの型を指定出来るようにするため、ジェネリック`T`型を保持します。

```move filename="Example.move"
module my_addr::object_playground {
  use aptos_framework::object::Object;

  /// オブジェクトへ凡用`T`が保存されていない場合、これは失敗します。
  entry fun do_something<T>(object: Object<T>) {
    // ...
  }
}
```

### オブジェクトの種類

オブジェクトは、そのオブジェクトが所有する任意のリソースの型で参照出来ます。便宜上、アドレスからオブジェクトへ変換したり、リソースが利用可能である限り、`address_to_object`と`convert<T>`を使用してオブジェクトをタイプ間で変換出来ます。

```move filename="Example.move"
module my_addr::object_playground {
  use aptos_framework::object::{self, Object, ObjectCore};

  struct MyAwesomeStruct has key {}

  fun convert_type(object: Object<ObjectCore>): Object<MyAwesomeStruct> {
    object::convert<MyAwesomeStruct>(object)
  }

  fun address_to_type(object_address: address): Object<MyAwesomeStruct> {
    object::address_to_object<MyAwesomeStruct>(object_address)
  }
}
```

<Callout type="info">
オブジェクトはオブジェクト、アカウント、リソースアカウントを含むあらゆるアドレスで所有出来ます。これにより、オブジェクト間の構成能力と、オブジェクト間の複雑な関連が可能となります。
</Callout>

## 構造体のフィールドの型としてオブジェクトを使用する

オブジェクトは構造体で使用する事で複雑な型を表現するのに役立ちます。例えば

```move filename="Example.move"
module my_addr::object_playground {
  use aptos_framework::object::{Self, Object};
  use aptos_framework::fungible_asset::Metadata;
  use aptos_framework::primary_fungible_store;

  struct MyStruct has key {
    fungible_asset_object: Object<Metadata>
  }

  entry fun create_fungible_asset(creator: &signer) {
    let fa_obj_constructor_ref = &object::create_sticky_object(@my_addr);
    let fa_obj_signer = object::generate_signer(fa_obj_constructor_ref);
    let fa_obj_addr = signer::address_of(&fa_obj_signer);
    primary_fungible_store::create_primary_store_enabled_fungible_asset(
        fa_obj_constructor_ref,
        max_supply,
        name,
        symbol,
        decimals,
        icon_uri,
        project_uri
    );
    move_to(creator, MyStruct {
      fungible_asset_object: object::address_to_object(fa_obj_addr)
    });
  }
}
```

## オブジェクトの所有者を調べる

オブジェクトのコントラクトを作成する時、オブジェクトを変更する前に所有権を確認する事が重要な事がしばしばあります。

どんなアドレスでもオブジェクトを所有出来るため、所有権を確認するには、所有者がひとつのアカウントかどうか、ひとつのリソースアカウントかどうか、または別のオブジェクトかを確認する必要があります。

```move filename="Example.move"
module my_addr::object_playground {
  use std::signer;
  use aptos_framework::object::{self, Object};

  // 承認されていません!
  const E_NOT_AUTHORIZED: u64 = 1;

  fun check_owner_is_caller<T>(caller: &signer, object: Object<T>) {
    assert!(
      object::is_owner(object, signer::address_of(caller)),
      E_NOT_AUTHORIZED
    );
  }

  fun check_is_owner_of_object<T>(addr: address, object: Object<T>) {
    assert!(object::owner(object) == addr, E_NOT_AUTHORIZED);
  }

  fun check_is_nested_owner_of_object<T, U>(
    caller: &signer,
    outside_object: Object<T>,
    inside_object: Object<U>
  ) {
    // 所有権が期待されています
    // 呼び出し元アカウント -> オブジェクトの外部 -> オブジェクトの内部

    // 外部オブジェクトが内部オブジェクトを所有している事を確認する
    let outside_address = object::object_address(outside_object);
    assert!(object::owns(inside_object, outside_address), E_NOT_AUTHORIZED);

    // 呼び出し元が外部オブジェクトを所有している事を確認する
    let caller_address = signer::address_of(caller);
    assert!(object::owns(outside_object, caller_address), E_NOT_AUTHORIZED);

    // 呼び出し元が内部オブジェクトを所有している事を確認する（外部オブジェクト経由）
    // これは最初の2回の呼び出しをスキップ出来ます（ネストされた呼び出しではもっと）
    This can skip the first two calls (and even more nested)
    assert!(object::owns(inside_object, caller_address), E_NOT_AUTHORIZED);
  }
}
```

## 所有権の移転

デフォルトでは、全てのオブジェクトは転送可能です。一部のオブジェクトは、構築時は`ungated_transfer`を無効にするよう構成されています (詳細はオブジェクトの構築を御覧下さい)。

オブジェクトは以下のように転送出来ます:

```move filename="Example.move"
module my_addr::object_playground {
  use aptos_framework::object::{self, Object};

  /// 他のアドレスへ転送します。これはオブジェクトもしくはアカウントとする事が出来ます。
  fun transfer<T>(owner: &signer, object: Object<T>, destination: address) {
    object::transfer(owner, object, destination);
  }

  /// 他のオブジェクトへ転送します
  Transfer to another object
  fun transfer_to_object<T, U>(
    owner: &signer,
    object: Object<T>,
    destination: Object<U>
  ) {
    object::transfer_to_object(owner, object, destination);
  }
}
```

<Callout type="warning">

`ungated_transfer`が**無効**となっている場合、全ての転送では`TransferRef` or `LinearTransferRef`から付与される特別な権限を使用する必要があります。
</Callout>

## イベント

オブジェクトはデフォルトで、オブジェクトが転送されるたびトリガーされる`TransferEvent`を持っているだけです。

オブジェクトを拡張してイベントを追加出来ます。

以下の関数を使用して、オブジェクトのイベントハンドルを作成出来ます。
You can use the following functions to create event handles for Objects:

```move filename="example.move"
module 0x1::object {
  /// オブジェクトのguidを作成します。通常はイベントに使用されます。
  Create a guid for the object, typically used for events
  public fun create_guid(object: &signer): guid::GUID {}
 
  /// 新しいイベントハンドルを生成します。
  public fun new_event_handle<T: drop + store>(object: &signer): event::EventHandle<T> {}
}
```

生成されたイベントハンドルは、オブジェクトの`SignerRef`がある限り、オブジェクトへ転送できます。例えば:

```move filename="example.move"
module 0x42::example {
  use aptos_framework::event;
  use aptos_framework::fungible_asset::FungibleAsset;
  use aptos_framework::object::{Self, Object};
 
  struct LiquidityPoolEventStore has key {
    create_events: event::EventHandle<CreateLiquidtyPoolEvent>,
  }
 
  struct CreateLiquidtyPoolEvent {
    token_a: address,
    token_b: address,
    reserves_a: u128,
    reserves_b: u128,
  }
 
  public entry fun create_liquidity_pool_with_events(
        token_a: Object<FungibleAsset>,
        token_b: Object<FungibleAsset>,
        reserves_a: u128,
        reserves_b: u128
  ) {
    let exchange_signer = &get_exchange_signer();
    let liquidity_pool_constructor_ref = &object::create_object_from_account(
      exchange_signer
    );
    let liquidity_pool_signer = &object::generate_signer(
      liquidity_pool_constructor_ref
    );
    let event_handle = object::new_event_handle<CreateLiquidtyPoolEvent>(
      liquidity_pool_signer
    );

    event::emit_event<CreateLiquidtyPoolEvent>(&mut event_handle, CreateLiquidtyPoolEvent {
      token_a: object::object_address(&token_a),
      token_b: object::object_address(&token_b),
      reserves_a,
      reserves_b,
    });

    move_to(liquidity_pool_signer, LiquidityPool {
      token_a,
      token_b,
      reserves_a,
      reserves_b
    });

    move_to(liquidity_pool_signer, LiquidityPoolEventStore {
      create_events: event_handle
    });
  }
}
```

## 作成後のオブジェクトの変更

通常、オブジェクトは構築中生成された`Refs`でのみ変更出来ます。使用可能な`Refs`、生成方法、使用方法の詳細は[オブジェクトの作成と構成](creating-objects.mdx)を御覧下さい。これは、オブジェクトに追加のリソースを追加したり、オブジェクトを削除したり、オブジェクトを拡張したりする方法です。

唯一の例外は、オブジェクトの「バーン」です。これにより、ウォレットプロバイダーがそのオブジェクトを表示していないことを知らせるフラグが設定されます。これは、オブジェクトが実際は削除できない場合、整理するため使用出来ます。

```move filename="Example.move"
module my_addr::object_playground {
  use aptos_framework::object::{self, Object};

  fun burn_object<T>(owner: &signer, object: Object<T>) {
    object::burn(owner, object);
    assert!(object::is_burnt(object) == true, 1);
  }
}
```

後でオブジェクトを再度表示する必要がある場合は、burn解除出来ます。

```move filename="Example.move"
module my_addr::object_playground {
  use aptos_framework::object::{self, Object};

  fun unburn_object<T>(owner: &signer, object: object::Object<T>) {
    object::unburn(owner, object);
    assert!(object::is_burnt(object) == false, 1);
  }
}
```

## コントラクト例

３つのオブジェクトを使用するリアルワールドコードスニペットはこちら:

- [デジタル資産マーケットプレイスの例](https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples/marketplace)
- [デジタル資産の例](https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples/token_objects)
- [代替資産の例](https://github.com/aptos-labs/aptos-core/tree/main/aptos-move/move-examples/fungible_asset)