# kotlin-parcelize

`Parcelable`なクラスの実装を自動でやってくれる  
`Serializable`は基本的に`Serializable`をimplementsして上げれば良いんだけど  
`Parcelable`はそれ用のメソッドを実装する必要がある  
それを勝手に実装してくれる

## 準備

https://developer.android.com/kotlin/parcelize
を読もう

`android-kotlin-extensions`の方はdeprecatedなので使わない

## 使い所

`SafeArgs`にわたすクラスとかで利用したりする
`String`とか`Int`とかで直接渡さないのは他で書く
`Serializable`ではなくこっち使ってるのはなんでなのかは忘れた
