# Yauyih A6 Modbus Tool Web Edition

https://kikiqqp.github.io/MINAS-A6-Modbus-Tool-Web-Edition/

**English** | [繁體中文](#繁體中文) | [日本語](#日本語)

***

## English

A browser based Modbus RTU test and commissioning tool for Panasonic MINAS A6 series servo drives. The whole tool is one HTML file. There is nothing to install and no server to run: open it in Chrome or Edge, authorize a serial port, and the page talks to the drive directly through the Web Serial API.

It is built from the Panasonic Technical Reference "Modbus communication and Block operation Specification" (SX-DSV03042, for the MINAS A6 / A6L series) and keeps the window layout of an earlier Windows Forms (C#) version, so people who used that version can find everything in the same place.

### Features

* **One file, no install.** Open `A6.html` locally, or host it on any HTTPS site such as GitHub Pages.
* **Live monitoring.** 16 poll groups in three speed tiers (100 ms, 500 ms and 2000 ms) cover status, I/O, encoder, external scale, load rates, temperatures, counters and degradation diagnosis. The screen refreshes every 100 ms and redraws only the fields that changed.
* **Status pages.** Error code with the alarm name, motor type and serial number, warning flags, speeds, positions, deviations, encoder and external scale data, physical, logical and virtual I/O lamps, status flags and Block status lamps.
* **Block motion parameters.** Reads and writes 4600h to 463Ch (velocity, acceleration and deceleration for 16 sets, home offset, position limits, homing speeds and acceleration, the homing not required setting, time units and the Pr60.48 operation mode). The write keeps Pr60.48 and reads everything back to verify.
* **Block motion planning.** Edits all 256 Blocks (command and data, decimal and hexadecimal), shows a plain language description of every command, highlights changed rows, and checks the plan before writing.
* **Trial operation.** Servo ON and OFF, JOG positive and negative, decelerating stop (`S-STOP`), immediate stop (`H-STOP`) and Block RUN for any Block number from 0 to 255.
* **Maintenance actions.** Write to EEPROM, clear multiturn data and clear alarms. Writing to EEPROM and clearing multiturn data ask for confirmation first, and the clear functions run only when the documented value is entered.
* **Optional communication right protection.** Acquires the right (4300h) together with the Pr5.40 timeout, so the drive stops by itself if communication is lost.
* **Communication log.** Timestamped hexadecimal frames with response times, retries and errors. You can show or hide poll packets, auto scroll, copy and clear.
* **Four interface languages.** Traditional Chinese, Simplified Chinese, English and Japanese. The language follows your system by default and can be switched instantly, including messages, the log and pop up dialogs.
* **Firmware aware.** On connection the tool logs the drive model and software version. If the drive does not support Pr60.56 and Pr60.57 (function enhancement version 8 or later is required), it falls back to 59 registers and disables the unit selectors automatically.

### Requirements

* A desktop browser that provides the Web Serial API, for example Chrome or Edge. Safari does not support it. The tool was developed and tested with Chromium.
* If you open it from a web address, the address must be HTTPS or localhost. Opening the file directly from disk also works.
* A serial connection to the drive: an RS232 or RS485 USB adapter wired to the drive's communication connector, as described in the Panasonic manual.
* The COM port must not be in use by anything else (a C# tool, PANATERM, a terminal program or another browser tab).

### Drive settings (set these first)

These settings follow manual SX-DSV03042. The values on the screen must match the drive.

* **Pr5.31** Slave address, 1 to 127. It cannot be 0.
* **Pr5.37** Set to 1 for RS232 or 2 for RS485.
* **Pr5.29 or Pr5.30** Baud rate.
* **Pr5.38** Parity and stop bits. The factory setting is even parity and 1 stop bit.
* **Pr6.28** Set to 1 so that Blocks can be started over Modbus. Power the drive off and on again after changing it.

### Quick start

1. Open `A6.html` in Chrome or Edge and read the start up warning.
2. Click `Select Port` and authorize your serial port in the browser dialog.
3. Set the baud rate, data bits, parity, stop bits and slave address to match the drive.
4. Click `Open Port`. The status bar changes to On Line and the monitor values start updating.
5. Use the tabs: Status Page, BLOCK Motion Parameters, BLOCK Motion Planning and Status Page2.
6. To close the connection, click `Close Port` and wait a few seconds while the tool stops any motion and restores trial data.

### Screens

* **Status Page** and **Status Page2** show live values and the I/O lamps.
* **BLOCK Motion Parameters** edits the shared parameters of the Block function. Click `Read` before `Write`. `EEP` stores the values in the drive's EEPROM after confirmation.
* **BLOCK Motion Planning** edits the 256 Block commands. `Check` looks for Blocks that refer to a velocity or acceleration set whose value is 0, which would cause Err93.1.

### Safety design

This tool can move a motor and change drive parameters. Read this section before connecting to real equipment.

* Trial operation backs up Block 254, Block 255 and the parameter set 15, and restores them afterwards. JOG writes only three registers.
* JOG stops when the button is released, the window loses focus, the tab is hidden or the connection drops. The stop sequence confirms that the motor is below 30 r/min, and escalates to an immediate stop (`H-STOP`) if that fails or times out.
* The optional communication right protection makes the drive stop by itself (Err80.0) when communication is lost for longer than the timeout. While the right is held, the front panel works only in monitor mode.
* Every write is read back and verified. If reading the Block plan fails, the tool never shows zeros on the screen, so you cannot overwrite all Blocks by accident.
* Writing to EEPROM and clearing multiturn data ask for confirmation. Clearing multiturn data requires Servo OFF and a power cycle afterwards. The clear functions run only when the documented value is entered.
* The stop buttons can be pressed at any time, even while another operation is running.
* Communication errors are written to the log instead of popping up a dialog every time.

Always keep a hardware emergency stop within reach and secure the mechanism before testing.

### How it works

* A single communication loop owns the serial link. Requests go through a priority queue: urgent commands (stops) first, then user operations, then idle time polling. Transactions never overlap.
* A resident reader loop fills a byte buffer. Frames are parsed by length and CRC16. Exception responses (5 bytes) are reported immediately instead of waiting for a timeout.
* Timeouts use real time. Supported function codes are 03 (read registers), 05 (write coil, used for Servo ON), 06 (write register) and 10 (write registers).
* A poll group that is rejected with an exception three times in a row is disabled, so one unsupported register range does not slow down everything else.
* The source inside the HTML file is organized in parts: header notes, the language engine and dictionary, core (constants, CRC16, command classes), communication layer, A6 data layer (decoders, alarm table, Block decoding), operations (Block read and write, trial operation, stop logic) and the user interface.

### Languages

The interface supports Traditional Chinese (zh-TW), Simplified Chinese (zh-CN), English (en) and Japanese (ja). The initial language is your last manual choice, then the browser language, then English. Code comments are written in Traditional Chinese. To add text, put the original sentence in the dictionary with all four translations and show it with the translation helpers. Names of buttons and signal codes (SI1, SRV-ON, S-RDY and so on) are intentionally not translated.

### Testing and verification status

During development the tool was exercised with automated tests against a simulated A6 drive, in a simulated browser environment (jsdom) and in a real Chromium browser. These tests cover the CRC16 example from the manual, the supported function codes, exception handling, Block read, write and read back, the firmware fallback, trial operation, language switching and the layout in all four languages. A simulation cannot replace real hardware. Behavior depends on your drive model, firmware version and wiring, so please verify on your own equipment first and start with a motor that is not connected to a load.

### Disclaimer

The software is provided as is, without warranty of any kind. You are responsible for the safety of your machine and for any damage or injury caused by its use.

### License and trademarks

The source code is released under the MIT License. See `LICENSE` for the full text (the English text is the only legally binding version) and `NOTICE.md` for the trademark and third party notices in detail.

* **MINAS and Panasonic belong to Panasonic.** Panasonic, MINAS, PANATERM and related product names and logos are trademarks or registered trademarks of Panasonic Holdings Corporation, Panasonic Industry Co., Ltd. or other Panasonic group companies. This is an independent project. It is not made, approved, sponsored or supported by Panasonic. The names are used only to say which products the tool works with.
* **The Panasonic manual is not included.** The Technical Reference (SX-DSV03042) is copyrighted by Panasonic Industry Co., Ltd. and is not part of this repository. Please obtain it from Panasonic.
* **Other names.** Modbus, Windows, Chrome, Edge, Safari, GitHub and other names mentioned here belong to their respective owners.
* **Yauyih name and logo.** The MIT License covers the code. It does not give you the right to use the Yauyih name or logo as your own brand, or to suggest that your version is approved by the author.

***

## 繁體中文

這是一套在瀏覽器裡執行的松下 MINAS A6 系列伺服驅動器 Modbus RTU 測試與調機工具。整個工具只有一個 HTML 檔，不用安裝，也不用架設伺服器：用 Chrome 或 Edge 開啟，授權序列埠，網頁就會透過 Web Serial API 直接和驅動器通訊。

本工具依據松下技術參考文件「Modbus communication and Block operation Specification」（SX-DSV03042，適用 MINAS A6 / A6L 系列）製作，並沿用先前 Windows Forms（C#）版本的視窗配置，用過舊版的人可以在原來的位置找到所有功能。

### 功能特色

* **單一檔案，免安裝。** 直接在本機開啟 `A6.html`，也可以放到任何 HTTPS 網站，例如 GitHub Pages。
* **即時監控。** 16 個輪詢群組分成快、中、慢三級（100 毫秒、500 毫秒、2000 毫秒），涵蓋狀態、輸入輸出、編碼器、外部尺、負載率、溫度、計數與劣化診斷。畫面每 100 毫秒更新一次，只重繪有變動的欄位。
* **狀態頁面。** 錯誤碼與警報名稱、馬達型號與序號、警告旗標、速度、位置、偏差、編碼器與外部尺資料、實體、邏輯與虛擬輸入輸出燈號、狀態旗標與 Block 狀態燈號。
* **Block 動作參數。** 讀寫 4600h 到 463Ch（16 組速度、加速與減速時間，原點偏移、位置限位、原點復歸速度與加減速、免原點復歸設定、時間單位與 Pr60.48 運轉方式）。寫入時保留 Pr60.48，並全部讀回驗證。
* **Block 動作規劃。** 編輯全部 256 個 Block（命令與資料，十進位與十六進位），為每一筆命令顯示白話說明，變更的列會標示出來，寫入前先檢查規劃內容。
* **試運轉。** Servo ON 與 OFF、JOG 正向與負向、減速停止（`S-STOP`）、立即停止（`H-STOP`），以及 0 到 255 任一 Block 的 Block RUN。
* **維護動作。** 寫入 EEPROM、清除多圈資料、清除警報。寫入 EEPROM 與清除多圈資料會先要求確認，清除功能必須輸入手冊規定的數值才會執行。
* **選用的通訊權保護。** 取得通訊權（4300h）並搭配 Pr5.40 逾時，通訊中斷時驅動器會自行停止。
* **通訊記錄。** 帶時間戳記的十六進位封包、回應時間、重試與錯誤。可以顯示或隱藏輪詢封包、自動捲動、複製與清除。
* **四種介面語言。** 繁體中文、簡體中文、English、日本語。預設跟隨系統語言，可以即時切換，訊息、記錄與彈跳視窗都會一起換。
* **自動辨識韌體。** 連線時在記錄中顯示驅動器型號與軟體版本。若驅動器不支援 Pr60.56 與 Pr60.57（需要功能增強版 8 以上），會自動改為讀寫 59 個暫存器並停用單位選單。

### 使用需求

* 提供 Web Serial API 的桌面瀏覽器，例如 Chrome 或 Edge。Safari 不支援。本工具以 Chromium 開發與測試。
* 若用網址開啟，網址必須是 HTTPS 或 localhost。直接從磁碟開啟檔案也可以使用。
* 與驅動器之間的序列連線：依松下手冊接線的 RS232 或 RS485 USB 轉換器。
* COM 埠不能同時被其他程式占用（C# 版程式、PANATERM、序列埠終端機、其他瀏覽器分頁）。

### 驅動器設定（請先設定）

以下設定依照手冊 SX-DSV03042，畫面上的設定必須與驅動器一致。

* **Pr5.31** 站號，1 到 127，不可為 0。
* **Pr5.37** 設為 1 代表 RS232，設為 2 代表 RS485。
* **Pr5.29 或 Pr5.30** 傳輸速率。
* **Pr5.38** 同位檢查與停止位元，出廠設定為偶同位、1 個停止位元。
* **Pr6.28** 設為 1，才能透過 Modbus 啟動 Block 運轉。變更後需要重新送電。

### 快速上手

1. 用 Chrome 或 Edge 開啟 `A6.html`，閱讀開場警告。
2. 按 `Select Port`，在瀏覽器對話框中授權序列埠。
3. 設定傳輸速率、資料位元、同位檢查、停止位元與站號，必須與驅動器一致。
4. 按 `Open Port`。狀態列會變成 On Line，監控數值開始更新。
5. 使用頁籤：Status Page、BLOCK Motion Parameters、BLOCK Motion Planning、Status Page2。
6. 要結束時按 `Close Port`，等待數秒，工具會先停止動作並還原試運轉資料。

### 畫面說明

* **Status Page** 與 **Status Page2** 顯示即時數值與輸入輸出燈號。
* **BLOCK Motion Parameters** 編輯 Block 功能共用的參數。寫入前請先按 `Read`。`EEP` 在確認後把數值存入驅動器的 EEPROM。
* **BLOCK Motion Planning** 編輯 256 個 Block 命令。`Check` 會檢查是否有 Block 引用到數值為 0 的速度或加減速組，這會造成 Err93.1。

### 安全設計

本工具可以驅動馬達並修改驅動器參數。連接真實設備前請先閱讀這一節。

* 試運轉會備份 Block 254、Block 255 與第 15 組參數，結束後還原。JOG 只寫入三個暫存器。
* 放開按鈕、視窗失去焦點、分頁被隱藏或連線中斷時，JOG 都會停止。停止流程會確認馬達低於 30 r/min，若失敗或逾時就升級為立即停止（`H-STOP`）。
* 選用的通訊權保護會讓驅動器在通訊中斷超過逾時時間後自行停止（Err80.0）。持有通訊權期間，驅動器前面板只能使用監視模式。
* 每次寫入都會讀回驗證。讀取 Block 規劃失敗時，畫面不會顯示全 0，避免誤按寫入而覆蓋全部 Block。
* 寫入 EEPROM 與清除多圈資料會先要求確認。清除多圈資料必須先關閉 Servo，事後還要重新送電。清除功能必須輸入手冊規定的數值才會執行。
* 停止按鈕任何時候都能按，即使其他操作正在進行中也一樣。
* 通訊錯誤只寫入記錄，不會每次都彈出對話框。

測試時請讓硬體緊急停止開關隨手可及，並先固定好機構。

### 運作原理

* 單一通訊迴圈掌管序列埠。請求經過優先權佇列：緊急命令（停止）優先，其次是使用者操作，最後才是閒置時的輪詢。交易絕不會重疊。
* 常駐的讀取迴圈把資料填入位元組緩衝區，再依長度與 CRC16 解析封包。例外回應（5 個位元組）會立即回報，不必等到逾時。
* 逾時以真實時間計算。支援的功能碼為 03（讀取暫存器）、05（寫入線圈，用於 Servo ON）、06（寫入暫存器）與 10（寫入多個暫存器）。
* 輪詢群組連續三次被驅動器以例外拒絕就會停用，避免一個不支援的暫存器範圍拖慢其他項目。
* HTML 檔內的程式依序分成：檔頭說明、語系引擎與字典、核心層（常數、CRC16、命令類別）、通訊層、A6 資料層（解碼、警報表、Block 解碼）、操作層（Block 讀寫、試運轉、停止邏輯）與畫面層。

### 多語系

介面支援繁體中文（zh-TW）、簡體中文（zh-CN）、English（en）與日本語（ja）。啟動時的語言順序是：使用者上次手動選的語言、瀏覽器語言、英文。程式註解一律使用繁體中文。要新增文字時，把原句連同四種翻譯放進字典，再用翻譯輔助函式顯示。按鈕名稱與訊號代號（SI1、SRV-ON、S-RDY 等）刻意不翻譯。

### 測試與驗證狀態

開發過程中，以模擬的 A6 驅動器，搭配模擬的瀏覽器環境（jsdom）與真實的 Chromium 瀏覽器執行自動化測試，涵蓋手冊中的 CRC16 範例、支援的功能碼、例外處理、Block 讀取、寫入與讀回、韌體相容、試運轉、語言切換，以及四種語言的版面。模擬無法取代真實硬體，實際行為會因驅動器型號、韌體版本與接線而不同，請先在自己的設備上驗證，並從未連接負載的馬達開始。

### 免責聲明

本軟體依現狀提供，不附帶任何形式的保證。機械的安全，以及使用本工具造成的任何損壞或傷害，由使用者自行負責。

### 授權與商標

原始碼以 MIT 授權釋出，完整條文請見 `LICENSE`（僅英文版本具有法律效力），商標與第三方聲明的詳細內容請見 `NOTICE.md`。

* **MINAS 與 Panasonic 屬於松下。** Panasonic、MINAS、PANATERM 及相關產品名稱與標誌，是 Panasonic Holdings Corporation、Panasonic Industry Co., Ltd. 或松下集團其他公司的商標或註冊商標。本專案為獨立專案，並非由松下製作、認可、贊助或提供支援，文中使用這些名稱只是為了說明本工具適用於哪些產品。
* **不附松下手冊。** 技術參考文件（SX-DSV03042）的著作權屬於 Panasonic Industry Co., Ltd.，不包含在此儲存庫中，請向松下取得。
* **其他名稱。** 文中提到的 Modbus、Windows、Chrome、Edge、Safari、GitHub 等名稱，屬於其各自的所有人。
* **Yauyih 名稱與 LOGO。** MIT 授權涵蓋的是程式碼，並不授予您把 Yauyih 名稱或 LOGO 當成自己的品牌使用，或暗示您的版本經作者認可的權利。

***

## 日本語

Panasonic MINAS A6 シリーズのサーボドライバを、ブラウザだけで Modbus RTU 通信して試験、調整するためのツールです。ツール全体が 1 つの HTML ファイルで、インストールもサーバーも要りません。Chrome または Edge で開いてシリアルポートを許可すると、Web Serial API を通じてドライバと直接通信します。

Panasonic の技術資料「Modbus communication and Block operation Specification」（SX-DSV03042、MINAS A6 / A6L シリーズ対象）に基づいて作られており、以前の Windows Forms（C#）版と同じ画面レイアウトを引き継いでいるため、旧版を使っていた方は同じ場所で同じ機能を見つけられます。

### 主な機能

* **1 ファイル、インストール不要。** `A6.html` をローカルで開くだけです。GitHub Pages など HTTPS のサイトに置いても使えます。
* **リアルタイム監視。** 16 個のポーリンググループを 3 段階の周期（100 ms、500 ms、2000 ms）で読み出し、ステータス、入出力、エンコーダ、外部スケール、負荷率、温度、カウンタ、劣化診断をカバーします。画面は 100 ms ごとに更新し、変化した項目だけを描き直します。
* **ステータスページ。** エラーコードとアラーム名、モータ型式とシリアル番号、警告フラグ、速度、位置、偏差、エンコーダと外部スケールのデータ、物理、論理、仮想の入出力ランプ、ステータスフラグ、Block ステータスランプ。
* **Block 動作パラメータ。** 4600h から 463Ch（16 組の速度、加速時間、減速時間、原点オフセット、位置リミット、原点復帰の速度と加減速、原点復帰不要の設定、時間単位、Pr60.48 の運転方式）を読み書きします。書き込み時は Pr60.48 を保持し、すべて読み戻して検証します。
* **Block 動作プラン。** 256 個すべての Block（コマンドとデータ、10 進と 16 進）を編集できます。各コマンドの内容をわかりやすい文章で表示し、変更した行を強調し、書き込み前にプランをチェックします。
* **試運転。** Servo ON と OFF、JOG 正方向と負方向、減速停止（`S-STOP`）、即時停止（`H-STOP`）、0 から 255 の任意の Block を起動する Block RUN。
* **メンテナンス操作。** EEPROM への書き込み、マルチターンデータのクリア、アラームのクリア。EEPROM への書き込みとマルチターンデータのクリアは先に確認が求められ、クリア機能はマニュアルで定められた値を入力したときだけ実行されます。
* **通信権保護（任意）。** 通信権（4300h）を取得し、Pr5.40 のタイムアウトと組み合わせることで、通信が途絶えるとドライバが自分で停止します。
* **通信ログ。** タイムスタンプ付きの 16 進フレーム、応答時間、リトライ、エラーを記録します。ポーリングパケットの表示切り替え、自動スクロール、コピー、クリアに対応します。
* **4 つの表示言語。** 繁体字中国語、簡体字中国語、English、日本語。既定ではシステム言語に従い、メッセージ、ログ、ポップアップを含めて即座に切り替えられます。
* **ファームウェア自動判別。** 接続時にドライバの型式とソフトウェアバージョンをログに表示します。ドライバが Pr60.56 と Pr60.57 に対応していない場合（機能拡張バージョン 8 以降が必要）は、自動的に 59 個のレジスタを読み書きする方式に切り替え、単位の選択を無効にします。

### 動作環境

* Web Serial API を備えたデスクトップブラウザ（Chrome、Edge など）。Safari は非対応です。本ツールは Chromium で開発と試験を行いました。
* Web アドレスから開く場合は HTTPS または localhost が必要です。ディスク上のファイルを直接開いても使えます。
* ドライバとのシリアル接続：Panasonic のマニュアルに従って配線した RS232 または RS485 の USB 変換器。
* COM ポートを他のプログラム（C# 版ツール、PANATERM、ターミナルソフト、別のブラウザタブ）が使用していないこと。

### ドライバ側の設定（最初に行ってください）

以下はマニュアル SX-DSV03042 に基づく設定です。画面側の設定はドライバと一致させる必要があります。

* **Pr5.31** 局番。1 から 127 で、0 にはできません。
* **Pr5.37** 1 で RS232、2 で RS485。
* **Pr5.29 または Pr5.30** ボーレート。
* **Pr5.38** パリティとストップビット。出荷時は偶数パリティ、ストップビット 1 です。
* **Pr6.28** 1 に設定すると、Modbus から Block 運転を開始できます。変更後は電源を入れ直してください。

### クイックスタート

1. Chrome または Edge で `A6.html` を開き、起動時の警告を読みます。
2. `Select Port` を押し、ブラウザのダイアログでシリアルポートを許可します。
3. ボーレート、データビット、パリティ、ストップビット、局番をドライバに合わせて設定します。
4. `Open Port` を押します。ステータスバーが On Line になり、監視値の更新が始まります。
5. タブを切り替えて使います：Status Page、BLOCK Motion Parameters、BLOCK Motion Planning、Status Page2。
6. 終了するときは `Close Port` を押し、数秒待ちます。動作の停止と試運転データの復元が行われます。

### 画面について

* **Status Page** と **Status Page2** はリアルタイムの値と入出力ランプを表示します。
* **BLOCK Motion Parameters** は Block 機能で共通のパラメータを編集します。書き込みの前に必ず `Read` を押してください。`EEP` は確認のあと、値をドライバの EEPROM に保存します。
* **BLOCK Motion Planning** は 256 個の Block コマンドを編集します。`Check` は、値が 0 の速度または加減速の組を参照している Block を探します。これは Err93.1 の原因になります。

### 安全設計

このツールはモータを動かし、ドライバのパラメータを変更できます。実機につなぐ前に、この項目を必ずお読みください。

* 試運転では Block 254、Block 255、パラメータ第 15 組をバックアップし、終了後に復元します。JOG が書き込むのは 3 つのレジスタだけです。
* ボタンを離したとき、ウィンドウがフォーカスを失ったとき、タブが非表示になったとき、接続が切れたときに JOG は停止します。停止処理ではモータが 30 r/min 未満になったことを確認し、失敗またはタイムアウトした場合は即時停止（`H-STOP`）に切り替えます。
* 任意の通信権保護を使うと、通信がタイムアウト時間を超えて途絶えたときにドライバが自ら停止します（Err80.0）。通信権を保持している間、ドライバ前面パネルはモニタモードのみ操作できます。
* すべての書き込みは読み戻して検証します。Block プランの読み出しに失敗したときは、画面に 0 を表示しません。誤って書き込んですべての Block を上書きすることを防ぐためです。
* EEPROM への書き込みとマルチターンデータのクリアは確認が求められます。マルチターンデータのクリアは Servo OFF の状態で行い、その後に電源の入れ直しが必要です。クリア機能はマニュアルで定められた値を入力したときだけ実行されます。
* 停止ボタンは、他の操作の実行中でもいつでも押せます。
* 通信エラーはログに記録するだけで、そのたびにダイアログを出すことはしません。

試験中はハードウェアの非常停止スイッチをすぐ押せる場所に置き、機構を固定してから行ってください。

### 仕組み

* 1 本の通信ループがシリアルポートを管理します。要求は優先度付きキューを通り、緊急コマンド（停止）が最優先、次にユーザー操作、最後にアイドル時のポーリングです。トランザクションが重なることはありません。
* 常駐の読み取りループがバイトバッファを満たし、長さと CRC16 でフレームを解析します。例外応答（5 バイト）はタイムアウトを待たずにすぐ報告されます。
* タイムアウトは実時間で計測します。対応するファンクションコードは 03（レジスタ読み出し）、05（コイル書き込み、Servo ON に使用）、06（レジスタ書き込み）、10（複数レジスタ書き込み）です。
* ポーリンググループが 3 回続けて例外で拒否されると無効になり、非対応のレジスタ範囲が他の項目を遅くすることを防ぎます。
* HTML 内のコードは、ヘッダの説明、言語エンジンと辞書、コア層（定数、CRC16、コマンドクラス）、通信層、A6 データ層（デコード、アラーム表、Block のデコード）、操作層（Block の読み書き、試運転、停止ロジック）、画面層の順に構成されています。

### 多言語対応

表示言語は繁体字中国語（zh-TW）、簡体字中国語（zh-CN）、English（en）、日本語（ja）です。起動時の言語は、ユーザーが最後に選んだ言語、ブラウザの言語、英語の順に決まります。コードのコメントは繁体字中国語で書かれています。文言を追加するときは、原文と 4 言語の訳を辞書に登録し、翻訳用のヘルパー関数で表示します。ボタン名と信号コード（SI1、SRV-ON、S-RDY など）は意図的に翻訳していません。

### 試験と検証の状況

開発中は、模擬の A6 ドライバを使い、シミュレートしたブラウザ環境（jsdom）と実際の Chromium ブラウザで自動テストを行いました。対象は、マニュアルにある CRC16 の例、対応ファンクションコード、例外処理、Block の読み出し、書き込み、読み戻し、ファームウェア互換、試運転、言語切り替え、4 言語での画面レイアウトです。シミュレーションは実機の代わりにはなりません。実際の動作はドライバの型式、ファームウェアのバージョン、配線によって異なりますので、必ずご自身の設備で確認し、負荷を接続していないモータから始めてください。

### 免責事項

本ソフトウェアは現状のまま提供され、いかなる保証もありません。機械の安全、および本ツールの使用によって生じた損害や傷害については、使用者が責任を負います。

### ライセンスと商標

ソースコードは MIT ライセンスで公開しています。全文は `LICENSE` をご覧ください（法的な効力を持つのは英語版のみです）。商標とサードパーティに関する詳しい表記は `NOTICE.md` にあります。

* **MINAS と Panasonic は Panasonic のものです。** Panasonic、MINAS、PANATERM および関連する製品名とロゴは、Panasonic Holdings Corporation、Panasonic Industry Co., Ltd. またはパナソニックグループ各社の商標または登録商標です。本プロジェクトは独立したものであり、Panasonic が製作、承認、後援、サポートしているものではありません。これらの名称は、本ツールが対応する製品を示すためだけに使っています。
* **Panasonic のマニュアルは含まれません。** 技術資料（SX-DSV03042）の著作権は Panasonic Industry Co., Ltd. にあり、このリポジトリには含まれていません。Panasonic から入手してください。
* **その他の名称。** ここに登場する Modbus、Windows、Chrome、Edge、Safari、GitHub などの名称は、それぞれの所有者のものです。
* **Yauyih の名称とロゴ。** MIT ライセンスが対象とするのはコードです。Yauyih の名称やロゴを自分のブランドとして使うこと、または自分のバージョンが作者に承認されているかのように示すことを許可するものではありません。
