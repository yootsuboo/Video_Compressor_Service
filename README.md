# Video_Compressor_Service
Recursion project

#### ステージ1
クライアントは TCP を通じてサーバに接続し、CLI プログラムを使ってサーバにファイルをアップロードします。

#### ステージ 2
ユーザーがファイルをアップロードした後、動画処理が行われ、処理済みの出力ファイルがユーザーに返される機能を実装します。

#### ステージ 3 （任意）
GUI デスクトップクライアントやモバイルクライアントの開発、暗号化などの追加機能を実装します。

#### 学習目的
このプロジェクトの主要な目的は、クライアントとサーバのサービスを構築する過程で、サーバ操作の基本を身につけることです。サーバに慣れることで、オペレーティングシステムをより効果的に活用できます。バックエンドエンジニアは、サーバ上で動くアプリケーションの開発、オペレーティングシステムのシステムコール（例えば、ファイルやキャッシュ共有メモリの生成、プロセススケジューリングなど）、データベースへのアクセス、Linux 環境での作業、基本的なネットワーキングとセキュリティ（暗号化や認証など）、そしてアプリケーションのスケーラビリティに関わる業務が中心となります。

副次的な目標としては、TCP/UDP の内部構造についての理解やプロトコルの基本概念を把握し、独自の低レベルネットワーキングをアプリケーションに実装する能力を高めることがあります。通常、特にインフラソフトウェアのバックエンドチームにいない限り、ネットワーキングには内部またはサードパーティが構築した SDK や API が用いられ、ウェブアプリケーションでは HTTP/S が使用されることが一般的です。しかし、このプロジェクトを通じて自分自身で実装することで、ネットワーキングの基礎を身につけ、サーバサイドのアプリケーション開発のスキルを向上させることができます。これはまさに一石二鳥の状況です。

## ステージ1
推奨使用言語：Python（サーバとクライアント両方で）

### 内容
クライアントがサーバに接続し、mp4 ファイルをアップロードするためのサービスを開発します。サーバはバックグラウンドで動作し、クライアントは CLI を通じてサーバに接続します。クライアントが CLI でコマンドを実行すると、アップロードするファイルが選択されます。ファイルのアップロードが完了したら、サーバから状態を報告するメッセージがクライアントに送られます。


### 機能要件
- サーバは CLI で起動し、バックグラウンドで着信接続を待機します。サーバが停止していると、それはサービスが停止しているという意味です。
- クライアントは TCP を使用してサーバにファイルを送信する必要があります。TCP は UDP よりもオーバーヘッドが大きいですが、確実にファイル全体が送信されることを保証するためです。
- TCP での送信では、比較的安定したパケットサイズを使ってください。TCP プロトコルに基づいて、パケットの上限サイズは 65535 バイト（16 ビット長）である可能性がありますが、通常の最大セグメントサイズは下位ネットワークレベルで 1460 バイトです。すべてのデータが送信されるまで、最大サイズとして 1400 バイトのパケットを使用してください。
- 接続プロトコルは基本的なものです。最初にサーバに送信される 32 バイトは、ファイルのバイト数をサーバに通知します。このプロトコルは最大で 4GB（232バイト）までのファイルに対応しています。
- サーバは、受け取ったデータバイトを常に mp4 ファイルとして解釈します。他のファイル形式はサポートされていませんので、クライアントは送信するファイルが mp4 であることを確認する必要があります。
- サーバは、レスポンスプロトコルを用いて応答します。これは、ステータス情報を含む 16 バイトのメッセージです。

### 非機能要件
- ファイルアップロードシステムは、ファイルが破損することなく完全に送信されることを保証しなければなりません。つまり、ファイル全体が正確に到着するか、プロセスが中断されてファイルが保存されない、どちらかの状態でなければなりません。TCP とデータサイズの設定は、この要件を満たすようにされています。
- ファイルアップロードサーバは、最大で 4TB のデータを保存可能です。この容量制限に達した場合、サービスはそれ以上稼働できません。
- ファイルシステムは、毎秒 20,000 個の 1400 バイトのパケットを処理する能力が必要です。

## ステージ 2
推奨使用言語：Python（サーバとクライアント両方で）

### 内容
このステージでは、ユーザーが動画ファイルをサーバにアップロードし、そのファイルに対して動画処理ができるサービスを開発します。処理が完了した動画ファイルはサーバからユーザーに返されます。サーバとクライアントは、TCP とカスタムアプリケーションレベルプロトコルを使って、さまざまな種類のファイルを互いに送信できるようにします。以下に、このサービスで提供される動画処理機能の一覧があります。

- 動画を圧縮する: ユーザーは動画ファイルをサーバにアップロードし、圧縮されたバージョンをダウンロードできます。サーバが自動的に最適な圧縮を選択します。
- 動画の解像度を変更する: ユーザーが動画をアップロードして希望する解像度を選択すると、その解像度に変更された動画がダウンロードできます。
- 動画のアスペクト比を変更する: ユーザーが動画をアップロードして希望するアスペクト比を選ぶと、そのアスペクト比に変更された動画がダウンロードできます。
- 動画を音声に変換する: 動画ファイルをアップロードすると、その動画から音声だけを抽出した MP3 バージョンがダウンロードできます。
- 指定した時間範囲で GIF や WEBM を作成: ユーザーが動画をアップロードし、時間範囲を指定すると、その部分を切り取って GIF または WEBM フォーマットに変換します。

すべての動画処理は FFMPEG を使って行います。

[FFMPEG](https://ffmpeg.org/about.html) は無料のオープンソースのソフトウェアプロジェクトであり、マルチメディアファイルを扱うためのツールとライブラリ、およびオーディオや動画ファイルを変換・処理するコマンドラインユーティリティを提供しています。さまざまなファイル形式間での変換はもちろん、動画のカット、リサイズ、エフェクト追加など、多くの操作が可能です。このソフトウェアはメディアやエンターテイメント業界で広く採用されており、多様なオペレーティングシステムやプラットフォームで利用できます。

### 機能要件
- クライアントとサーバが通信できるように、カスタムアプリケーションレベルプロトコルを設計します。TCP コネクション は UDP と比べて信頼性が高く、初期のハンドシェイクや余計なオーバーヘッドがありますが、メッセージの受信順序が保証されます。以下のカスタムプロトコルを Multiple Media Protocol（MMP）と呼びます。
  - ヘッダー（64 バイト）: JSONサイズ（16 バイト）、メディアタイプサイズ（1 バイト）、ペイロードサイズ（47 バイト）を含みます。
  - ボディ：最初の JSON サイズバイトは、すべての引数を含む JSON ファイルで、最大で 216バイト（64 KB）です。次のメディアタイプサイズバイトは、メディアタイプ（mp4、mp3、json、avi など）です。メディアタイプは UTF-8 でエンコードされ、1~4 バイトの幅を持つことができます。その後にペイロードが続き、そのサイズはペイロードサイズバイトで、最大 247 バイトです。ペイロードは常にファイルとして格納され、メディアタイプに基づいたファイルタイプを持ち、メディアタイプに従って読み込まれることに注意しましょう。
- サーバは、ペイロードを保存した後で、JSON ファイルを読み込んでリクエストを処理します。
- クライアントとサーバは、データの送受信に同じプロトコルを使用します。
- 何らかのエラーが発生した場合、エラーコード、説明、解決策を含む JSON ファイルが送信されます。この場合、メディアサイズとペイロードサイズは両方とも 0 に設定されます。
- クライアントがサーバからファイルをダウンロードしていない間、定期的に動画ファイルの処理状況を確認します。デフォルトの確認間隔は 1 分です。

### 特徴
- サーバは、すべての動画メディア処理に FFMPEG を使用します。
- このサービスは、FFMPEG が対応している全てのフォーマットをサポートします。詳細は [FFMPEG のフォーマット（muxerとdemuxer）](https://ffmpeg.org/ffmpeg-formats.html)を参照してください。
- 全てのファイルは一時的に保存され、処理完了後にはサーバに保存されたファイルはストレージから削除されます。
- サーバは IP アドレスごとに処理を 1 つに制限します。
- 動画ファイルの圧縮
  - このサービスは動画ファイルをユーザーに代わって圧縮し、自動的に最適な圧縮方法を選びます。サービスは、大きなファイルサイズの削減を実現しながらも、オリジナルに近い画質を維持した動画を返す必要があります。
- 動画の解像度の変更
  -ユーザーが動画をアップロードし、望む解像度を選ぶと、その解像度に変換された動画が返されます。
- 動画のアスペクト比の変更
  - ユーザーが動画をアップロードし、望むアスペクト比を選ぶと、そのアスペクト比に変換された動画が返されます。
- 動画をオーディオに変換
  - ユーザーが動画ファイルをアップロードすると、その動画から音声だけを抽出した MP3 バージョンが返されます。
-時間範囲での GIF と WEBM の作成
  - ユーザーが動画をアップロードし、時間範囲を指定すると、その部分を切り取り、GIF または WEBM 形式に変換して返します。

### 非機能要件
- ファイルアップロードシステムは、ファイルが破損せずに完全に送られることを最優先としなければいけません。ファイルが完全に送信される、もしくはアップロードが中止されて保存されない、という状態のどちらかでなければならない。この要件は TCP とデータサイズの設定によって達成されます。
- ファイルアップロードサーバは、一台のサーバで最大で 4TB のデータを保存できます。全てのユーザーファイルは一時的に保管され、速やかに削除される必要があります。
- 各ファイルシステムサーバは、少なくとも毎秒 5,000 個の 1,400 バイトのパケットを処理する能力が必要です。そして、常時、リソースの 60% 以上を動画処理に割り当てる必要があります。

## ステージ 3（任意・上級者）
ステージ 3は、より高度な機能や特定の要件を実装する際に必要な作業が含まれます。

### デスクトップアプリケーションの作成
推奨される言語は JavaScript または TypeScript で、Electron.js を使用します。Electron.js は Node.js に基づいており、JavaScript コードはブラウザではなくオペレーティングシステム上で実行されます。

このデスクトップアプリケーションは、ユーザーが動画を簡単に圧縮・変換できるように設計されます。アプリにはダッシュボードが備わっており、ユーザーは動画をアップロードして、圧縮するか変換するかを選べます。圧縮を選択した場合、ユーザーは設定を指定し、アップロードする動画ファイルを選んで圧縮を開始します。変換を選んだ場合は、専用の GUI が表示され、ユーザーは動画をアップロードし、出力形式を選んで変換を始めます。変換が進行中のときには、その進捗状況を確認できます。

### メッセージの暗号化
暗号化方式として、RSA に似た手法を使用して、サーバとクライアント間で送受信されるすべてのメッセージを保護します。クライアントとサーバは、送受信するすべてのメッセージを以下の暗号化方式で処理します。


#### クライアントの手順：
- クライアントはローカル環境で秘密鍵と公開鍵を生成します。
- サーバへの接続要求を行う際に、生成した公開鍵をサーバに送信します。
- サーバはこの公開鍵を保存し、この鍵を用いてメッセージを暗号化した後でクライアントに送信します。
- メッセージに含まれるトークンを復号化する唯一の方法は、正確な秘密鍵を使用することです。クライアントはローカルに保存された秘密鍵で復号化を行います。

#### 逆のプロセス：
- すべてのクライアントは、初めにサーバからその公開鍵を取得します。
- クライアントがサーバにメッセージを送る際は、このサーバの公開鍵を使用してメッセージを暗号化する必要があります。

これらがメッセージ暗号化における主要な要件と手順です。

#### 拡張性
拡張性について実装する必要はありませんが、以下の非機能要件について考慮してみてください。

動画システムは、少なくとも毎秒 10,000 パケットの送信をサポートする必要があります。

Netflix、Amazon Video、YouTube などの大規模なサービスと同じ規模になった場合、どうなるでしょうか。これらの動画ダウンロードやストリーミングサービスには数億、あるいは数十億人のユーザーがいます。ピーク時には 4 億人のアクティブユーザーが 2 時間にわたって平均 2GB の映画を視聴またはダウンロードすると仮定します。この場合、サービスが 1 秒あたりに処理する必要があるパケット数を計算してみてください。

ピーク時にこれだけのユーザー数をサポートする場合、動画編集の処理をどのように対応するか考えてみましょう。

#### ロードバランシング
ロードバランサーが存在し、それぞれが同じタスクを実行できる n 台のサーバがあるとします。ロードバランサーがネットワークトラフィックを複数のサーバに効率よく振り分ける場合、1 台のサーバが 1 秒間に 10k パケットを処理できると仮定すると、1 秒間に処理されるソケットの総数はどれくらいになるでしょうか？

#### 同時処理
単一のコアで毎秒 10k のソケットを処理できる場合、16 個のコアでプログラムを実行すると処理能力はどう変わるでしょうか？

#### 水平スケーリング
たとえば、ユーザーが 10 年以上もの長期間ファイルを保存できるように、無制限のストレージが必要だとします。1 台のハードドライブには保存できるデータ量に上限がありますが、ハードディスクを何台でも追加することができるでしょうか？例えば、10TB のハードディスクを 2 万台追加すると何が起こるか考えてみましょう。これは一つの解決策ですが、何千台もの異なるハードディスクにデータを安全に保存するには、自動化と品質管理も必要です。10 年という長い期間で考えると、ハードディスクが突然故障する可能性もあります。


#### 分散プログラミング
特定のタスクが小さなサブタスクに分割され、それが複数のコアを持つ複数のマシンに分散される場合、1 秒間に処理できるソケットの総量はどれくらいになるでしょうか？また、動画処理のワークロードを維持するためにはどのような処理能力が必要でしょうか？
