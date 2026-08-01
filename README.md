# `Express`におけるデータベース連携と非同期処理

## `JavaScript`の非同期処理
前回まで`Node.js`環境で`Express`を使ったWebアプリケーション作成について勉強してきました。<br>
今回は、WEBアプリケーションをデータベースと連携させて、大規模なデータを永続的に扱えるようにしたいと思います。<br>

その前に、`JavaScript`でデータベースを扱う時には、避けては通れない「**非同期処理**」という重要な概念があります。<br>
ここからは実際にJavaScriptのコードを実行しながら、まずは非同期処理について理解していきたいと思います。<br>
`Node.js`環境の任意の場所に`test`というディレクトリを用意しておいてください。<br>

以下は、`JavaScript`のコードです。みなさんはこのコードの動作を想像できますか？<br>
```JavaScript
console.log(1);
console.log(2);
console.log(3);
```
それでは、用意したディレクトリの中に`test1.js`という`JavaScript`ファイルを作成して、上記コードをコピー&ペーストしてみましょう。<br>
そして、`test1.js`を保存出来たら、以下のコマンドを実行して`JavaScript`ファイルを実行してみましょう。<br>
```bash
node test1.js
```
実行結果は予想通りだったでしょうか？以下のような出力がされたはずです。<br>
ここでは、`JavaScript`の3行のコードは上から下へと実行されています。<br>
```bash
1
2
3
```

それではここで、`JavaScript`の以下の関数を覚えてもらいたいと思います。<br>
```JavaScript
setTimeout(function, delay)

//function：遅延後に実行する関数。
//delay   ：遅延時間をミリ秒単位で指定。
```
この **`setTimeout`** は第2引数で指定したミリ秒後に第1引数で受けた関数を実行させるというはたらきを持っています。<br>
実際の動作を確かめるために、`test2.js`という`JavaScript`ファイルを作成して、下記コードをコピー&ペーストしてみましょう。<br>
```JavaScript
setTimeout(()=>{
    console.log('1秒後に表示');
},1000);
```
このコードは、`setTimeout`の第1引数に「引数なしで`console.log`を実行するアロー関数」を、第2引数に「1000」という数値を渡しています。<br>
ファイルの保存後に先程と同様にコマンドから`JavaScript`ファイルを実行すると、1秒後に以下のような表示が出現するはずです。<br>
```bash
1秒後に表示
```
`setTimeout`の仕組みが理解できれば、`test2.js`の中身をすべて削除して、次のコードをコピー&ペーストして保存して、コマンドから実行してみてください。<br>
```JavaScript
console.log(1);
setTimeout(()=>{
  console.log(2);
},1000);
console.log(3);
```
コマンドから`JavaScript`ファイルを実行すると、実行後にすぐ以下のように表示されるはずです。<br>
```bash
1
3
```
そして、1秒後に表示が追加されて、全体としては以下のような表示になると思います。<br>
```bash
1
3
2
```
みなさんはこの結果を予想できましたか？実は、この結果に`JavaScript`の非同期処理という特徴が現れています。<br>

`JavaScript`は、WEBブラウザの表示に動きを付けるために設計されたシングルスレッドで動作するプログラミング言語です。<br>
シングルスレッドという特徴を持つ以上、`JavaScript`は、基本的には一度に1つの処理しか実行できません。<br>
しかし、時間のかかる処理を1つずつ実行するとWEBブラウザの利用者のユーザー体験が悪くなってしまいます。<br>
そこで、`JavaScript`には、時間のかかる処理は、完了を待たずに次の処理へ進むという**非同期処理**が組み込まれたのです。<br>
実際に、みなさんがWEBブラウザを操作するときもこの`JavaScript`の非同期処理の恩恵を受けているはずです。<br>
例えば、画像や動画のロードが進行中でも、他の要素が先に表示され、ページの表示速度が遅いという印象が緩和されます。<br>

先程のコードは、この非同期処理によって、以下のように動作していたのです。

1. console.log(1)が実行され、1と表示される。
2. setTimeoutは時間がかかる処理なので、非同期処理の効果で完了を待たずに次に進む。
3. console.log(3)が実行され、3と表示される。
4. 実行できる全ての処理が完了したため、まだ完了していないsetTimeoutが完了するのを待つ。
5. 1秒後にconsole.log(2)が実行され、2と表示される。

ここで、非同期処理についてよくある誤解なのですが、後回しされた時間のかかる処理は、完了後にすぐに表示されるのではなく、<br>
上の4のように時間のかからない処理が全て終わった後に結果を受け取るという仕組みになっています。<br>
以下のコードを`test3.js`にコピー＆ペーストして保存し、コマンドから実行してみてください。
```JavaScript
console.log(1);
setTimeout(()=>{
  console.log(2);
},0);
console.log(3);
console.log(4);
console.log(5);
console.log(6);
console.log(7);
console.log(8);
console.log(9);
```
実行結果は以下の通りです。
```bash
1
3
4
5
6
7
8
9
2
```
たとえ、`setTimeout`で指定した遅延時間が0ミリ秒であっても、`setTimeout`は、時間のかかる処理として後回しになっているのです。<br>
そして、時間のかからない処理が全て終わったタイミングで、初めて非同期処理の結果が処理に組み込まれるのです。<br>


`Node.js`環境でも、この`JavaScript`の非同期処理の仕組みが組み込まれています。<br>
例で見たタイマー処理やファイル操作、ネットワークを介してやりとりをする処理などは、時間のかかる処理として、後回しにされます。<br>
もちろん、今回の主役であるデータベースとの連携処理も時間がかかる処理として後回しにされることになります。<br>
この`JavaScript`の仕組みがある以上、工夫してデータベースとの連携処理をコーディングしないと以下のような不具合が生じることになります。<br>

- データベースからデータを取得する前に、後続のコードが実行され、取得したデータが使われずに処理が進む
- データの追加や更新、削除のタイミングがずれてしまい、結果的に意図しない順序で処理が実行される

次の章では、この非同期処理を上手く扱うための`JavaScript`の機能を学んでいきましょう。<br>

## コールバック関数とその限界
`JavaScript`で非同期処理の結果を受け取る最も基本的な方法が「**コールバック関数**」の利用です。<br>
この方法では、非同期で実行される処理の引数に非同期で実行される関数を渡す書き方をします。<br>
実際に、このコールバック関数を覚えることで、データベースとの連携処理は、一応は、書けるようになります。<br>
ただし、コールバック関数を覚えるだけでは、やがてある重大な問題に突き当たることになります。<br>
以下の`setTimeout`のコールバック関数で書かれたコードを見てください。<br>
```JavaScript
setTimeout(() => {
    console.log(1);
    setTimeout(() => {
        console.log(2);
        setTimeout(() => {
            console.log(3);
            setTimeout(() => {
                console.log(4);
                setTimeout(() => {
                    console.log(5);
                }, 1000);
            }, 1000);
        }, 1000);
    }, 1000);
}, 1000);

```
コードの内容は、1秒ごとに数字が1から順に5まで表示される`JavaScript`の記述です。<br>
こんな単純な処理のはずが、恐ろしく読みずらいコードになってしまっています。<br>
コールバック関数は多用すると「**コールバック地獄**」と呼ばれるネストが深くなり可読性が低下する問題が発生します。<br>

この問題を解決するために、`JavaScript`では、「**`Promise`**」というオブジェクトが導入されました。<br>

## Promiseオブジェクト
`Promise`オブジェクトは、処理の状態を「**`pending`**（未解決）」と「 **`fulfilled`**（解決済み）」の2つで表現します。<br>

まずは、「`pending`（未解決）」の状態の`Promise`を実際のコードを書きながら確認していきましょう。<br>
`test4.js`という`JavaScript`ファイルを作成して、下記コードをコピー&ペーストして保存してみましょう。<br>
```JavaScript
const promise = new Promise(()=>{});
console.log(promise);
```
これは何も処理をしないアロー関数をコンストラクタの引数に渡して`Promise`オブジェクトをインスタンス化しています。<br>
`Promise`オブジェクトでは、何か特別な動作や結果が指定されない限り、「`pending`」の状態のままになります。<br>
この`test4.js`を実行すると、以下のように表示されるはずです。<br>
```bash
Promise { <pending> }
```
`{}`の中に`<pending>`と表示されていて、`Promise`オブジェクトが未解決の状態になっていることが分かります。<br>

それでは、次に`Promise`オブジェクトが「`fulfilled`（解決済み）」になっているコードの例を見てみましょう。<br>
`test5.js`という`JavaScript`ファイルを作成して、下記コードをコピー&ペーストして保存してみましょう。
```JavaScript
const promise = new Promise((resolve)=>{
    resolve('実行完了');
});
console.log(promise);
```
コードの説明の前に、`test5.js`を実行して結果を確かめてみましょう。以下のように表示されるはずです。<br>
```bash
Promise { '実行完了' }
```
先程は「`<pending>`」と表示されていた部分が「`'実行完了'`」となっているはずです。<br>

それでは、このコードの中身を詳しく見ていきましょう。<br>
このコードでは、`Promise`オブジェクトをインスタンス化する際のコンストラクタに渡す関数が先程と異なっています。<br>
その関数の部分のみを取り出してみると以下のようになります。<br>
```JavaScript
(resolve)=>{
    resolve("実行完了");
}
```
この関数の1つ目の特徴は、引数に`resolve`という関数を受けます。この関数は「**resolve関数**」と呼ばれます。<br>
そして、2つ目の特徴は、処理の中で、引数で受けたresolve関数を実行している点です。<br>

実は、`Promise`オブジェクトは、resolve関数が実行されたタイミングで、状態が「`pending`」から「 `fulfilled`」に変化します。<br>
そして、resolve関数の引数に渡した値(ここでは「実行完了」という文字列)が実行結果としてセットされます。<br>
ターミナルの表示は、`Promise`オブジェクトが`fulfilled`になり、セットされた実行結果が表示されていたのです。<br>

そして、この`Promise`オブジェクトから実行結果を取り出して処理をしたい場合は、`Promise`オブジェクトの`then`というメソッドを使います。<br>
`test5.js`に以下の3行を追記して、保存して実行してください。<br>
```JavaScript
promise.then((result)=>{
    console.log(result);
});
```
`Promise`オブジェクトの`then`メソッドは、`Promise`オブジェクトが`fulfilled`になるのを待ち受け、その実行結果を受け取ることができます。<br>
受け取った実行結果は、`then`メソッドの引数の関数の引数(ここではresult)に渡され、処理の中で自由に使用できます。<br>
`test5.js`の実行結果は、以下のような表示になるはずです。<br>
```bash
Promise { '実行完了' }
実行完了
```
1行目の表示は`Promise`オブジェクトをそのまま出力しているものですが、2行目の表示は取り出した実行結果を表示しています。<br>

なお、resolve関数に複数の引数を渡して、`then`メソッドで受け取ることはできません。そのような場合は配列やオブジェクト型を活用しましょう。<br>
また、resolve関数に何も渡さない場合は、実行結果を受け取ることはできませんが、`Promise`オブジェクトを`fulfilled`に変える動作は行われます。<br>

```JavaScript
const promise1 = new Promise((resolve)=>{
    resolve('実行完了1','実行完了2'); //2つ目の引数はthenメソッドで受け取られない
});

const promise1 = new Promise((resolve)=>{
    resolve(); //引数がない場合、Promiseの状態をfulfilledに変えるだけの動作となる
});
```

ここまで、resolve関数で`Promise`オブジェクトを`pending`の状態から`fulfilled`の状態に変える方法を見てきました。<br>
実は、`Promise`オブジェクトを`fulfilled`の状態に変える方法はもう1つあります。それは、「**`reject`関数**」です。<br>
resolve関数は**正常系**と呼ばれる処理が順調に進んだ場合に使用しますが、それに対してreject関数は**異常系**と呼ばれる処理がエラーになった場合に使用します。<br>
reject関数は、`Promise`オブジェクトをインスタンス化する際のコンストラクタに渡す関数の第2引数に渡されます。<br>
そして、`then`メソッドでresolve関数の引数を受けた形と同じように、`Promise`オブジェクトの`catch`メソッドでreject関数の引数を受け取ることができます。<br>
※関数名ではなく、何番目の引数かが重要になります。名前を`reject`としても、第1引数に渡すと`resolve`の動作をする関数になってしまいます。<br>
`test6.js`に以下のコードをコピー＆ペーストして、保存してみましょう。
```JavaScript
const promise = new Promise((resolve,reject)=>{
    reject('エラーが発生しました');
});
promise.catch((error)=>{
    console.log(error);
});
```
実行すると以下のような表示になると思います。
```bash
エラーが発生しました
```
なお、`then`メソッドと`catch`メソッドは連ねて記述することができ、この書き方をプロミスチェーンと呼びます。<br>
それでは、`Promise`オブジェクトと非同期処理を組み合わせた処理を実際に実行してみたいと思います。
`test7.js`に以下のコードをコピー＆ペーストして、保存してみましょう。
```JavaScript
const targetNumber = 0;

const promiseFunction = (number)=>{
    return new Promise((resolve,reject)=>{
        setTimeout(() => {
            if(number % 2 == 0){
                resolve(number);
            }else{
                reject('奇数が投入されました');
            }
        }, 1000);
    });
}

promiseFunction(targetNumber)
    .then((result)=>{
        console.log('投入された数は' + result + 'です');
    })
    .catch((error)=>{
        console.log('エラー発生:' + error);
    });
```
このコードの中では、`Promise`オブジェクトの返す`promiseFunction`を宣言しています。<br>
この関数は実行開始から1秒後に、引数が偶数ならばresolve関数が呼ばれ、奇数ならreject関数が呼ばれます。<br>
実際にこの`JavaScript`ファイルを実行すると、1秒後に以下のように表示されるはずです。<br>
```bash
投入された数は0です
```
この表示が確認できた人は、1行目を以下のように書き換えて保存してみましょう。<br>
```JavaScript
const targetNumber = 1;
```
実行してみると、こちらも1秒経ってから、以下のように表示されるはずです。
```bash
エラー発生:奇数が投入されました
```
以上で  `Promise`オブジェクトと`then`や`catch`の使い方の説明は終わりです。
ここで、先程のコールバック地獄のコードを`Promise`オブジェクトを返す関数と`then`を使って書き直してみたコードを見てみましょう。<br>

```JavaScript
const logAfterOneSecond = (number) => {
    return new Promise((resolve) => {
        setTimeout(() => {
            console.log(number);
            resolve();
        }, 1000);
    });
};

logAfterOneSecond(1)
    .then(() => logAfterOneSecond(2))
    .then(() => logAfterOneSecond(3))
    .then(() => logAfterOneSecond(4))
    .then(() => logAfterOneSecond(5));
```
コールバック地獄のコードと比べると、かなり見やすくなったのではないでしょうか？

## `JavaScript`の`async`・`await`キーワード

`JavaScript`では近年、 **「`async`/`await`構文」** が導入され、非同期処理の記述の可読性をより高めることができるようになりました。<br>
`async`キーワードは関数宣言の前に置かれ、その関数が非同期関数であることを示し、返り値を常にPromiseでラップして返します。<br>
```JavaScript
const myArrowFunction = async()=>{
  return "Hello"; // Promise((resolve)=>{resolve('Hello')})と同等
};
```
`await`キーワードは`async`関数内でのみ使用でき、直後に記述する`Promise`オブジェクトが解決するまで`async`関数の実行を一時停止させ、解決後にその結果を取り出して、処理を再開させます。<br>
```JavaScript
const myArrowFunction = async()=>{
    const result1 = await promiseFunction(); //awaitキーワードによりthenを使わなくてもPromiseオブジェクトの結果を取り出す
    const result2 = function(); //async関数内では、awaitで待ち受けるPromiseオブジェクトが解決するまで次の処理には進まない

    return result1 + result2; //async関数の効果によって返り値は自動的にPromiseオブジェクトでラップされる
}
```
また、`async`/`await`構文では、`Promise`オブジェクトについて、`reject`関数が実行された場合は、`try-catch文`でエラーを補足できます。<br>

ちなみに、`async`/`await`構文を使うと、1秒ごとに数字が1から順に5まで表示させるプログラムは以下のようになります。<br>
`test7.js`に以下のコードをコピー&ペーストして保存し、実行して結果を確かめてみましょう。
```JavaScript
const logAfterOneSecond = (number) => {
    return new Promise((resolve) => {
        setTimeout(() => {
            console.log(number);
            resolve();
        }, 1000);
    });
};

const logNumbers = async () => {
    await logAfterOneSecond(1);
    await logAfterOneSecond(2);
    await logAfterOneSecond(3);
    await logAfterOneSecond(4);
    await logAfterOneSecond(5);
};

logNumbers();
```
ここまで、かなり遠回りをしてきましたが、次の章ではいよいよ`Express`でデータベースとを連携させる方法を説明していきます。<br>

## `Express`と`MySQL`の連携
まずは、`Node.js`環境上に、`dbTest`というプロジェクトディレクトリを作成し、`npm init -y`を実行して、プロジェクトを初期化しましょう。<br>
続いて、パッケージのインストールですが、今回は`Express`に加えて、`MySQL`というデータベースを操作するためのパッケージもインストールします。<br>
以下のコマンドを実行してください。 ※`mysql2`の「2」は現在使われている改良版のパッケージであることを示すものです。<br>
```bash
npm install express mysql2
```
続いて、`src/server.js`に以下のような`JavaScript`のコードを記述していってください。<br>
`Docker`の配布環境を使用している人は問題ありませんが、独自で環境構築した人はコード中の`host`,`user`,`password`を環境に合うように設定してください。<br>
```JavaScript
const express = require('express');
const mysql = require('mysql2/promise');
const PORT = 3000;

const app = express();

const pool = mysql.createPool({
    host:'mysql', //mysqlのホスト名やIPアドレスを記述
    user:'user', //mysqlのユーザー名を記述
    password:'password', //mysqlのユーザー名に対応するパスワードを記述
    database:'app', //接続するデータベース名を記述
    dateStrings: true //日付を文字列として取得する設定
});

app.get('/',async(req,res)=>{
    const [rows, fields] = await pool.query('SELECT now() AS "DB環境の現在時刻",1+1 AS "計算テスト"');
    res.send(rows);
});

app.listen(PORT, () => {
    console.log(`サーバー起動中…（ポート番号:${PORT}）`);
});
```
このコードの説明をしていきたいと思います。<br>

まず、2行目は`npm install mysql2`で取得した`mysql2`パッケージのモジュールを`require`で読み込み、定数`mysql`にオブジェクトとして代入しています。<br>
このとき、呼び出すパッケージを`mysql2`ではなく、`mysql2/promise`とすることで、データベースから返される結果が`Promise`でラップされます。
7行目からは、`mysql`のオブジェクトから**コネクションプール**のオブジェクトを作成する`createPool`というメソッドを呼び出しています。<br>
このメソッドの引数には、接続に必要な情報をオブジェクトリテラルで記述したものを渡していて、他にも使用するデータベース名を指定する`database`という項目も設定できます。<br>
コネクションプールとは、データベースとの接続を毎回切断するのではなく、貯めておくことで再利用を可能にして効率的にリソース管理をするという仕組みです<br>

そして、13行目からは実際にデータベースに接続して、DB環境の現在時刻や算術演算機能を呼び出しています。<br>
まずは、`app.get`の第2引数となっているアロー関数に注目しましょう。この関数は`async`関数となっています。<br>
そして、データベースに`SQL`を実行させるコネクションプールオブジェクトの`query`メソッドを`await`キーワードをつけて使用しています。<br>
データベースへの接続は、`JavaScript`では時間がかかる処理として、デフォルトでは、後回しにされてしまいます。<br>
しかし、`async/await`構文により、後続の`res.send`は、データベースに接続して、結果が返ってくるまで実行が止まることになるのです。<br>
`query`メソッドは2つのオブジェクトを入れた配列を返しますが、1つ目のオブジェクトはSQLの結果、2つ目のオブジェクトは結果のメタデータが格納されています。<br>

さて、大まかなコードの仕組みを理解した上で、以下のコマンドからWEBサーバーを起動してみましょう。
```bash
node src/server.js
```
そして、サーバーが起動できれば、[http://localhost:3000/](http://localhost:3000/)にWEBブラウザでアクセスしてみましょう。
以下のような文字列が表示されれば、正常にデータベースと接続できています。
```json
[{"DB環境の現在時刻":"yyyy-MM-ddThh:mm:ss.000Z","計算テスト":2}]
```
## この後の学習
ここまで`Express`におけるデータベース連携と非同期処理を学んできましたが、ここからは、この後にどのような学習を進めればいいかを示してみます。
- `MySQL`側で`CREATE DATABASE`文からデータベースを作成した上で、簡単なテーブルを`CREATE TABLE`文から作ってみましょう。
- `Express`側で`mysql2`の`createPool`メソッドにデータベース名の情報を加えて、作成したテーブルにアクセスするルートハンドラを追加してみましょう。
- `Express`側から`INSERT`文や`UPDATE`文を発行するルートハンドラを追加してみましょう。
- テンプレートエンジン`EJS`を活用して、データベースから取得したデータを`HTML`形式にレンダリングしてみましょう。
- `dotenv`パッケージをインストールして、`.env`ファイルを利用したデータベースのパスワード情報の環境変数としての取り込みを行ってみましょう。
- `passport`パッケージと`bcrypt`パッケージをインストールして、データベースを使い、パスワードのハッシュ化も施すような認証・認可の機能を実装してみましょう。
