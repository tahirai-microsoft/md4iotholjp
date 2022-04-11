
# Microsoft Defender for IoT - ハンズオン を始める前に

> 最終更新日: 2022/04/12  
> 対象 : Defender for IoT 10.5.x

ハンズオン実施のための環境をセットアップします

## コンテンツ: 

- [アクション: 環境構築](#アクション-環境構築)
  - [作業 1: リソースグループの作成](#作業-1-リソースグループの作成)
  - [作業 2: 仮想マシン (VM) の作成](#作業-2-仮想マシン-vm-の作成)
  - [作業 3: 仮想マシン に接続](#作業-3-仮想マシン-に接続)
  - [作業 4: Hyper-V を有効化](#作業-4-hyper-v-を有効化)
  - [作業 5: Microsoft Sentinel の初期設定](#作業-5-microsoft-sentinel-の初期設定)

## アクション: 環境構築

このハンズオンでは利用する全てのサービスをホストするため、1つのリソースグループを利用します

### 作業 1: リソースグループの作成

1. Azure ポータル で **リソースの作成** をクリックし、検索ボックスに **Resouce Group** と入力、**作成** をクリックします  
1. サブスクリプションが今回のハンズオンで用いるものであることを確認後、リソースグループ名に **rg-md4iot+SUFFIX** (例: rg-md4iot-tahirai0414) を、リージョンは **(Asia Pacific) Japan East** を入力し、**確認及び作成** をクリックします
1. 確認が完了したら **作成** をクリックします

   > **Note:** リソースグループの名前はサブスクリプション内で一意である必要がありますので、例えばアカウント名に数字などのサフィックスを追加することをお勧めします

   ![RG Create](./images/ActB-T01-rg-create.png 'Create a Resource Group')

### 作業 2: 仮想マシン (VM) の作成

1. Azure Portal の検索ボックスに **Virtual** を入力し、サービスの **Virtual Machine** をクリック、**作成** をクリックします

   ![VM Resource](./images/ActB-T02-resource-create.png 'Create VM resource')

1. **仮想マシンの作成** では、**基本** タブを以下のように入力、設定します  

   | 設定 | 値                                          |
   |-----------------------|----------------------------------|
   | **プロジェクトの詳細** |  |
   | サブスクリプション | ハンズオンで用いるサブスクリプションを選択 |
   | リソースグループ | 直前に作成したリソースグループを選択 |
   | **インスタンスの詳細** |  |
   | 仮想マシン名 | **vm-md4iot-host** を入力|
   | リージョン | **(Asia Pacific) Japan East** または 近いリージョン を選択 |
   | 可用性オプション | **インフラストラクチャ冗長は必要ありません** を選択|
   | セキュリティの種類 | **Standard** を選択|
   | イメージ | **Windows 10 Pro, Version 20H2 - Gen2** を選択|
   | Azure スポット インスタンス | **チェックしない** |
   | サイズ | **Standard_D4s_v3 - 4 vcpu 数, 16 GiB のメモリ** を選択 |
   | **管理者アカウント** | **以下のクレデンシャルを入力** |
   | ユーザー名 | **MDefenderLab** |
   | パスワード | **Learningmode123!** |
   | パスワードの確認 | **Learningmode123!** |
   | **受信ポートの規則** |    |
   | パブリック受信ポート | **選択したポートを許可する** を選択 |
   | 受信ポートを選択 | **RDP (3389)** |
   | **ライセンス** |    |
   | マルチテナントをホストする権利を持つ有効な Windows 10 ライセンスを所有しています | **チェックする** |

   ![VMCreate](./images/ActB-T02-vm-create.png 'Create a new Virtual Machine')

1. サイズ セクションでは **すべてのサイズを表示** をクリックし、表示されたリストから対象のサイズ (D4s_v3) を選択、**選択** をクリックします

   ![VMsize](./images/ActB-T02-vm-size.png 'Select VM size')

1. **管理** タブに移動し、**監視** セクションの **ブート診断** で **無効化** を選択します 

   ![VMsize](./images/ActB-T02-vm-boot.png 'VM Boot Diagnostics')

1. 画面下部の **確認および作成** をクリック、確認を完了後 **作成** をクリックします  
1. (仮想マシンのデプロイには数分かかります) 完了すると画面に表示されます  

   ![VM Deploy](./images/ActB-T02-vm-deployed.png 'VM deployment complete')

### 作業 3: 仮想マシン に接続

1. Azure ポータル から作成した仮想マシンを選択します  

   > **TIP:** 作業3: 6. の画面がまだ残っている場合は、そこから **リソースに移動** をクリックします  

1. 仮想マシンの状態が **実行中** であることを確認します  

   ![VM Running](./images/ActB-T03-vm-running.png 'VM Status Running')

   > **TIP:** 状態が **実行中** ではない場合は、状態更新が完了するまで 1-2分 お待ちください

1. 仮想マシンのメニューから **接続** を選択、続いて **RDP** をクリックします

   ![VM Connect](./images/ActB-T03-vm-connect.png 'Virtual Machine Connection options')

   > **NOTE:** このハンズオンでは RDP を利用して仮想マシンに接続しますが、より安全な方法として Bastion を利用いただくことも可能です (今回は費用等を考慮し RDP としています)  

1. **接続** のページで **RDP ファイルのダウンロード** をクリックし、**ファイルを開く** をクリックします

   ![RDP Download](./images/ActB-T03-download-rdpfile.png 'Download RDP file')

   > **TIP:** この RDP ファイルは仮想マシンのパブリックIPアドレス変更時、再作成が必要になります  

1. RDP のログインスクリーンに (作成時に設定した) 以下ユーザー名とパスワードを入力します  

   | フィールド | 値 |
   |-------|-------|
   | **ユーザー名** | *MDefenderLab* |
   | **パスワード** | *Learningmode123!* |

   ![RDP Login](./images/ActB-T03-rdp-login.png 'RDP Login to VM')

1. **OK** をクリックして仮想マシンにログインします  
   続いて証明書エラーが表示されますが、**はい** をクリックします
   ![RDP Warning](./images/ActB-T03-rdp-warning.png 'RDP certificate warning')

1. 仮想マシンのスクリーンが (別ウィンドウに) 表示されます   
1. **Accept** をクリックします

   ![Default Settings](./images/ActB-T03-vm-connected.png 'Accept privacy settings')

### 作業 4: Hyper-V を有効化

仮想マシン上に更に仮想マシン(a.k.a nested Hyper-V) を作成可能にするため、Hyper-V を有効にします  
先ほど、比較的大きなサイズの仮想マシンを作成した理由がこちらになります  

1. Search for **PowerShell** を検索し、表示されたメニューから **Run as Administrator** をクリックします  

   ![SearchPS](./images/ActB-T04-Start-Powershell.png 'Run PowerShell as Admin')

1. PowerShell ウィンドウに以下のコマンドを入力し、Hyper-V を有効化します  

   ``` powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
   ```

   > **NOTE:** 実行できなかった場合は PowerShell を **管理者** で実行しているか再確認してください

1. Hyper-V の有効化が完了すると再起動を促されますので、**Y** を入力します

   ![PowerShell](./images/ActB-T04-HyperV-Enabled.png 'Enable Hyper-V')

1. 仮想マシン に再接続します  

   > **NOTE:** PowerShell 内で再起動を促されなかった場合には、RDP接続を切断し Azure ポータル の仮想マシンメニューから **仮想マシンを再起動する** を選択、起動を確認後 RDP で再接続してください

1. **Microsoft Edge** を開いて Defender for IoT の ISO ファイル、PCAP ファイルをダウンロードします  

   > **NOTE:** ダウンロード URL は別途ご案内します

<!-- 
https://aka.ms/md4iotiso
https://aka.ms/md4iotpcaps
//-->

1. ダウンロード完了後、RDP 接続を切断します
1. Azure Portal で 仮想マシン を選択し、**停止** をクリックします  

   ![StopVM](./images/ActB-T05-stop-vm.png 'Stop the VM') 

## 作業 5: Microsoft Sentinel の初期設定

   > **NOTE:** この作業は PC から Azure ポータルにアクセスして行います  

1.	検索ボックスに **Sentinel** を入力、 サービスに表示された **Microsoft Sentinel** をクリックします
 
    ![SentinelSearch](./images/ActB-T06-search-sentinel.png 'Sentinel search') 

1.	メニューの **+ 作成** をクリックし、続いて **新しいワークスペースの作成** をクリックします
 
    ![SentinelCreate](./images/ActB-T06-create-sentinel.png 'Sentinel create') 

1. **Log Analytics ワークスペースの作成** において以下の項目を設定、入力します

    - **サブスクリプション**: ハンズオンで用いるサブスクリプションを選択
    - **リソースグループ**: 先ほど作成したリソースグループを選択
    - **Name**: mylogworkspace+SUFFIX (例:mylogworkspace-tahirai)
    - **Regions**: Japan East または 近いリージョン  

    ![SentinelCreateWS](./images/ActB-T06-create-log-analytics-ws.png 'Sentinel create workspace') 

 1. **確認および作成** をクリック、確認を完了後 **作成** をクリックします  
 1. ワークスペースの作成完了後、**追加** をクリックします

     ![Sentineladdworkplace](./images/ActB-T06-create-log-analytics-ws-to-sentinel.png 'addworkplacetosentinel') 

これでハンズオンの事前準備は完了です  
Azure 費用が発生することを防ぐため、ハンズオン実施までの間は必ず仮想マシンを **停止** ください  


