# Navigation

Android用の画面遷移ライブラリ  
XMLで画面と画面間の接続を定義し、コードから呼び出して利用する  
`Activity`で`startActivity`,`startActivityForResult`や`Fragment`で`FragmentManager`を利用する方法だと、  
各`Activity`と`Fragment`を確認しないと画面同士の繋がりを把握できないが、  
`Navigation`を利用する場合XMLファイルを確認するだけで把握することができる


## 準備

https://developer.android.com/guide/navigation/navigation-getting-started
に従う

`SafeArgs`と併用すると便利


## 使い所

基本的に画面遷移はこれで実装すると良い  
`Activity`や`FragmentManager`は扱いがめんどくさいので`Activity`を1つだけ用意し、  
`Fragment`の遷移で画面遷移を実装するのが楽  
SingleActivityとか呼ばれるパターンらしい。Jake Warthonさんもこれがいいって言ってたらしい  
`ActivityContext`のリークとかにあまり気を使わなくてよくなるのも良い  
20以上は画面があるようなアプリで利用してみたが、特に大きな問題なく利用できた

ChromeCastを利用する場合は注意が必要  
[Googleのガイドライン](https://developers.google.com/cast/docs/design_checklist) での必須要素として、通知領域の実装と通知領域からのExpandedControllerの実装が必要とされている  
これを実装するためのAPIは`setExpandedControllerActivityClassName`となっていて、起動されるActivity名を指定することになる  
SingleActivityの場合、特定の画面を指定できないので、特別な調整が必要となる


## 複数のNavigation

`Navigation`のXMLは1つである必要はないので、アプリケーション内に複数あっても問題ない  
例えば、2022時点のPlayStoreのようなアプリで考えると、大きく2つの`Navigation`になる  
`BottomNavigation`をタップしたときの遷移のNavigationと  
コンテンツをタップしたときに遷移する詳細画面や設定画面への遷移のような画面全体の`Navigation`のような感じ  
ただし、`BottomNavigation`とバックキーの挙動には注意が必要  
私的には。バックキーでアプリが終了する前に`BottonNavigation`の一番左にフォーカスが移動するタイプが好き  
場合によっては、`Navigation`を使用せずに実装することも視野に入れる　


## SafeArgsの利用

`SafeArgs`を利用して`Navigation`に`Action`を記述すると、`XXXDirections`というクラスが生成される  
`arguments`も合わせて、idを利用した遷移より安全なので、利用すると良い


## NavigationとDialogFragmentの注意点

`DialogFragment`も記述できるが少々工夫が必要だったので、無理に利用しないほうが良いかもしれない  
`targetFragment`を利用して`DialogFragment`の結果で遷移するとクラッシュしたりした記憶がある  
`currentDestination`との整合性のとり方が意外と難しい  
その他、`DialogFragment`が多重に表示されたりする  
私の設計では、`DialogFragment`は通常通りに`FragmentManager`を利用して表示するほうが調整しやすかった  
`Navigation`と混在しても問題はあまり発生しない模様


## 画面遷移と関数

画面遷移を一種の関数として考えると、わかりやすくなる場合がある  
コンテンツの購入画面やログイン画面、ダイアログ的に利用する画面がそれに当たることが多い  
コンテンツの購入画面は購入対象のコンテンツのIDを必要とし、コンテンツの購入結果を遷移元画面に返却すると考えるとわかりやすい  
擬似的にコードっぽくすると以下のようになる  

```Kotlin
fun navigateToPurchase(contentId: String): Boolean
```

`contentId: String`の部分は、`Navigation`と`SafeArgs`を利用すると作りやすい  
`Boolean`を返却する部分は、先に書いた方法では作成できないので、  
`NavBackStackEntry#savedStateHandle`を利用すると、遷移先のFragmentから遷移元に値を返せるようなので、  
これを利用すると良さそう

NavBackStackEntryを使った実装は本番で利用したことがないので、使うことがあればまた書きたい


## その他

`DeepLink`とかの機能もあるけど利用したことはない  
