# kotlinのcoroutineの第一歩

Coroutineはkotlinで導入された非同期のしくみ。  
非同期処理がきれいに書けるんだけど、C#やEcmaのAsync/Awaitとは書き味が結構違う  
メモは自分で色々試した結果だけど運用したわけじゃないので間違ってるかも

## 書き方

* 導入方法はめんどくさいので端折り
* 動くコードにするのがめんどくさいので以下に置くコードがそのまま動くとは限らない
* コードはandroidのやつ
* 目指すところは一定時間待つスプラッシュ画面で同時に裏で他の処理を行うところ

### launch

CoroutineScopeのlaunchから始めるのが良さそう  
以下がCoroutineを開始するコード

```kotlin
val uiScope = CoroutineScope(Dispatchers.Main)
uiScope.launch {
    // ここに処理を書く
}
```

* launch内がバックグラウンドに・・・なるわけではない。
* Dispatchers.Mainを指定しているのでMainのスレッド？で動く
* じゃあこれは何かと言うと、この内側がコルーチンになる
* suspend関数といわれるものとして？動くので、この中でdelayしたりできる
* RxJavaで書くときにStreamに対してsubscribeしていたあたりに該当する

### delay

JSだったらsetTimeoutとかしてたやつ

```kotlin
uiScope.launch {
    Log.d("before")
    delay(10000)
    Log.d("after")
}
```

* delayは`kotlinx.coroutines`パッケージのsuspend関数
* 渡した引数ミリ秒だけ待つ
* beforeのあと10秒待ってafterがログに出る
* async/await系の非同期処理と違うのはawaitとする必要がないこと

jsだったら以下のAやB様な感じ。いちいちどこでawaitするか書く必要がない

```js
function async A() {
    console.log("before")
    await delay(10000)
    console.log("after")
}

function B() {
    console.log("before")
    delay(10000).then(() -> {
        console.log("after")
    })
}

function delay(ms) {
    return new Promise(resolve -> setTimeout(resolve, ms))
}
```

### suspend関数

suspend関数は内部でdelayとかのsuspend関数が呼べる。逆に言うと普通の関数からは呼べない。  
じゃあどうやって使うんだよに対応するのが先にやった`launch`

自分でsuspend関数を作るときは以下のような感じ

```kotlin
class MyClass {
    suspend fun methodAsync(): Unit =
        suspendCoroutine { continuation ->
            continuation.resume(Unit)
            // continuation.resumeWithException(Exception())
        }
}
```
```
uiScope.launch {
    Log.d("before")
    myClassInstance.methodAsync()
    Log.d("after")
}
```

* suspendCorontineの中でRetrofitで実装したAPIを呼んでコールバックでresumeするとかそんな感じ

やってることはJSのPromiseを返す関数やRXのstreamを返す部分

```js
function methodAsync() {
    return new Promise((resolve, reject) -> {
        resolve()
        // reject()
    })
}
```

他にもいろいろやり方がありそうな感じ

### 複数同時の処理

suspend関数から`kotlinx.coroutines`のasync関数を呼ぶ

```
uiScope.launch {
    // task1を開始
    val task1 = async(Dispatchers.IO) { delay(2000) }
    // task2を開始
    val task2 = async(Dispatchers.IO) { delay(10000) }

    task1.await()
    task2.await()

    // 待ったあとの処理...
}
```

* `async`はawaitできる`launch`みたいな感じ
* `Dispatchers.IO`はバックグラウンドでの実行の指示
  * `Dispatchers.IO`が最適なのかは未確認
* asyncで呼んだ時点で開始される
* task1とtask2をawaitしているが、 __task1の終了を待ってtask2ではない__ ので、10000ms後に待ったあとの処理が始まる

### 非同期処理とキャンセル

CoroutineScopeの生成を工夫することでキャンセルできるようにすることができる

以下のコードがonResumeで開始してonPauseでキャンセルする処理

```
val job = Job()
val uiScope = CoroutineScope(Dispatchers.Main + job)

override fun onResume() {
    super.onResume()

    uiScope.launch {
        Log.d("before")
        delay(10000)
        Log.d("after")
    }
}

override fun onPause() {
    super.onPause()
    job.cancel()
}
```

* CoroutineScopeの生成時にjobを一緒に与えると構造化できる

## 起動時に一定時間待つのと他の処理を一緒にやる

* アプリの最初のスプラッシュやりたいとき
* 画面が表示されたら2秒待ってメイン画面へ
* myClass.methodAsyncも呼んで、終了は待ってほしい
* 画面を離れたらちゃんとキャンセルしてメイン画面に勝手に行かない

```
class SplashActivity : AppCompatActivity() {
    private val job = Job()
    private val uiScope = CoroutineScope(Dispatchers.Main + job)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.splash_activity)
    }

    override fun onResume() {
        super.onResume()

        uiScope.launch {
            val taskList = ArrayList<Deferred<Unit>>()
            taskList.add(async(Dispatchers.IO) { myClassInstance.methodAsync() })
            taskList.add(async(Dispatchers.IO) { delay(2000) })
            taskList.forEach { it.await() }

            MainActivity.startActivity(this@SplashActivity)
            finish()
        }
    }

    override fun onPause() {
        super.onPause()
        job.cancel()
    }
}
```