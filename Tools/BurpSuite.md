# BurpSuite

### プロキシ情報

![image-20221229155221773](img/BurpSuite/image-20221229155221773.png)



### 証明書のインストール

BurpSuiteのProxy > Options > Proxy Listenersに移動し、Regenerate CA certificateをクリック

![image-20221229155014536](img/BurpSuite/image-20221229155014536.png)

確認ダイアログで「はい」をクリックし、Burp Suiteを再起動

新しいCA証明書をFirefoxにインポートするために、まず、http://burp をブラウズして、証明書へのリンクを見つける

![image-20221229155043161](img/BurpSuite/image-20221229155043161.png)

証明書を表示するには、この画面で CA 証明書をクリックし（または http://burp/cert に接続し）、cacert.der ファイルをローカルマシンに保存

![image-20221229155105856](img/BurpSuite/image-20221229155105856.png)

ダウンロードが完了したら、ダウンロードしたファイルをFirefoxにドラッグ＆ドロップし、「Trust this CA to identify websites」を選択して「OK」をクリック

![image-20221229155128069](img/BurpSuite/image-20221229155128069.png)



### Intruderを利用したパスワードアタック

* リクエストをプロキシしHTTPHistoryからintruderにリクエストを転送

  ![image-20221229213421814](img/BurpSuite/image-20221229213421814.png)

* intruderタブをクリックする。Targetは特に設定不要

  ![image-20221229213505979](img/BurpSuite/image-20221229213505979.png)

* Positionsタブを編集する

  ![image-20221229213529884](img/BurpSuite/image-20221229213529884.png)

  * `Clear $`を押下し選択を解除した後、動的に設定した箇所のみ選択して`Add $`で追加していく。ここで選択した箇所ぶん後ほどpayloadの設定をする

    ![image-20221229213619757](img/BurpSuite/image-20221229213619757.png)

    * set-cookieの箇所を2か所設定
    * tokenの箇所を1か所設定
    * パスワードの箇所を1か所設定

    計4か所に設定をする

    * Attack typeに`Pitchfork`を設定すること

* Optionタブを設定する

  * Grep - Extractセクションを設定する。 "Recursive grep "というペイロードがあり、あらかじめ定義された値をgrep4で検索して、その結果を次のリクエストで利用できるようにするための設定

    ![image-20221229214031620](img/BurpSuite/image-20221229214031620.png)

    cookieの設定箇所を選択して登録する

    ![image-20221229214042981](img/BurpSuite/image-20221229214042981.png)

    tokenの設定箇所も選択して登録する

    ![image-20221229214128104](img/BurpSuite/image-20221229214128104.png)

* 「payload」タブを設定する

  * 1つ目と2つ目はcookieの設定。Payload typeを「Recursive grep」に設定する
    ![image-20221229214200632](img/BurpSuite/image-20221229214200632.png)

  * 3つ目はパスワードの設定。Payload typeを「Simple list」に設定する

    ![image-20221229214328351](img/BurpSuite/image-20221229214328351.png)

  * 4つ目のtokenを設定する

    ![image-20221229214345707](img/BurpSuite/image-20221229214345707.png)

  * 「Resource Pool」タブを設定する

    ![image-20221229214440152](img/BurpSuite/image-20221229214440152.png)

  * 「Start Attack」で攻撃開始する。

    ![image-20221229214506367](img/BurpSuite/image-20221229214506367.png)

    
