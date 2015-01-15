---
layout: page
categories:
- blog
title: RMBluetooth - BTLEでROMOのログを出力
---

### 概要

[https://github.com/sonsongithub/RMBluteooth](https://github.com/sonsongithub/RMBluteooth)

RMBluetoothは，ROMOでリモートホストにBluetoothLE経由でログを出力することを目的としたライブラリだ．
BluetoothLEの性質上，接続先を選ぶ必要がある．
今回の実装では，ROMOに接続するiOSデバイスをCentralに，ログを読むデバイスをPeripheralにした．
ログを受け取る側のデバイスは，OSX/iOSの両方を使える．

手っ取り早くサンプルを使うと挙動がわかる．

1. RMBTReceiver
  ROMOに接続するiOSデバイス上で実行するアプリケーション．
  
2. RMBTControllerSampleOSX/RMBTControllerSample
  ログを受け取るOSX/iOS用のアプリケション．

まず，ROMO側で接続先のデバイスを選択する．
RMBluetoothは，ROMOの接続状況を監視しているので，ROMOが接続されると，接続がUIで表示される．
RMBTControllerSampleOSXでは，左ペインにコントロール用のUI，右ペインにログビューが配置される．
ROMOを接続した状態で，BluetoothLEを接続すると，左ペインのボタンを押すと，ROMOが動き出す．

ログは右ペインに出力される．
RMBTReceiverは，ROMOの接続時および切断時にログをRMBTControllerSampleOSXへ送る．

### 使い方 - Receiver(ROMO)側

ROMOに接続したiPhoneでは，デバイスを選択するビューコントローラを利用する必要がある．

    - (void)viewDidLoad {
        [super viewDidLoad];
        [RMCore setDelegate:self];
        [RMBTReceiver sharedInstance].delegate = self;
    }

    - (IBAction)open:(id)sender {
        [self presentViewController:[RMBTDeviceSelectViewController viewController] animated:YES completion:nil];
    }

    - (void)robotDidConnect:(RMCoreRobot *)robot {
        if (robot.isDrivable && robot.isHeadTiltable && robot.isLEDEquipped) {
            self.robot = (RMCoreRobot<HeadTiltProtocol, DriveProtocol, LEDProtocol> *) robot;
            [[RMBTReceiver sharedInstance] sendLog:@"ROMO is connected."];
        }
    }

    - (void)robotDidDisconnect:(RMCoreRobot *)robot {
        if (robot == self.robot) {
            self.robot = nil;
            [[RMBTReceiver sharedInstance] sendLog:@"ROMO is disconnected."];
        }
    }

### 使い方 - Controller(OSX/iOS)側

    - (void) viewDidLoad{
        [super viewDidLoad];
        [[RMBTController sharedInstance] setDelegate:self];
        _log = [NSMutableString string];
    }

    - (void)controller:(RMBTController*)controller didReceiveLog:(NSString*)log {
        NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
        formatter.dateStyle = NSDateFormatterMediumStyle;
        formatter.timeStyle = NSDateFormatterMediumStyle;
        [_log appendFormat:@"[%@] %@\r", [formatter stringFromDate:[NSDate date]], log];
        _textView.text = _log;
    }

### 動きの例

下の動画は，iPadでiPhoneがささったROMOを制御しているところだ．
ログ画面にROMOを接続したタイミングでログが出力されていることがわかる．
最初にROMOに接続されたiPhoneの画面から，接続先のデバイスを設定し，BluetoothLEの接続完了する．
その後は，ボタンでラジコンのように操作できる．

<iframe width="480" height="360" src="http://www.youtube.com/embed/Mp9KDvKB0Is" frameborder="0"></iframe>

### 既知の課題

BluetoothLEが不慮の事態切断された後，BluetoothLEの挙動がおかしくなる．
これは，一旦，Macの場合なら，右上のメニューバーのBluetoothアイコンから接続中のiPhoneを切断する．
それでもうまくいかない場合は，一度，Bluetooth自体の電源を入れ直せば良い．

### 応用

BluetoothLEを使ったログ吐きは，何もROMOに限って必要なものではないので，色々応用できる可能性がある．
理想を言うと，Bluetoothに関するコードとROMOのコードを完全に疎結合にできれば，問題ないはず．
いずれは，RMBTReceiverやRMBTControllerの役割を整理して，その子クラスでROMO関連を含んだクラスを実装できればいいと思っている・・・・・．