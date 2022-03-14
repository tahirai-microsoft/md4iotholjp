# Internet of Things - Microsoft Defender for IoT ハンズオン

作業を開始する前に [事前準備](../Before%20HOL/Microsoft%20Defender%20for%20IoT%20BHOL.md "Microsoft Defender for IoT Before Hands-on-Lab") が完了しているか、今一度ご確認ください  

> 最終更新日: 2022/03/13  
> 対象 : Defender for IoT 10.5.x

## アーキテクチャ ダイアグラム ##

このハンズオンでは、IoT センサー向け、オンラインアラート、そしてオフラインシナリオ用の Microsoft Defender のセットアップに重点を置き、環境の構築方法、結果の評価、及び Microsoft Sentinel のような SIEM システムとの統合方法について学習します  

具体的にはブラウンフィールド（既存）、及びグリーンフィールド（新規）デバイスが存在する施設のセキュア化に焦点を当てます  
（グリーンフィールド デバイス向けの作業はまだハンズオンに組み込まれていません）

以下の図は工場に当てはめた例ですが、同様なシナリオはエネルギープラントなど多く存在します  

![Architecture](./images/E0T0-architecture-diagram.png 'Architecture Diagram')

## **コンテンツ:** ##

- [演習 #1: Defender を有効化する](#Exercise-1-Enabling-Defender)
   - [作業 1: Microsoft Defender for IoT を有効化する](#Task-1-Enabling-Microsoft-Defender-for-IoT)
   - [作業 2: IoT Hub を作成する](#Task-2-Create-an-IoT-Hub)
   - [作業 3: センサーをオンボード（有効化）する](#Task-3-Onboarding-sensors)
- [演習 #2: オフラインセンサーを設定する](#Exercise-2-Setting-up-your-offline-sensor)
   - [作業1: オフラインセンサー（nested VM）を設定する](#Task-1-Set-up-your-nested-Virtual-Machine)
   - [作業2: オフライン用の Microsoft Denfender for IoT を構成する](#Task-2-Configure-a-Microsoft-Defender-for-IoT-offline-sensor)
- [演習 #3: システム設定を有効にする](#exercise-3-enabling-system-settings)
   - [作業1: システム プロパティ](#task-1-System-properties)
   - [作業2: PCAP ファイル](#task-2-Pcap-Files)
- [演習 #4: データを分析する](#Exercise-4-Analyzing-the-Data)
   - [作業1: デバイスマップ](#Task-1-Devices-Map)
   - [作業2: アラート](#Task-2-Alerts)
   - [作業3: デバイス インベントリ](#Task-3-Device-Inventory)
   - [作業4: イベントタイムライン](#Task-4-Event-Timeline)
   - [作業5: データ マイニング](#Task-5-Data-Mining)
- [演習 #5: オンラインセンサーを設定する](#Exercise-5-Online-Sensor)
   - [作業1: センサーの再設定](#Task-1-reconfiguring-sensor)
- [演習 #6: Sentinel と統合する](#Exercise-6-Integrate-with-Sentinel)
   - [作業1: IoT を Sentinel に統合する](#Task-1-Enabling-IoT-to-Integrate-with-Sentinel)
   - [作業2: データコネクターを接続する](#Task-2-Connecting-Data-Connectors)
   - [作業3: アラートを確認し PCAP を実行する](#Task-3-Acknowledge-Alerts-and-Re-run-PCAPs)
   - [作業4: IoT インシデントと Sentinel の連携](#Task-4-Sentinel-interaction-with-IoT-Incidents)
   - [作業5: Kusto クエリでアラート詳細を検索する ](#Task-5-Kusto-Query-Language-to-Find-Alert-Details)
- [演習 #7: クリーンアップ](#Exercise-7-Clean-Up)
   - [作業1: リソースの削除](#Task-1-Delete-resources)
- [付録 #1: トラブルシューティング](#Appendix-1-Troubleshooting)

## 演習 #1: Defender を有効化する
### 作業 1: Microsoft Defender for IoT を有効化する
この作業は、後に用いる Microsoft Defender for IoT をホストする VM 上ではなく、PC から Azure ポータルにアクセスして行います  
1. [Azure ポータル](https://portal.azure.com/#home "Microsoft Azure Home") の検索ボックスに **defender for iot** を入力し、サービスの項目に表示された **Microsoft Defender for IoT** をクリックします  
   ![Microsoft Defender for Cloud Getting Started](./images/E1T1-Find-Microsoft-Defender4IoT.png 'Microsoft Defender for IoT')
1. 左メニュー Management の **Pricing** をクリックします  
   ![Defender for IoT](./images/E1T1-Defender4IoT-GettingStarted.png 'Defender for IoT Pricing')

1. 画面中央に表示された **サブスクリプションのオンボード** をクリックします  
   ![Defender for IoT Trial](./images/E1T1-Defender4IoT-OnboardSubscription.png 'Defender for IoT Free Trial')  

1. サブスクリプションが今回のハンズオンに用いるものであることを再確認の上、Purchase method を **Trial** に変更し、**I accept the terms** にチェックを入れ、**保存** をクリックします  

   ![Defender for IoT Evaluate](./images/E1T1-Defender4IoT-Evaluate.png 'Defender for IoT Trial')

以上で 100台 までデバイスを監視可能な Microsoft Defender for IoT の 30日間 無償試用版が開始します

### 作業 2: IoT Hub を作成する
この作業は PC から Azure ポータルにアクセスして行います  

このハンズオンではオンラインセンサー、オフラインセンサーの両方を利用します  
オフラインセンサーはネットワークが切断された状態でも動作しますが、オンラインセンサーは Azure IoT Hub に接続する必要があります  

センサーをオンボードする前に、オンラインセンサー用の Azure IoT Hub を作成します  

1. [Azure ポータル](https://portal.azure.com/#home "Microsoft Azure Home") の検索ボックスに **IoT Hub** と入力し、サービスの項目に表示された **IoT Hub** をクリック、**作成** をクリックします  

1. 基本タブで必要な情報を記入します  
   - **サブスクリプション**: ハンズオンで用いるものが選択されていることを再確認します
   - **リソースグループ**: ハンズオン用に作成したリソースグループを選択します
   - **IoT Hub 名**: IoT Hub の名前を入力します (md4iot-固有のサフィックスを推奨、例: md4iot-tahirai)
   - **領域**: IoT Hub を設置するリージョンを選択します（Japan East を推奨）

   ![IoT Hub Create](./images/E1T2-iothub-create.png 'Create an IoT Hub')
   
1. 管理タブで価格とスケールティアが **S1: Standard レベル** になっていることを確認し、画面下部の **確認および作成** 、続いて **作成** をクリックします

ここで Microsoft Sentinel がアラートを収集できるように アクセス制御 を追加します

1. [Azure ポータル](https://portal.azure.com/#home "Microsoft Azure Home") で **サブスクリプション** をクリックし、ハンズオンで用いるものをクリックします

1. メニューから **アクセス制御 (IAM)** をクリック、続いて **追加** 、**ロールの割り当ての追加** をクリックします  

1. **共同作成者** をクリックし、**次へ** をクリック、**メンバーを選択する** でサブスクリプションにアクセスしているアドレスを検索、**選択** をクリック、戻った画面で **レビューと割り当て** をクリックします  

   ![Contributor Role](./images/E1T2-Subscription-Contributor-role.png)

### 作業 3: センサーをオンボード（有効化）する
この作業は PC から Azure ポータルにアクセスして行います  

このハンズオンではインターネットへの接続が不要なオフラインセンサーと、Azure に接続されたオンラインセンサーの両方を利用します  
まず、オフラインセンサーのオンボーディング（有効化）を行います  

1. [Azure ポータル](https://portal.azure.com/#home "Microsoft Azure Home") の検索ボックスに **Defender** を入力し、サービスの項目に表示された **Microsoft Defender for IoT** をクリックします  

1. **センサー** タブをクリックし、**アプライアンスの購入とソフトウェアのインストール**、**バージョンの選択** で **10.5.5（安定）以上 - 推奨** を選択し、**ダウンロード** をクリック、ISO ファイルを入手します  
   ![Onboard sensor](./images/E1T3-download-sensor-iso.png 'Download Sensor ISO')  

   > **NOTE:** お問い合わせ ポップアップウィンドウ は **送信せずに続行** で継続ください  
1. 左メニュー Management の **Site and sensors** をクリックし、**OTセンサーのオンボード** をクリックします  
   ![Onboard sensor](./images/E1T3-onboard-sensor.png 'Onboard Sensor')  

1. センサー名 に **myofflinesensor** と入力、サブスクリプションに今回ハンズオンで用いるものを選択し、**クラウドに接続済み** を無効にした後 **登録** をクリックします  
   ![Register offline sensor](./images/E1T3-register-sensor.png 'Register Sensor')  
1. センサーアクティベーション用のファイル保存が促されますので、保存して **Finish** をクリックします
   ![Activation Offline sensor](./images/E1T3-download-activation-file.png 'ActivationOffline Sensor')  
1. **Site and sensors** に新しいセンサーがオンボードされたのを確認します  
   ![Sensor Onboarded](./images/E1T3-locally-managed-sensor.png 'Sensor Oonboarded')  

続いてもう一つのセンサー、オンラインセンサーをオンボーディングします  

1. **OTセンサーのオンボード** をクリックし、以下のように記入、設定します  
   - **センサー名:** myonlinesensor  
   - **サブスクリプション:** 今回ハンズオンで用いるものを選択  
   - **クラウドに接続済み:** 有効（標準のまま）  
   - **脅威インテリジェンスの自動更新:** 有効（標準のまま）  
   - **センサーのバージョン:**  
サイト セクションは、  
   - **名前:**  
   - **表示名:**  
   - **ゾーン:**  
   
   ![Reg Online Sensor](./images/E1T3-register-online-sensor.png 'Online Sensor')

1. **登録** をクリックします  
1. センサーアクティベーション用のファイル保存が促されますので、保存して **Finish** をクリックします

1. 改めて **Site and sensors** で2つのセンサーがオンボードされたのを確認します  

2つのファイル（センサー向けアクティベーション用ファイル）をダウンロードしました  
Defender for IoT センサーをホストする 仮想PC(VM) への接続には RDP を利用しますので、アクティベーション用ファイルを利用する VM に対して copy (ctrl-c) and paste (ctrl-v) でコピーできます  

## 演習 #2: オフラインセンサーを設定する
この作業は [事前準備](../Before%20HOL/Microsoft%20Defender%20for%20IoT%20BHOL.md "Microsoft Defender for IoT Before Hands-on-Lab") で作成した VM 上で行います

### 作業 1: オフラインセンサー（nested VM）を設定する

事前準備で作成した WIndows 10 VM に RDP でログインします  

1. コマンドプロンプトで "ipconfig" を実行します


1. ホストの Ethernet アダプタ の IPアドレス を控えます、 (Default Switch) は無視してください

   > **NOTE:** この例では ホストの Ethernet アダプタに 10.0.0.4 の IP が割り当てられていますので、192.168.0.0/24 を "NATSwitch" のネットワークスコープとして用いますが、もし Primary アダプタで 192.168.x.x を既に利用している場合には、172.27.0.0/24 を用いてください

1. PowerShell を検索し、右クリックから **管理者として実行** をクリックします

1. 次の2つのコマンドを PowerShell 上で実行します
   ```powershell
   New-VMSwitch -SwitchName "NATSwitch" -SwitchType Internal
   ```
   ```powershell
   New-VMSwitch -SwitchName "MySwitch" -SwitchType Internal
   ```
1. 次のコマンドを実行し、ネットワークアダプタ の情報をローカル変数に保存します
   ```powershell
   $s1 = Get-NetAdapter -name "vEthernet (NATSwitch)"
   ```
1. IP アドレスを NATSwitch に割り当てます (192.168.0.1 または 172.27.0.1)
   ```powershell
   New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex $s1.ifIndex
   ```
1. 新しい NAT ネットワークを作成します (192.168.0.0/24 または 172.27.0.0/24)
   ```powershell
   New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
   ```
   ![PowerShell Admin](./images/E2T1-switches-network.png 'Define switches and network')


1. VM 内の検索ボックスに **Hyper-V** と入力し Enter を押すと新しい Hyper-V コンソールが表示されますので、左メニューから **New** を選択、表示された項目の中から **Virtual Machine...** をクリックします

   ![Create nested vm](./images/E2T1-create-nested-vm.png 'New Virtual Machine')

1. VM を以下のように設定します

   - 最初のタブで VM の名前に **md4iotsensoroffline** を入力し、**Next** をクリック
   - **Specify Generation** で **Generation 1** を選択し、**Next** をクリック
   - メモリを **8196MB** に変更し **Next** をクリック
   - **Configure Network** タブの **Connection** で **NATSwitch** を選択し、**Next** をクリック
   - **Connect Virtual Hard Disk** タブで **Create a virtual hard disk** で **Next** をクリック
   - **Installation Options** で **Install an operating system from a bootable CD/DVD-ROM** を選択、**Image file (.iso)** でダウンロードした Azure Defender の iso ファイルを指定し **Finish** をクリック

   ![Disk Size](./images/E2T1-select-os-image.png 'OS Image')

1. 作成した VM を右クリック、**Add Hardware** セクションの **Settings** を選択、 **Network Adapter** を選択し、**Add** をクリック、ここで **My Switch** という名前で以前作成した Virtual Switch を選択し、**Apply** 、続いて **OK** をクリック、仮想プロセッサー数を **1** から **4** に増やし、**Apply** 、続いて **OK** をクリックします


### 作業2: オフライン用の Microsoft Denfender for IoT を構成する

### Task 2: Configure a Microsoft Defender for IoT offline sensor

During this task we will configure Azure Defender based on the IPs highlighted before, this first configuration will be based on an offline sensor.

1. In the Hyper-V Manager, find the **Connect...** in the lower right hand of the screen and click on it, and in the newly opened VM connection window click **Start**.

1. When you connect to the Ubuntu VM you should see the following screen to start the configuration process. 

   > **Note!**: If you don't see the screen below, your installation timed out or you pressed enter, selecting a different configuration by mistake, delete the virtual machine and start this task over. The timeout period is relatively short so make sure you connect immediately to the nested VM and select the language and the sensor type (in Task 2).

   ![Connect to Sensor](./images/E2T2-connect-to-sensor.png 'Initial connect to offline Sensor')

1. Press **Enter** for English.

1. Select the third option *(Office 4CPUs)* and press **Enter**.

   ![Setting up Sensor](./images/E2T2-sensor-type.png 'Setting up the offline Sensor')

   At this moment, the offline sensor will be installed (including its operating system). This installation takes some time, expect it to run for approximately 15 minutes.

1. As part of the installation process, you will be asked to provide some parameters, it is ***VERY IMPORTANT*** you paid attention to the previous task because you will use the network information you captured before. This information is unique to each Virtual Machine. So the following is an **EXAMPLE**.

   - **configure hardware profile**: **office**, then press enter. 
   - **Configure network interface**, type **eth0**
   - **Configure management network interface**: in this example we're using **192.168.0.50**, you will use one of the **Ipv4 Addresses** depending on your network scope from the previous task, either **192.168.0.50 or 172.27.0.50**. Click Enter to continue. ***Take a note of this IP you will need it later on***.
   - **Subnets mask**: **255.255.255.0** this will be the SAME for everyone.
   - **Configure DNS**: **8.8.8.8**
   - **Configure default gateway IP Address**: We are intentionaly mis-configuring this value to force the sensor in **offline** mode. Use either 192.168.0.**2** or 172.27.0.**2**.
   - **Configure input interface(s)**: **eth1**
   - **Configure bridge interface**: Just press Enter
   - Then type **Y** to apply the changes and click **Enter**.

   Below, a ***sample*** screen, your parameters might be different.

   ![Configuring Sensor](./images/E2T2-defender-config.png 'Configuring the offline Sensor')

   Now the installation will continue running for another 10-15 minutes.

1. ***IMPORTANT STEP!!!*** Once the installation is complete, you will have the login information availabe in the screen **TAKE A SCREENSHOT!!** before continuing, press **Enter**. Now you will have the support account login information, again **TAKE THE SCREENSHOT!!** press **Enter** to continue. If you fail to capture the credentials, you will need to start over.

   ![Sensor Credentials](./images/E2T2-credentials.png 'Saving Sensor Credentials')

1. Once the installation finished you will ask to login, enter the credentials from previous step. In this screen you can also validate the IP, you will use that IP in your browser.

   ***Note:** At this stage your IPs should look similar to the example below. If you can't reach the portal validate the IPs. If you restarted your VM there is a chance your IPs changed so you will need to go back and reconfigure them, if that is the case follow the troubleshooting guidance below.*

   ***Troubleshooting Note:** Once the installation is complete, you will be able to access Azure Defender Console.
   Check if you can open a cmd window, ping the IP Address you entered in the step 'Configure management network interface'.
   If the request times out, you will need to reconfigure this step again, for that review the IPs one more time and use the command below to start over:*

   ```bash
   sudo cyberx-xsense-network-reconfigure
   ```

   In the next steps you will be prompt to enter the password capture above, some characteres look alike but they are not, this image will help you to identify some of them.

   ![Defender characters](./images/E2T2-characters.png 'Different defender characteres')

1. Login with the credentials provided in step **4**.

   ![Defender IP](./images/E2T2-AzureDefenderForIoTSensor.png 'Defender IP Address')

   > **NOTE:** the "md4iotsensoroffline" VM's keyboard layout is US by default, and it may not match the layout of your physical keyboard. To avoid issues when entering the password, you may use the windows 10 on-screen keyboard. To run it, type "osk" in the search box and click on "On-Screen Keyboard"...

   ![Start-OSK](./images/E2T2-Start-OSK.png 'Start on-screen keybaord')

   > ...and use it to enter the credentials:

   ![OSK-Keyboard-Logon](./images/E2T2-keyboard.png)

1. Next, you will be ask to activate the product, click **Upload**, then **Browse Files**, in your downloads folder select the file you downloaded from the Storage Explorer, in this example **myofflinesensor.zip**.

   ![Defender Login](./images/E2T2-offline-sensor-activation.png 'Defender Login Screen')

1. Click **Approve these terms and Conditions**, then **Activate**.

1. You will be prompted to select **SSL/TLS Certificates | Onboarding 1/2** for this lab will use the second option **Use a locally generated self signed certificate(..)**. Then click **I CONFIRM**, **Next**.

   ![Certificate selection](./images/E2T2-offline-sensor-certificate.png 'Defender certificate selection')

1. For this lab in the next step we will **Disable** the system wide validation. **Finish**.

1. Let's analyze together what information we already have available before moving forward.

## Exercise 3: Enabling system settings

### Task 1: System Properties

1. In your offline sensor you will find **System Settings** on the left side of the Azure Defender portal, click there as shown below.

   ![Select System Settings](./images/E3T1-System-Settings.png 'Defender for IoT Select System Settings')

1. Next, look for the icon **System Properties** on the right side. Click on the icon. You you will see a pop up warning, click **Ok**.

1. In the new window on the left side, scroll down until you see **Pcaps**, click there. Now on the right side scroll all the way down and we will modify three parameters as shown below:

   - **player_max_amount=200**
   - **enabled=1**
   - **player.params=-M 3** <br> <br>

   Amongst others, these settings enable the PCAP player and allow it to playback faster than real-time.

   ![Set PCAPS](./images/E3T1-Enable-PCAPs.png 'Enable PCAP Settings')

1. Click **Save** and then **Ok**.

1. At this point you should see the Pcap Player available (you can close the **Edit System Properties** screen now by clicking the **Cancel** button):

   ![Pcap Player](./images/E3T1-PCAP-Player.png 'Pcap player')

### Task 2: Pcap Files

1. In a previous step you already downloaded a  **holpcaps.zip** file from the Storage account. It should be in your Azure Virtual Machine's **Downloads** folder.

1. Unzip **holpcaps.zip**

1.  Go back to Azure Defender, Click on **System Settings**, then **PCAP Player** now select **Upload**,
**Browse Files**, browse to the folder where you download the files in the previous step, select all the files and click **Open**. This operation will take a few minutes to upload all the files.

1. At this point you should see all the files uploaded.

   ![Pcap Files Ready](./images/E3T2-PCAP-Files-Uploaded.png 'PCAP Files Uploaded')

1. Click on **Play All**, in a few minutes you will receive a message saying all the files has been played. 

## Exercise 4: Analyzing the Data

After Defender for Cloud learnt about your environment it will be able to share insights pretty fast.

### Task 1: Devices Map

Your first interaction with Devices map you will see a similar map like the one below (details of what you actually see may vary):

![Pcap Files Uploaded](./images/devices-map.png 'Pcap files uploaded')

1. Use the four icon bar on the left to select **Layout by Purdue**. In this model you will see the different layers between Corporate IT and site operations.

   ![purdue-layout](https://user-images.githubusercontent.com/60540284/140969899-b83965cc-0900-4f45-95df-e944856d99d3.gif)

1. Check your notifications available and you can take action at this point.

   ![notifications](https://user-images.githubusercontent.com/60540284/140969923-5634ea88-6d74-4b7b-9e13-c278e0cce20f.gif)

1. For each device right click to analyze properties, show events, reports and simulate attack vectors.

   ![device-right-click](https://user-images.githubusercontent.com/60540284/140969957-1f51fa73-1e20-4930-8c8d-3271ecb68149.gif)

1. In the hamburger menu on the left, click the highlights and select one of the **OT Protocols** i.e. **MODBUS** and click on **Filter**. Now your map will show those devices only

   ![modbus](https://user-images.githubusercontent.com/60540284/140970027-dad74aba-4d88-45cb-8505-830c62b3ecc0.gif)

1. Then filter your devices by **CIP** OT Protocol, at the bottom of your map you will see a PLC, where the Vendor is Rockwell Automation, has already 3 alerts activated. Right click on the device, **View Properties**. In this view you will be able to analyze the Backbone of your PLCs, take actions and analyze the Alerts.

   ![cip](https://user-images.githubusercontent.com/60540284/140970072-7db949da-f87c-41ef-88c0-45cea6da0f62.gif)

### Task 2: Alerts

1. Once you click Alerts in your PLC you will see a new window pop up showing three different types of alerts.

   - Operational(high Alert and lower alert)
   - Policy Violation

   For each of these alerts you will be able to analyze the pcap file, export a report, analyze the timeline or mute the alert.

   ![ex4-t2-1](https://user-images.githubusercontent.com/60540284/141076357-ef22ec24-1d94-462b-8076-a3077cbca2a7.gif)
    
1. If we remove the device filter from the top of the screen, then click **Confirm** you will see 20 Alerts in process.

1. Apply **Custom Groups** to filter different scenarios, such as **Unclassified subnets** then **Confirm**

   ![ex4-t2-2-3](https://user-images.githubusercontent.com/60540284/141076872-1b8350d6-ad56-4444-995d-256ce0785c81.gif)

### Task 3: Device Inventory

1. In this view, filter all your devices by **Is Authorized**, True or False are possible values.

   > **NOTE:** if you don't see the column "Is Authorized", click on the "Device Inventory Settings" gear icon (upper-right corner) and add it to the view.

   ![ex4-t3-st1](https://user-images.githubusercontent.com/60540284/141078788-04910c9d-6dfe-4a03-bc93-42c26d08d778.gif)

1. Organize your devices based on filters.

1. Export the list to a csv files.

### Task 4: Event Timeline

This view will allow you a Forensic analysis of your alerts.

1. Choose **Advanced Filters**, filter the timeline by **CIP**, let's analyze the alert timeline.

   ![Event-Time-Line-by-CIP](./gifs/MD4IoT-EventTimeline.gif)

### Task 5: Data Mining

In this section you can create multiple custom reports.
As an example we will create a Report based on firmware updates versions.

1. Go To **+**, **New report**, in the categories section select **Modules and Firmware update versions**

1. Assign a name to your report. Then go to Filters, **add** and select **Firmware version(generic)**

   ![Create-New-Report](./images/E4T5-Create-New-Report.png 'Create PLC Firmware Version Report')

1. In the new field added **Firmware Version(GENERIC)** add **0.4.1**, then **Save**.

1. You can remove the filter to list all the firmware updates version in your list also.

1. Export you report(pdf, csv) for further actions. 

### Task 6: Risk Assessment

1. Go to the Risk assessment, run the assessment. During this task we will show you how to analyze the assessment. 

## Exercise 5: Online Sensor

To modify our sensor to become an online sensor, we will use the same virtual machine that we used for the offline sensor, but we will reactivate the sensor using **System settings**. In a real scenario you probably would create a new sensor, running in its own virtual machine or physical appliance.

### Task 1: Reconfiguring sensor

To modify your sensor to be connected with Azure, you will need to modify the network configuration.

1. In your sensor's Azure Defender for IoT Portal (in the Virtual Machine), select **System Settings** and **Network**.

   ![NetworkSettings](./images/E5T1-SystemSettings-Networking.png 'Change Network Settings')

1. Change the IP Address of the Default Gateway to 192.168.0.1 or 172.27.0.1, depending on the settings you used earlier in the HOL.
 
   ![ChangeDefaultGatewayIP](./images/E5T1-Change-Default-Gateway.png 'Change Default Gateway IP') 

1. On the "md4iotsensoroffline" Virtual Machine Connection, select **Action** and **Ctrl+Alt+Delete** to reboot the sensor.

   ![RebootSensor](./images/E5T1-Reset-Sensor-VM.png 'Reboot Sensor VM')

1. Login to your sensor's Azure Defender for IoT Portal (in the Virtual Machine) again, select **System Settings** and then, **Reactivation**.

1. In the new window, select **Upload**, **Browse File**, select the zip file you downloaded from the storage account in previous steps **myonlinesensor.zip**, then **Open** and **Activate**, **Ok** to the instructions

   ![ReactivateSensor](./images/E5T1-Sensor-Reactivation.png 'Reactivate')

1. Last, you should receive a message showing your sensor modified to **Connected**. 

1. Close the screen, open again the **Reactivation** window and double check if your sensor is **Cloud Connected** as shown below:

   ![OnlineSensor](./images/E5T1-CloudConnected-Sensor.png 'Reactivate as Online Sensor')

1. Run the Pcap files again in your console. In a few minutes you can verify if IoT Hub in Azure Portal on your physical machine is receiving messages from your sensor:

   ![IoT Hub Azure](./images/E5T1-Monitoring-IoTHub.png 'Monitoring IoT Hub')

1. In the same IoT Hub now you should see the alerts generated by Defender for IoT. Scroll down to **Defender for IoT**, select **Security Alerts**, on the right side you will see some alerts already available.

   ![IoTHub](./images/E5T1-IoTHub-SecurityAlerts.png 'IoT Hub Security Alerts')

## Exercise 6: Integrate with Sentinel

You will execute most of this task on your physical machine, not in the Virtual Machine that hosts your your Microsoft Defender for IoT sensor.

> **Note**: Please ensure you have completed Task 6 in the ['Before HOL'](../Before%20HOL/Microsoft%20Defender%20for%20IoT%20BHOL.md "Microsoft Defender for IoT Before Hands-on-Lab") instructions prior to working through the following tasks.

### Task 1: Enabling IoT to Integrate with Sentinel

1. Ensure your IoT Hub is configured to send Security Alerts to Sentinel.
1. Navigate to your IoT Hub > Defender for IoT > Settings > Data Collection

   ![Data Collection](./images/E6T1-Data-Collection.png 'IoTHub Data Collection')

1. Double check that Data Collection blade, is enabled for **Enable Microsoft Defender for IoT**
 
   ![Data Collection DIoT](./images/E6T1-Data-Collection-D4IoT.png 'Data Collection D4IoT')

### Task 2: Connecting Data Connectors
	
1. With the *Microsoft Defender for IoT* switch enabled, go to **Microsoft Sentinel** > Configuration > Data Connectors > Search **Microsoft Defender for IoT** to connect Microsoft Defender for IoT to Microsoft Sentinel.
 
   ![DataConnectorsSentinel](./images/E6T2-Data-Connectors-Sentinel.png 'Data Connectors Sentinel')

1. Click the **Open Connector Page**
 
   ![DataConnectorsIoT](./images/E6T2-Data-Connectors-IoT.png 'Data Connectors IoT')

1. Review the instructions and click the “Connect” button to connect Microsoft Defender for IoT to Sentinel. If the connection continues to fail, this will most likely be due to the user not having the "Contributor" permissions and you may have missed the access step in the prerequisites. 

   ![Sentinel Connect](./images/E6T2-Sentinel-Connect.png 'Sentinel Connect')

1. If connected correctly you should expect to see the Status change to “Connected” and the link light up green.
 
   ![Sentinel Connect 2](./images/E6T2-Sentinel-Connected.png 'Sentinel Connect 2')

1. Use the next steps tab to enable Out of the Box alerts. For example, click the create rule and follow the instructions to turn on the rule.
 
   ![SentinelRuleCreation](./images/E6T2-Create-OOB-Rule.png 'Sentinel Rule Creation')

1. Fill in the “Name” and click **Review and Create**, followed by **Create**. This is enabling incidents to be created based on the Azure Defender IoT alerts that are ingested into Sentinel.
 
   ![SentinelRuleSubmission](./images/E6T2-OOB-Rule-Created.png 'Sentinel Rule Submission')

1. Additionally, you can create the rule not only on the data connectors page but also on the Microsoft Sentinel “Analytics” blade. See an example below when you go to the “Rule Templates” tab and filter data sources by “Microsoft Defender for IoT (Preview)”.

   ![SentinelAnalyticsScreen](./images/E6T2-Sentinel-Analytics-Screen.png 'Sentinel Analytics Screen')

### Task 3: Acknowledge Alerts and Re-run PCAPs

You will execute most of this task on the Virtual Machine that hosts your your Microsoft Defender for IoT sensor.

1. Go back to your browser interface and acknowledge all of the alerts. The reason we are doing this is so we can re-run the alerts to show how they are sent and analyzed by Sentinel.

	1. Navigate to the Alerts Page
	1. Click the double check box
	1. Click **Ok** to acknowledge the alerts 
    
   ![iot-portal-acknowledge-alerts](./images/E6T3-IoT-Portal-Ack-Alerts.png 'IoT Portal Acknowledge alerts')

    1. Now go to the System Setting tab.
    1. Click the **Play All** on the PCAP Files to replay simulating the alerts.

   ![Rerun-pcaps](./images/E6T3-Rerun-pcaps.png 'Rerun pcaps')

### Task 4: Sentinel interaction with IoT Incidents

You will execute most of this task on your physical machine, not in the Virtual Machine that hosts your your Microsoft Defender for IoT sensor.

1. Go back to the Sentinel console and under the **Threat Management** section, select the **Incidents** tab.  Filter by Product Name **Azure Defender for IoT**.

   ![SentinelFilterAlerts](./images/E6T4-Sentinel-Incidents.png 'Sentinel Filter Alerts')

1. Select one of the alerts and click **View full details**

   ![IncidentDetails](./images/E6T4-Sentinel-Incident-Details.png 'Incident Details')

1. It will take you to this screen to get all the information relative to the incident. This allows analyst to get more details on the entity including what other alerts made up the incident, playbooks to enrich the context of the alert, and comments section to leave details on what the analyst discovered during review or how they came to the determination to dismiss the incident.

   ![IncidentFullDetails](./images/E6T4-Sentinel-Incident-FullDetails.png 'Incident Full Details')

1. By clicking the **Investigate** button, you can dig deeper in the cause of the incident and the relation to other incidents.

   ![IncidentInvestigate](./images/E6T4-Sentinel-Incident-Investigate.png 'Incident Investigate')

### Task 5: Kusto Query Language to Find Alert Details

1. Navigate to the “Logs” tab and run this query. Querying the data will provide the ability to join tables and datasets to curate data from multiple sources. KQL is a similar language to SQL but will take some research and some dedicated time to become familiar with.

   Here are two basic examples:

   ```sql
	SecurityAlert | where ProviderName contains "IoTSecurity"
   ``` 
 
   ![KustoQquery](./images/E6T5-FindAlerts-IoTSecurity.png 'kusto query')

   ```sql
   SecurityAlert | where CompromisedEntity == "hub-md4iot-mst01"
   ```
 
   ![KustoQuery2](./images/E6T5-FindAlerts-IoTHub.png 'kusto query 2')

## Exercise 7: Clean Up

### Task 1: Delete resources

The Azure Passes will allow you to run the services for 90 days for training purposes. Although it is a best practice to delete all your resources after the training. 

Search for the Resource Group created for this training.

Select Delete resource group on the top right side.

Enter your-resource-group-name for **TYPE THE RESOURCE GROUP NAME** and select Delete. This operation will take a few minutes.

After that is done go to Microsoft Defender for IoT and deactivate the subscription.


## Appendix 1: Troubleshooting

1. If your Defender portal is not working properly run the following command to validate if the components are running properly

   ```powershell
   cyberx-xsense-sanity
   ```
   ![Sanity check](./images/A1T1-sanity-check.png 'Sanity check')

1. If your IoT hub is not receiving messages, check if ubuntu machine can reach IoT Hub, first run the following command to identify the IP of your IoT Hub:

   ```powershell
   netstat -na | grep EST | grep -v 127.0.0.1
   ```

   ![Sanity check](./images/A1T1-ips-connected.png 'Sanity check')

   Then, ping the IoT Hub using the connection string from the overview blade in Azure Portal.

   ![Ping IoT Hub](./images/A1T1-ping-iot-hub.png 'Ping IoT Hub')

