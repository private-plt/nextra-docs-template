---
title: "CLI"
---

import { Card, Cards } from '@components/index'

# CLI

Aptosコマンドラインインターフェース(CLI)は、Moveコントラクトのコンパイルとテストに役立つツールです。また、チェーン上でAptos機能をすぐ試す時も役立ちます。

より上級のユーザーは、CLIを使用してプライベートのAptosネットワークを実行する事もできます(ローカルでのコードテストで役立ちます)。ネットワークノードの管理でも役立ちます。


## 📥 Aptos CLIをインストールする

<Cards className="xl:grid-cols-2">
  <Card href="./cli/install-cli/install-cli-mac" className="flex flex-row gap-4">
    <Card.Image className="object-contain dark:invert" src="/docs/apple.svg" />
    <div className="flex flex-col gap-2">
      <Card.Title>Mac</Card.Title>
      <Card.Description>
         Homebrew経由でAptos CLIをインストールする 
         ```sh
         brew install aptos
         ```
      </Card.Description>
    </div>
  </Card>
  <Card href="./cli/install-cli/install-cli-windows" className="flex flex-row gap-4">
    <Card.Image src="/docs/windows.svg" />
    <div className="flex flex-col gap-2">
      <Card.Title>Windows</Card.Title>
      <Card.Description>
          Pythonスクリプトもしくはコンパイル済みバイナリを介してWindowsへAptos CLIをインストールする
      </Card.Description>
    </div>
  </Card>
  <Card href="./cli/install-cli/install-cli-linux" className="flex flex-row gap-4">
    <Card.Image className="object-contain" src="/docs/linux.svg" />
    <div className="flex flex-col gap-2">
      <Card.Title>Linux</Card.Title>
      <Card.Description>
         Pythonスクリプトもしくはコンパイル済みバイナリを介してLinuxへAptos CLIをインストールする
      </Card.Description>
    </div>
  </Card>
  <Card href="./cli/install-cli/install-cli-specific-version" className="flex flex-row gap-4">
    <Card.Image className="dark:invert" src="/docs/advanced.svg" />
    <div className="flex flex-col gap-2">
      <Card.Title>上級 (特定のバージョンをインストールする)</Card.Title>
      <Card.Description>
         ソースからAptos CLIの特定のバージョンをインストールする
      </Card.Description>
    </div>
  </Card>
</Cards>

## ⚙️ Aptos CLIを設定する

<Cards>
  <Card href="./cli/setup-cli">
    <Card.Title>CLIを設定する</Card.Title>
    <Card.Description>Aptos CLIのセットアップと構成 </Card.Description>
  </Card>
  <Card href="./cli/setup-cli/install-move-prover">
    <Card.Title>Move Prover</Card.Title>
    <Card.Description>Move Proverの設定とインストール</Card.Description>
  </Card>
</Cards>

## 🛠️ Aptos CLIの使用

<Cards>
  <Card href="./cli/working-with-move-contracts">
    <Card.Title>Moveコントラクト</Card.Title>
    <Card.Description>Moveコントラクトのコンパイル、公開、シミュレート、ベンチマーク</Card.Description>
  </Card>
  <Card href="./cli/trying-things-on-chain">
    <Card.Title>オンチェーンで試す</Card.Title>
    <Card.Description>Aptosとやり取りし、アカウントを作成し、アカウントをクエリし、Ledger等のハードウェアデバイスを使用します</Card.Description>
  </Card>
  <Card href="./cli/running-a-local-network">
    <Card.Title>ローカルネットワークの実行</Card.Title>
    <Card.Description>ローカルノード/ネットワークを実行する</Card.Description>
  </Card>
</Cards>