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


## 複数のNavigation

`Navigation`のXMLは1つである必要はないので、アプリケーション内に複数あっても問題ない  
例えば、2021時点のPlayStoreのようなアプリで考えると、大きく2つの`Navigation`になる  
`BottomNavigation`をタップしたときの遷移のNavigationと  
アプリをタップしたときに遷移するアプリの詳細や設定画面への遷移のような画面全体の`Navigation`のような感じ  
ただし、`BottomNavigation`とバックキーの挙動には注意が必要  
私的には。バックキーでアプリが終了する前に`BottonNavigation`の一番左にフォーカスが移動するタイプが好き  
場合によっては、`Navigation`を使用せずに実装することも視野に入れよう


## SafeArgsの利用

`SafeArgs`を利用して`Navigation`に`Action`を記述すると、`XXXDirections`というクラスが生成される  
`arguments`も合わせて、idを利用した遷移より安全なので、利用すると良い


## NavigationとDialogFragmentの注意点

`DialogFragment`も記述できるが少々工夫が必要だったので、無理に利用しないほうが良いかも  
`targetFragment`を利用して`DialogFragment`の結果で遷移するとクラッシュしたりした記憶がある  
`currentDestination`との整合性のとり方が意外と難しい  
その他、`DialogFragment`が多重に表示されたりする  
私の設計では、`DialogFragment`は通常通りに`FragmentManager`を利用して表示するほうが調整しやすかった  
`Navigation`と混在しても問題はあまり発生しない模様

## その他

`DeepLink`とかの機能もあるけど利用したことはない  
