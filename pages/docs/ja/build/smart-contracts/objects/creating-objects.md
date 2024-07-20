---
title: "Creating objects"
---
import { Callout } from 'nextra/components'


# オブジェクトの作成と構成

オブジェクトの作成では2つの手順が必要です。

1. `ObjectCore`リソースグループを作成します(後でオブジェクトを参照するため使用出来るアドレスを持ちます)。
2. `Ref`と呼ばれる権限を使用して、オブジェクトの動作をカスタマイズします

<Callout type="info">

  `Ref`を生成する事でオブジェクトを構成します。 `Ref`はオブジェクトを作成するのと同じトランザクションで生じる必要があります。後でこれらの設定を変更する事は出来ません。
</Callout>

### オブジェクトの作成

作成できるオブジェクトの種類は3つ有ります。

1. **通常のオブジェクト。** このタイプは削除可能で、ランダムなアドレスを持ちます。 `0x1::object::create_object(owner_address: address)`を使用して作成出来ます。
例:

```move filename="object_playground.move"
module my_addr::object_playground {
  use std::signer;
  use aptos_framework::object;

  entry fun create_my_object(caller: &signer) {
    let caller_address = signer::address_of(caller);
    let constructor_ref = object::create_object(caller_address);
    // ...
  }
}
```

2. **名前付きオブジェクト** この型は削除**出来ず**、決定論的なアドレスを持ちます。`0x1::object::create_named_object(creator: &signer, seed: vector<u8>)`の方法で作成できます。例:
```move filename="object_playground.move"
module my_addr::object_playground {
  use std::signer;
  use aptos_framework::object;

  /// 名前付きオブジェクトのシードは、作成したアカウントがグローバルで一意である必要が有ります。
  const NAME: vector<u8> = b"MyAwesomeObject";

  entry fun create_my_object(caller: &signer) {
    let caller_address = signer::address_of(caller);
    let constructor_ref = object::create_named_object(caller, NAME);
    // ...
  }

  #[view]
  fun has_object(creator: address): bool {
    let object_address = object::create_object_address(&creator, NAME);
    object_exists<0x1::object::ObjectCore>(object_address)
  }
}
```

3. **スティッキーオブジェクト。** 。このタイプも削除**出来ず**ランダムなアドレスを持ちます。`0x1::object::create_sticky_object(owner_address: address)`を使用して作成出来ます。例:

```move filename="object_playground.move"
module my_addr::object_playground {
  use std::signer;
  use aptos_framework::object;

  entry fun create_my_object(caller: &signer) {
    let caller_address = signer::address_of(caller);
    let constructor_ref = object::create_sticky_object(caller_address);
    // ...
  }
}
```

# オブジェクト機能のカスタマイズ

オブジェクトを作成すると、追加の`Ref`の生成で使用出来る`ConstructorRef`を受け取ります。`Ref`は将来、リソースの転送、オブジェクト自体の転送、オブジェクトの削除等のオブジェクト機能を可能/不可能/実行するため使用できます。

以下のセクションでは、よく使用される`Ref`と、`Ref`が可能な機能の解説をします。

<Callout type="info">

  注: `ConstructorRef`は保存出来ません。オブジェクトの作成で使用されたトランザクションの終了時、破棄されるためその他の`Ref`はオブジェクトの作成中生成する**必要があります**。
</Callout>

### リソースの追加

`ConstructorRef`と`object::generate_signer`を使用して、オブジェクトへリソースを転送出来る署名者を作成できます。これは、アカウントへリソースを追加する場合と同じ`move_to`関数を使用します。

```move filename="Example.move"
module my_addr::object_playground {
  use std::signer;
  use aptos_framework::object;

  #[resource_group_member(group = aptos_framework::object::ObjectGroup)]
  struct MyStruct has key {
    num: u8
  }

  entry fun create_my_object(caller: &signer) {
    let caller_address = signer::address_of(caller);

    // オブジェクトを作成します
    let constructor_ref = object::create_object(caller_address);

    // オブジェクトの署名者を取得します
    let object_signer = object::generate_signer(&constructor_ref);

    // マイストラクトリソースをオブジェクトへ移動します
     the MyStruct resource into the object
    move_to(&object_signer, MyStruct { num: 0 });

    // ...
  }
}
```

### 拡張性の追加  (`ExtendRef`)

オブジェクトを後で編集可能としたい場合があります。その場合は、`object::generate_extend_ref`を使用し`ExtendRef`を生成出来ます。
この ref を使用して、オブジェクトの署名者を生成出来ます。

以下の例のように、スマートコントラクトロジックを介し、誰が`ExtendRef`を使用する許可を持つのか、制御出来ます。

```move filename="Example.move"
module my_addr::object_playground {
  use std::signer;
  use std::string::{self, String};
  use aptos_framework::object::{self, Object};

  /// 呼び出し元はこのオブジェクトの所有者では有りません。
  const E_NOT_OWNER: u64 = 1;
  /// 呼び出し元はこのコントラクトの発行者では有りません
  const E_NOT_PUBLISHER: u64 = 2;

  #[resource_group_member(group = aptos_framework::object::ObjectGroup)]
  struct MyStruct has key {
    num: u8
  }

  #[resource_group_member(group = aptos_framework::object::ObjectGroup)]
  struct Message has key {
    message: string::String
  }

  #[resource_group_member(group = aptos_framework::object::ObjectGroup)]
  struct ObjectController has key {
    extend_ref: object::ExtendRef,
  }

  entry fun create_my_object(caller: &signer) {
    let caller_address = signer::address_of(caller);

    // オブジェクトを作成します
    let constructor_ref = object::create_object(caller_address);

    // オブジェクトの署名者を取得します
    let object_signer = object::generate_signer(&constructor_ref);

    // マイストラクトリソースをオブジェクトへ移動します
    move_to(&object_signer, MyStruct { num: 0 });

    // 拡張refを作成し、それをオブジェクトへ移動します
    let extend_ref = object::generate_extend_ref(&constructor_ref);
    move_to(&object_signer, ObjectController { extend_ref });
    // ...
  }

  entry fun add_message(
    caller: &signer,
    object: Object<MyStruct>,
    message: String
  ) acquires ObjectController {
    let caller_address = signer::address_of(caller);
    // 許可の方法は２つ有ります

    // オブジェクトの所有者のみ許可します
    assert!(object::is_owner(object, caller_address), E_NOT_OWNER);
    // コントラクトの発行者のみ許可します
    assert!(caller_address == @my_addr, E_NOT_PUBLISHER);
    // その他の考えうる許可スキームの可能性を考えたらキリが有りません

    // 拡張refを使い、署名者を取得します
    let object_address = object::object_address(object);
    let extend_ref = borrow_global<ObjectController>(object_address).extend_ref;
    let object_signer = object::generate_signer_for_extending(&extend_ref);

    // オブジェクトを拡張し、メッセージを取得します
    move_to(object_signer, Message { message });
  }
}
```

### 転送の無効化/切り替え
無効化/転送の切り替え (`TransferRef`)

デフォルトでは全てのオブジェクト転送可能です。これは`object::generate_transfer_ref`で生成可能な`TransferRef`を介して変更可能です。

以下の例は、オブジェクトが転送可能かどうか測定する許可の生成及び管理の方法を示します。

```move filename="Example.move"
module my_addr::object_playground {
  use std::signer;
  use std::string::{self, String};
  use aptos_framework::object::{self, Object};

  /// 呼び出し元はこのコントラクトの発行者では有りません
  const E_NOT_PUBLISHER: u64 = 1;

  #[resource_group_member(group = aptos_framework::object::ObjectGroup)]
  struct ObjectController has key {
    transfer_ref: object::TransferRef,
  }

  entry fun create_my_object(
    caller: &signer,
    transferrable: bool,
    controllable: bool
  ) {
    let caller_address = signer::address_of(caller);

    // オブジェクトを作成します
    let constructor_ref = object::create_object(caller_address);

    // オブジェクトの署名者を取得します
    let object_signer = object::generate_signer(&constructor_ref);

    // 転送を制御する転送参照を作成します
    let transfer_ref = object::generate_transfer_ref(&constructor_ref);

    //ここでひとつ選択出来ます。オブジェクトを転送出来るようにし、後で変更を許可したい場合は決定出来ます。？　デフォルトでは転送可能です。
    if (!transferrable) {
      object::disable_ungated_transfer(&transfer_ref);
    };

    // 制御可能を望む場合、後で転送参照を保存する必要が有ります。
    if (controllable) {
      move_to(&object_signer, ObjectController { transfer_ref });
    }
    // ...
  }

  /// この例ではコントラクトの発行者のみが転送の許可を変更します。？
  entry fun toggle_transfer(
    caller: &signer,
    object: Object<ObjectController>
  ) acquires ObjectController {
    // 発行者のみが転送を切り替えるようにします
    let caller_address = signer::address_of(caller);
    assert!(caller_address == @my_addr, E_NOT_PUBLISHER);

    // 転送参照を取得します
    let object_address =

 object::object_address(object);
    let transfer_ref = borrow_global<ObjectController>(
      object_address
    ).transfer_ref;

    // 現在の状態を基準としてきり替えます
    if (object::ungated_transfer_allowed(object)) {
      object::disable_ungated_transfer(&transfer_ref);
    } else {
      object::enable_ungated_transfer(&transfer_ref);
    }
  }
}
```

### 1回限りの転送(`LinearTransferRef`)

作成者が全ての転送を制御したい場合、`TransferRef`から`LinearTransferRef`を作成し1回限りの転送機能を提供する事が出来ます。これを使用すると、オブジェクト作成者から受信者へ1回限りの転送を行う事「”ソウルバウンド」オブジェクトを作成出来ます。`LinearTransferRef`はオブジェクトの所有者が使用する必要が有ります。

```move filename="Example.move"
module my_addr::object_playground {
  use std::signer;
  use std::option;
  use std::string::{self, String};
  use aptos_framework::object::{self, Object};

  /// 呼び出し元はこのコントラクトの発行者ではありません
  const E_NOT_PUBLISHER: u64 = 1;

  #[resource_group_member(group = aptos_framework::object::ObjectGroup)]
  struct ObjectController has key {
    transfer_ref: object::TransferRef,
    linear_transfer_ref: option::Option<object::LinearTransferRef>,
  }

  entry fun create_my_object(
    caller: &signer,
    transferrable: bool,
    controllable: bool
  ) {
    let caller_address = signer::address_of(caller);

    // オブジェクトを作成します
    let constructor_ref = object::create_object(caller_address);

    // オブジェクトの署名者を取得します
    let object_signer = object::generate_signer(&constructor_ref);

    // 転送参照を作成し、転送を制御します
    let transfer_ref = object::generate_transfer_ref(&constructor_ref);

    // 非ゲート転送を無効化する
    object::disable_ungated_transfer(&transfer_ref);
    move_to(&object_signer, ObjectController {
      transfer_ref,
      linear_transfer_ref: option::none(),
    });
    // ...
  }

  /// この例ではコントラクトの発行者のみが転送の許可を変更するようにします 
  entry fun allow_single_transfer(
    caller: &signer,
    object: Object<ObjectController>
  ) acquires ObjectController {
    // 発行者のみが転送を切り替えるようにします
    let caller_address = signer::address_of(caller);
    assert!(caller_address == @my_addr, E_NOT_PUBLISHER);

    let object_address = object::object_address(object);

    // 転送参照を取得する
    let transfer_ref = borrow_global<ObjectController>(
      object_address
    ).transfer_ref;

    // 1回限り使用可能な`LinearTransferRef`を生成します
    let linear_transfer_ref = object::generate_linear_transfer_ref(
      &transfer_ref
    );

    //後で使用出来るよう保管します
    let object_controller = borrow_global_mut<ObjectController>(
      object_address
    );
    option::fill(
      &mut object_controller.linear_transfer_ref,
      linear_transfer_ref
    )
  }

  /// 所有者のみが1回転送出来ます
  entry fun transfer(
    caller: &signer,
    object: Object<ObjectController>,
    new_owner: address
  ) acquires ObjectController {
    let object_address = object::object_address(object);

    // linear_転送参照を取得します。これは消費されるのでリソースから抽出する必要が有ります
    let object_controller = borrow_global_mut<ObjectController>(
      object_address
    );
    let linear_transfer_ref = option::extract(
      &mut object_controller.linear_transfer_ref
    );

    object::transfer_with_ref(linear_transfer_ref, new_owner);
  }
}
```

## **オブジェクトの削除を許可する(`DeleteRef`)**

デフォルトの方法(削除を許可する)で作成されたオブジェクトでは`DeleteRef`を生成出来、後で使用出来ます。これは混乱を取り除くだけでなくストレージの払い戻しを受け取る時も役立ちます。

削除出来ないオブジェクトに対し`DeleteRef`を作成する事は出来ません。

```move filename="Example.move"
module my_addr::object_playground {
  use std::signer;
  use std::option;
  use std::string::{self, String};
  use aptos_framework::object::{self, Object};

  /// 呼び出しもとはこのオブジェクトの所有者ではありません
  const E_NOT_OWNER: u64 = 1;

  #[resource_group_member(group = aptos_framework::object::ObjectGroup)]
  struct ObjectController has key {
    delete_ref: object::DeleteRef,
  }

  entry fun create_my_object(
    caller: &signer,
    transferrable: bool,
    controllable: bool
  ) {
    let caller_address = signer::address_of(caller);

    // オブジェクトを作成します
    let constructor_ref = object::create_object(caller_address);

    // オブジェクトの署名者を取得します
    let object_signer = object::generate_signer(&constructor_ref);

    // 削除参照の作成と保存
    let delete_ref = object::generate_delete_ref(&constructor_ref);
    move_to(&object_signer, ObjectController {
      delete_ref
    });
    // ...
  }

  /// 所有者のみがオブジェクトを削除するようにしました
  entry fun delete(
    caller: &signer,
    object: Object<ObjectController>,
  ) {
    // 呼び出し元のみが削除するようにします
    let caller_address = signer::address_of(caller);
    assert!(object::is_owner(&object, caller_address), E_NOT_OWNER);

    let object_address = object::object_address(object);

    // 削除参照を取得します。消費するのでリソースから抽出する必要が有ります。 
    let ObjectController {
      delete_ref
    } = move_from<ObjectController>(
      object_address
    );

    // オブジェクトを永久に削除します
    object::delete(delete_ref);
  }
}
```

## オブジェクトを不変にする

不変と関連するコントラクトの作成によってオブジェクトを不変にし、オブジェクトのいかなる拡張性も変更機能も除去出来ます。デフォルトではコントラクトは不変ではなく、`ExtendRef`で拡張出来、コントラクトが許可すればリソースは変更出来ます。

# 詳細

全ての可能な`Refs`のドキュメントは`0x1::object`[ここ](https://aptos.dev/reference/move?branch=mainnet&page=aptos-framework/doc/object.md)のMoveリファレンスドキュメントで見つける事が出来ます。

構築したオブジェクトの使用方法も[こちらで](using-objects.mdx)調べる事が出来ます。


