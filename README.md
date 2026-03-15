No	パッケージ名	パッケージ名 (英字名)	パッケージ概要	パッケージ種別	パス	備考
1	ストリームリット	streamlit	Python で Web アプリケーションを作成するフレームワーク	標準パッケージ	-	-
2	パンダス	pandas	Python でデータ加工を行うための代表的なライブラリ	標準パッケージ	-	-
3	バリデーションチェック部品	checks	データフレームの項目別バリデーションを実行する共通部品	共通部品	common	内部ステージに格納
4	メッセージ取得・表示部品	message_util	メッセージ ID から多言語メッセージを取得・画面表示する共通部品	共通部品	common	内部ステージに格納
5	Excel ユーティリティ	excel_file_util	Excel ファイルを DataFrame に読み込む共通部品	共通部品	common	内部ステージに格納
6	バリデーションルール定義	validation_rules	各補正対象テーブルのバリデーションルールを定義した設定ファイル	共通部品	common	内部ステージに格納
7	アップロードサービス	hosei_upload_service	ファイル整合性チェック・DB 登録処理を実装したサービス部品	サービス部品	pages	-
8	標準ライブラリ（日付）	datetime	日付・時間操作を行う標準ライブラリ	標準パッケージ	-	-
9	標準ライブラリ（ファイル IO）	io	ファイル入出力操作を行う標準ライブラリ	標準パッケージ	-	-
10	標準ライブラリ（正規表現）	re	正規表現操作を行う標準ライブラリ	標準パッケージ	-	-



No	項目名（ラベル）	物理名	仕様エリア	UI コンポーネント	備考
1	画面名	-	ヘッダーエリア	st.subheader	「法人 DM 基礎データ補正（アップロード）」を表示
2	補正対象テーブル	selected_table_en	アップロードエリア	st.selectbox	テーブル選択肢を事前定義（table_options）
3	補正事由	correction_reason	アップロードエリア	st.text_input	必須入力項目
4	Browse files	upload	アップロードエリア	st.file_uploader	xlsx/xls 形式のみ受け付け
5	メッセージ	messages	メッセージエリア	st.success/st.error	mu.show_messages () で表示
6	エラー一覧	error_display	エラー一覧エリア	st.markdown	CSS でスタイル定義（最大 10 件表示）
7	チェック結果ダウンロード	download_button	ボタンエリア	st.download_button	エラー存在時のみ有効
8	補正データの登録	submit_button	ボタンエリア	st.button	ファイル選択 + 補正事由入力 + エラーなしの場合有効
9	アップロードしますか？	confirm_dialog	確認ダイアログ	@st.dialog	登録前確認用
10	はい	confirm_yes	確認ダイアログ	st.button	登録処理実行用
11	いいえ	confirm_no	確認ダイアログ	st.button	ダイアログ閉じ用




No	session_state 項目名	session_state 項目名 (英字)	種類	デフォルト値	設定条件	開始画面	終了画面	備考
1	画面：セッション	session	Snowpark セッション	None	初期処理で取得	法人 DM 基礎データ補正画面	同左	必要時に DB 接続用
2	画面：エラー表示フラグ	hosei_upload["show_error"]	Boolean	False	初期化 / チェック処理後設定	同左	同左	エラー一覧表示制御用
3	画面：ログ表示フラグ	hosei_upload["showlog"]	Boolean	False	初期化 / 実行ボタン押下後設定	同左	同左	確認ダイアログ表示制御用
4	画面：バリデーションエラーリスト	hosei_upload["validation_errors"]	List	[]	初期化 / チェック処理後設定	同左	同左	全エラーを保持（ダウンロード用）
5	画面：表示用エラーリスト	hosei_upload["display_errors"]	List	[]	初期化 / チェック処理後設定	同左	同左	最大 10 件まで画面表示
6	画面：検証済みデータ	hosei_upload["valid_data"]	pd.DataFrame	None	初期化 / チェック処理成功後設定	同左	同左	DB 登録用データ
7	画面：基準年月	hosei_upload["kijundate"]	String	現在年月（% Y% m）	初期化	同左	同左	画面初期表示用
8	画面：ユーザー ID	ss.user_id	String	"UNKNOWN"	セッションから取得	同左	同左	DB 登録時の更新者用
9	画面：ユーザー名	ss.user_name	String	"匿名"	セッションから取得	同左	同左	画面表示用
10	画面：店舗番号	ss.shop_no	String	"0000"	セッションから取得	同左	同左	DB 登録用









5. イベント詳細（初期処理）
イベント概要
画面初期表示時にセッション状態を初期化し、共通ヘッダーと UI コンポーネントを描画する。
処理詳細
セッション状態初期化
hosei_upload 辞書を初期化：show_error=False, showlog=False, validation_errors=[], valid_data=None, kijundate=現在年月, display_errors=[]
ユーザー情報（user_id, user_name, shop_no）をセッションから取得（必要時）
共通ヘッダー描画
ui_common_header.draw_header("法人DM 基礎データ補正（アップロード）") を呼び出し、画面ヘッダーを表示
UI コンポーネント描画
補正対象テーブル選択用 st.selectbox を描画（選択肢：table_options）
補正事由入力用 st.text_input を描画
ファイルアップロード用 st.file_uploader を描画
エラー一覧表示用コンテナを準備（CSS でスタイル定義）
ボタンエリアを描画（チェック結果ダウンロード、補正データの登録）
📌 6. イベント詳細（ファイル選択後）
イベント概要
Browse files ボタンでファイル選択後、自動でチェック処理を実行し、結果を画面に表示する。
処理詳細
ファイル選択検知
upload is not None の場合、セッション状態をリセット：show_error=False, validation_errors=[], display_errors=[], valid_data=None
前置チェック
補正対象テーブル未選択 → mu.push_messages("456ERR3053","対象テーブル")
補正事由未入力 → mu.push_messages("456ERR3054","補正事由")
ファイル読み込み
excel_file_util.read_wb_as_df(upload) で Excel を pd.DataFrame に変換
読み込み失敗 → mu.push_messages("456ERR0064"), show_error=True
ファイル整合性チェック
SHORI_KBN 列不存在 → mu.push_messages("456ERR3056","補正事由"), validation_errors.append("SHORI_KBNエラー")
列数不一致 → mu.push_messages("456ERR3057"), validation_errors.append("列数エラー")
データバリデーション
validation_rules から対象テーブルのルールを取得し、checks.validate_dataframe(df, rules) を実行 → 結果を validation_errors に格納
チェック結果処理
エラーなし → valid_data=df, mu.push_messages("456INF0012"), show_error=False
エラーあり → show_error=True, display_errors=validation_errors[:10]（超過時は「合計 X 件のエラー」を追加）, mu.push_messages("456ERR4058")
結果表示
mu.show_messages() でメッセージを表示
show_error=True 且つ display_errors 有り → エラー一覧を st.markdown で表示（CSS スタイル適用）
📌 7. イベント詳細（「補正データの登録」ボタン押下）
イベント概要
「補正データの登録」ボタン押下後、確認ダイアログを表示し、DB 登録処理を実行する。
処理詳細
確認ダイアログ表示
@st.dialog で「アップロードしますか？」を表示
「はい」「いいえ」ボタンを配置
「はい」押下時
補正事由未入力 → mu.push_messages("456ERR3054","補正事由")
ユーザー情報収集：user_id, user_name, shop_no
service.insert_hosei_management_data() を呼び出し、DB にデータ登録
登録成功 → st.success("補正データの登録が完了しました。精査者へ精査依頼をしてください。"), valid_data=None, validation_errors=[], correction_reason="", st.rerun()
登録失敗 → st.error(f"登録失敗：{msg}"), showlog=False, st.rerun()
「いいえ」押下時
showlog=False, st.rerun() でダイアログを閉じる
📌 8. イベント詳細（「チェック結果ダウンロード」ボタン押下）
イベント概要
「チェック結果ダウンロード」ボタン押下後、バリデーションエラーを Excel ファイルとしてダウンロードする。
処理詳細
エラー Excel 生成
service.create_error_excel(validation_errors) を呼び出し、エラーリストを Excel に変換
ファイルダウンロード
st.download_button で Excel をダウンロード（ファイル名：データチェックエラー_YYYYMMDDHHMMSS.xlsx）
ボタン制御
validation_errors が空の場合、ボタンを disabled=True に設定














法人 DM 基礎データ補正（アップロード）内部設計書
共通ヘッダー
業務 (システム) 名：法人 DM
対象アプリケーション：法人 DM 基礎データ補正（アップロード）
作成者：チィチィ
作成日：2026/03/15
ドキュメント名：画面構成情報 / イベント詳細
対象機能：基礎データ補正アップロード
更新者：チィチィ
更新日：2026/03/15
■ 1. パッケージ一覧
パッケージ名：ストリームリット
英字名：streamlit
概要：Python で Web アプリケーションを作成するフレームワーク
種別：標準パッケージ
備考：-
パッケージ名：パンダス
英字名：pandas
概要：Python でデータ加工を行うための代表的なライブラリ
種別：標準パッケージ
備考：-
パッケージ名：スノーパーク
英字名：snowpark
概要：Snowflake 上で Python/Java を使用してデータ処理を行うための開発フレームワーク
種別：標準パッケージ
備考：-
パッケージ名：IO モジュール
英字名：io
概要：ファイル入出力操作を行う標準ライブラリ
種別：標準パッケージ
備考：-
パッケージ名：正規表現モジュール
英字名：re
概要：正規表現操作を行う標準ライブラリ
種別：標準パッケージ
備考：-
パッケージ名：日付ユーティリティ
英字名：datetime
概要：日付・時間操作を行う標準ライブラリ
種別：標準パッケージ
備考：-
パッケージ名：バリデーションチェック部品
英字名：checks
概要：データフレームの項目別バリデーションを実行する共通部品
種別：共通部品
パス：common
備考：内部ステージに格納
パッケージ名：メッセージ取得・表示部品
英字名：message_util
概要：メッセージ ID から多言語メッセージを取得・画面表示する共通部品
種別：共通部品
パス：common
備考：内部ステージに格納
パッケージ名：Excel ユーティリティ
英字名：excel_file_util
概要：Excel ファイルを DataFrame に読み込む共通部品
種別：共通部品
パス：common
備考：内部ステージに格納
パッケージ名：バリデーションルール定義
英字名：validation_rules
概要：各補正対象テーブルのバリデーションルールを定義した設定ファイル
種別：共通部品
パス：common
備考：内部ステージに格納
パッケージ名：アップロードサービス
英字名：hosei_upload_service
概要：ファイル整合性チェック・DB 登録処理を実装したサービス部品
種別：サービス部品
パス：pages
備考：-
パッケージ名：共通ヘッダ
英字名：ui_common_header
概要：タイトルのほか部署名とログイン者の名前を表示する共通ヘッダ
種別：共通部品
パス：common
備考：内部ステージに格納
パッケージ名：SQL 取得部品
英字名：sql_reader
概要：ファイル名を指定し Stage から特定の SQL 文字列を取得する共通部品
種別：共通部品
パス：common
備考：内部ステージに格納
パッケージ名：SQL 実行部品
英字名：sql_runner
概要：SQL を実行するための部品
種別：共通部品
パス：common
備考：内部ステージに格納
■ 2. 画面項目一覧
項目名：画面名
物理名：-
仕様エリア：ヘッダーエリア
UI コンポーネント：st.subheader
備考：「法人 DM 基礎データ補正（アップロード）」を表示
項目名：補正対象テーブル
物理名：selected_table_en
仕様エリア：アップロードエリア
UI コンポーネント：st.selectbox
備考：補正対象のテーブルを選択
項目名：補正事由
物理名：correction_reason
仕様エリア：アップロードエリア
UI コンポーネント：st.text_input
備考：補正実施理由を入力
項目名：ファイルアップロード
物理名：upload
仕様エリア：アップロードエリア
UI コンポーネント：st.file_uploader
備考：xlsx、xls 形式のみ受付
項目名：処理メッセージ
物理名：messages
仕様エリア：メッセージエリア
UI コンポーネント：st.success/st.error
備考：処理結果 / エラーメッセージを表示
項目名：エラー一覧
物理名：error_display
仕様エリア：エラー一覧エリア
UI コンポーネント：st.markdown
備考：最大 10 件まで画面表示
項目名：チェック結果ダウンロード
物理名：download_button
仕様エリア：ボタンエリア
UI コンポーネント：st.download_button
備考：エラー存在時のみ有効
項目名：補正データの登録
物理名：submit_button
仕様エリア：ボタンエリア
UI コンポーネント：st.button
備考：ファイル選択 + 補正事由入力 + エラーなしの場合有効
項目名：アップロードしますか？
物理名：confirm_dialog
仕様エリア：確認ダイアログ
UI コンポーネント：@st.dialog
備考：登録前確認用
項目名：はい
物理名：confirm_yes
仕様エリア：確認ダイアログ
UI コンポーネント：st.button
備考：登録処理実行用
項目名：いいえ
物理名：confirm_no
仕様エリア：確認ダイアログ
UI コンポーネント：st.button
備考：ダイアログ閉じ用
■ 3. session_state 一覧
session_state 項目名：画面：セッション
英字名：session
概要：Snowpark セッション
デフォルト値：None
設定条件：初期処理で取得
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：DB 接続用
session_state 項目名：画面：エラー表示フラグ
英字名：hosei_upload ["show_error"]
概要：エラー一覧表示制御用フラグ
デフォルト値：FALSE
設定条件：初期化 / チェック処理後に設定
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：エラー存在時に TRUE
session_state 項目名：画面：ログ表示フラグ
英字名：hosei_upload ["showlog"]
概要：確認ダイアログ表示制御用フラグ
デフォルト値：FALSE
設定条件：初期化 / 実行ボタン押下後に設定
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：登録ボタン押下時に TRUE
session_state 項目名：画面：バリデーションエラーリスト
英字名：hosei_upload ["validation_errors"]
概要：全チェックエラーを保持（ダウンロード用）
デフォルト値：[]
設定条件：初期化 / チェック処理後に設定
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：エラーなしの場合は空配列
session_state 項目名：画面：表示用エラーリスト
英字名：hosei_upload ["display_errors"]
概要：画面表示用エラーリスト（最大 10 件）
デフォルト値：[]
設定条件：初期化 / チェック処理後に設定
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：超過時は合計件数のメッセージを追加
session_state 項目名：画面：検証済みデータ
英字名：hosei_upload ["valid_data"]
概要：DB 登録用のチェック正常済みデータ
デフォルト値：None
設定条件：初期化 / チェック成功後に設定
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：エラーありの場合は None
session_state 項目名：画面：基準年月
英字名：hosei_upload ["kijundate"]
概要：画面初期表示用の基準年月
デフォルト値：現在年月（% Y% m）
設定条件：初期化時に設定
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：システム日付から自動生成
session_state 項目名：画面：ユーザー ID
英字名：ss.user_id
概要：DB 登録時の更新者用ユーザー ID
デフォルト値："UNKNOWN"
設定条件：セッションから取得
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：ログイン情報から取得
session_state 項目名：画面：ユーザー名
英字名：ss.user_name
概要：画面表示用ログインユーザー名
デフォルト値："匿名"
設定条件：セッションから取得
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：ログイン情報から取得
session_state 項目名：画面：店舗番号
英字名：ss.shop_no
概要：DB 登録用店舗番号
デフォルト値："0000"
設定条件：セッションから取得
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：ログイン情報から取得
session_state 項目名：画面：Browse files のキー情報
英字名：hosei_upload ["uploader_key"]
概要：ファイルアップロードコンポーネントのリセット用キー
デフォルト値："0"
設定条件：フォーマットの選択値を変更した場合に設定
開始画面：法人 DM 基礎データ補正画面
終了画面：同左
備考：コンポーネントリセット時にインクリメント
■ 4. 初期処理
イベント概要
画面初期表示時にセッション状態を初期化し、共通ヘッダーと UI コンポーネントを描画する。
処理詳細
セッション状態の初期化
1.1) get_active_session で Snowflake と接続し、画面セッションに設定する。設定する session 名は、【session_state 一覧】を参照すること。
1.2) 下記の共通部品を呼び出し、共通ヘッダを表示する。
パッケージ名：共通ヘッダ（ui_common_header）
クラス名：UICommonHeader
メソッド名：ヘッダ描画 (head.draw)
引数：Page_title → 「法人 DM 基礎データ補正（アップロード）」
戻り値：なし
1.3) 下記のテーブル情報を取得する。
配置場所：DB:MAIN_H、スキーマ：UPLOAD、テーブル：フォーマットマスタ
取得項目名：FORMAT_ID、FORMAT_MEI
ソート順：HYOJI_JUN
1.4) DB に関する共通部品の初期化処理を行う。
a. SqlReader の初期化処理を行う。
パッケージ名：SQL 取得部品（sql_reader）
クラス名：SqlReader
メソッド名：コンストラクタ (init)
引数：sql_path → 「SC_5_01_05_01.sql」
戻り値：なし
b. SqlRunner の初期化処理を行う。
パッケージ名：SQL 実行部品（sql_runner）
クラス名：SqlRunner
メソッド名：コンストラクタ (init)
引数：session → 上記 1.1) で取得した SqlReader
戻り値：なし
1.5) 下記の共通部品を呼び出し、SQL を実行する。
パッケージ名：SQL 実行部品（sql_runner）
メソッド名：SELECT 文を実行処理 (query)
引数：sql → get_format
戻り値：結果
※SQL 文は、No.1.3 を参照すること。
1.6) st.selectbox () の options に上記処理で取得したデータを設定する。
■ 5. イベント詳細（ファイル選択後イベント）
イベント概要
画面より「Browse files」ボタンの操作を行われた際にチェック処理を行って、メッセージを画面に表示する。
処理詳細
「Browse files」ボタン押して、ファイル選択後のイベント
1.1) st.selectbox に値が設定されることを確認する。
1.2) file_upload のファイル名をチェックする。
※チェック内容は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（ファイル選択後）シート」にある項番 2.2) を参照すること。
チェック処理にエラーがある場合、以下に設定して、1.5) を実施する。
hosei_upload ["messages"]["error"] → 上記のエラーメッセージを設定 ※セッション設定
hosei_upload ["error_df"] → 上記のエラーメッセージを設定（ダウンロード用） ※セッション設定
hosei_upload ["error_log"] → 上記のエラーログメッセージを設定（画面表示用） ※セッション設定
1.3) 上記処理にエラーがない場合、file_upload がアップロード可否をチェックする。
a. 下記の共通部品を呼び出し、SQL を実行する。
パッケージ名：SQL 実行部品（sql_runner）
メソッド名：SELECT 文を実行処理 (query)
引数：sql → st.selectbox の値が契約明細項目の場合、get_upload_info_by_uchiwake、上記以外の場合、get_upload_info_by_key
引数：param → st.selectbox の値が契約明細項目の場合、{"uchiwake_code": ファイル名からの内訳コード}、上記以外の場合、{"uchiwake_code": ファイル名からの内訳コード，"kijunym": ファイル名からの基準年月 / 基準年}
戻り値：結果
※SQL 文は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「(別紙) 処理詳細 (新規・アップロード不可判断) シート」にある項番 2.1) を参照すること。
b. 上記の結果によって、ステータスを設定。
※設定条件は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「(別紙) 処理詳細 (新規・アップロード不可判断) シート」にある項番 2.2) を参照すること。
c. 上記で設定したステータスによってチェック処理を行う。
※チェック内容は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（ファイル選択後）シート」にある項番 2.3.2、2.3.3 を参照すること。
チェック処理にエラーがある場合、以下に設定して、1.5) を実施する。
hosei_upload ["messages"]["error"] → 上記のエラーメッセージを設定 ※セッション設定
チェック処理にエラーがない場合、以下に設定する。
hosei_upload ["uchiwake_code"] → ファイル名から抽出した内訳コードを設定 ※セッション設定
hosei_upload ["kijun_ym"] → ファイル名から抽出した基準年月を設定 ※セッション設定
hosei_upload ["format_id"] → st.selectbox の値を設定 ※セッション設定
hosei_upload ["file_name"] → st.file_uploader のファイル名を設定 ※セッション設定
1.4) 上記処理にエラーがない場合
a. excel_file_util を使用して file_upload を読み込む。
b. 各フォーマットの fmt_check 処理を呼び出す。
サービス名：対象フォーマットのクラス（【別紙】フォーマットとソースファイル一覧）シートを参照
メソッド名：fmt_check（フォーマットチェック処理）
引数：df → 上記処理で読み込んだデータ
引数：kijunym → 基準年月（ファイル名から取得）
戻り値：エラーリスト、DF
c. 上記の処理から戻り値を以下に設定。
hosei_upload ["data_df"] → 戻り値の DF ※セッション設定
file_data_errors → 戻り値のエラーリスト
1.4.2) st.selectbox に選択された値は固定長フォーマットの場合、
a. 各フォーマットの fmt_check 処理を呼び出す。
サービス名：対象フォーマットのクラス（【別紙】フォーマットとソースファイル一覧）シートを参照
メソッド名：fmt_check（フォーマットチェック処理）
引数：file_upload → アップロードファイル
引数：kijunym → 基準年月
戻り値：エラーリスト、マスタ DF、データ DF
b. 上記の処理から戻り値を以下に設定。
hosei_upload ["master_df"] → 戻り値のマスタ DF ※セッション設定
hosei_upload ["data_df"] → 戻り値のデータ DF ※セッション設定
file_data_errors → 戻り値のエラーリスト
1.4.3) file_data_errors に値がある場合、
hosei_upload ["messages"]["error"] = msg_util.get_messages_data ("456ERR4058") ※セッション設定
hosei_upload ["error_df"] = pd.DataFrame (file_data_errors) ※セッション設定
hosei_upload ["error_log"] = file_data_errors ※セッション設定
1.4.4) file_data_errors に値がない場合、
hosei_upload ["messages"]["success"] = msg_util.get_messages_data ("456INF0012") ※セッション設定
hosei_upload ["btn_jiko"] = FALSE ※セッション設定
hosei_upload ["btn_download"] = TRUE ※セッション設定
1.5) メッセージを表示する。
a. hosei_upload ["messages"]["error"] に値がある場合、
i. st.error に上記のメッセージを設定して画面に表示する。
hosei_upload ["btn_jiko"] = TRUE ※セッション設定
hosei_upload ["btn_download"] = FALSE ※セッション設定
ii. st.markdown（チェック結果ログ）に hosei_upload ["error_log"] をループして 10 個まで表示する。
b. hosei_upload ["messages"]["success"] に値がある場合、
st.success に上記のメッセージを設定して画面に表示する。
■ 6. イベント詳細（「補正データの登録」ボタン押下）
イベント概要
画面より「補正データの登録」ボタンの操作を行われた際に DB 登録処理を行って、メッセージを画面に表示する。
処理詳細
「補正データの登録」ボタン押下後のイベント
1.1) @st.dialog を表示する。
1.2) @st.dialog の「はい」ボタン (st.button) を押下された場合、
1.2.1) file_upload のアップロード可否をチェックする。
a. 下記の共通部品を呼び出し、SQL を実行する。
パッケージ名：SQL 実行部品（sql_runner）
メソッド名：SELECT 文を実行処理 (query)
引数：sql → st.selectbox の値が契約明細項目の場合、get_upload_info_by_uchiwake、上記以外の場合、get_upload_info_by_key
引数：param → st.selectbox の値が契約明細項目の場合、{"uchiwake_code":hosei_upload ["uchiwake_code"]}、上記以外の場合、{"uchiwake_code":hosei_upload ["uchiwake_code"], "kijunym":hosei_upload ["kijun_ym"]}
戻り値：結果
※SQL 文は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「(別紙) 処理詳細 (新規・アップロード不可判断) シート」にある項番 2.1) を参照すること。
b. 上記の結果によって、ステータスを設定。
※設定条件は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「(別紙) 処理詳細 (新規・アップロード不可判断) シート」にある項番 2.2) を参照すること。
c. 上記で設定したステータスによってチェック処理を行う。
※チェック内容は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（「実行」ボタン押下）シート」にある項番 2.2.2、2.2.3 を参照すること。
チェック処理にエラーがある場合、st.error に正常終了したメッセージを表示する。
hosei_upload ["messages"]["error"] → 上記のエラーメッセージを設定 ※セッション設定
当イベントの処理を終了する。
i. 上記以外の場合、ステータスを設定するための処理を行う。
※チェック内容は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（「実行」ボタン押下）シート」にある項番 2.2.2 の b を参照すること。
1.2.2) DB から取込 ID を取得するため SQL を実行する。
a. 下記の共通部品を呼び出し、SQL を実行する。
パッケージ名：SQL 実行部品（sql_runner）
メソッド名：SELECT 文を実行処理 (query)
引数：sql → get_torikomi_id
戻り値：結果
※SQL 文は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（「実行」ボタン押下）シート」にある項番 2.3.1 を参照すること。
b. データ登録用 取込み ID を設定する。
※設定条件は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（「実行」ボタン押下）シート」にある項番 2.3.2、2.3.3 を参照すること。
1.2.3) DB から管理情報を取得する。
a. 下記の共通部品を呼び出し、SQL を実行する。
パッケージ名：SQL 実行部品（sql_runner）用メソッド名を実行処理 (execute)
メソッド名：INSERT / UPDATE / DELETE 用メソッド名を実行処理 (execute)
引数：sql → insert_to_fileuploadkanri
引数：param → 外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（「実行」ボタン押下）シート」にある項番 2.3) を参照すること。
戻り値：結果
※SQL 文は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（「実行」ボタン押下）シート」にある項番 2.3) を参照すること。
1.2.4) アップロード管理テーブルにデータ登録する。
※詳細内容は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（「実行」ボタン押下）シート」にある項番 2.3) を参照すること。
a. st.selectbox に選択された値は固定長フォーマット以外の場合、
サービス名：対象フォーマットのクラス（【別紙】フォーマットとソースファイル一覧）シートを参照
メソッド名：insert_to_work_table（ワークテーブルへ登録処理）
引数：data_df → hosei_upload ["data_df"]（アップロードデータ情報）
引数：uchiwake_code → hosei_upload ["uchiwake_code"]（内訳コード）
引数：kijun_ym → hosei_upload ["kijun_ym"]（基準年月）
引数：torikomi_id → No.1.2.2 に取得した取り込み ID
戻り値：登録結果
b. st.selectbox に選択された値は固定長フォーマットの場合、
サービス名：対象フォーマットのクラス（【別紙】フォーマットとソースファイル一覧）シートを参照
メソッド名：insert_to_work_table（ワークテーブルへ登録処理）
引数：master_df → hosei_upload ["master_df"]（アップロードマスタ情報）
引数：data_df → hosei_upload ["data_df"]（アップロードデータ情報）
引数：uchiwake_code → hosei_upload ["uchiwake_code"]（内訳コード）
引数：kijun_ym → hosei_upload ["kijun_ym"]（基準年月）
引数：torikomi_id → No.1.2.2 に取得した取り込み ID
戻り値：登録結果
1.2.4) 上記の登録処理が異常終了した場合、st.error に正常終了したメッセージを表示する。
※詳細内容は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（「実行」ボタン押下）シート」にある項番 2.5) を参照すること。
1.2.5) 上記の登録処理が正常終了した場合、st.success に正常終了したメッセージを表示する。
※詳細内容は、外部設計書：「5-04-01-0_入出力設計書_法人 DM 基礎データ補正.xlsx」 - 「イベント詳細（「実行」ボタン押下）シート」にある項番 2.6) を参照すること。
1.3) @st.dialog の「いいえ」ボタン (st.button) を押下された場合、
st.dialog を非表示にして当処理を終了する。
■ 7. イベント詳細（「チェック結果ダウンロード」ボタン押下）
イベント概要
画面より「チェック結果ダウンロード」ボタンの操作を行われた際にチェック結果ログを Excel としてダウンロードする。
処理詳細
「チェック結果ダウンロード」ボタン押下後のイベント
1.1) hosei_upload ["error_df"] のデータをダウンロードする。
■ 8. イベント詳細（ファイル削除時）
イベント概要
アップロードしたファイルの「×」ボタンを押下し、ファイルを削除した際にセッション情報をリセットする。
処理詳細
ファイル削除イベント検知
1.1) upload is None の場合、以下のセッション情報をリセットする。
hosei_upload ["show_error"] → False
hosei_upload ["validation_errors"] → []
hosei_upload ["display_errors"] → []
hosei_upload ["valid_data"] → None
hosei_upload ["btn_submit"] → False
hosei_upload ["messages"] → {"error": "","success":""}
1.2) 画面をリロードし、初期状態に戻す。
