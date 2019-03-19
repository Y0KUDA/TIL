import from qiita  
https://qiita.com/Y0KUDA/items/6e4ad1af39b8fb8c0319  

---
title: C++でクイックソートを4行で書きたかった
tags: C++ Haskell 日記 STL
author: Y0KUDA
slide: false
---
## 概要
C++でHaskellのクイックソートみたいなのを書きたくてvectorクラスを拡張してみた話。

## 経緯
なんとなくHaskellを勉強してたら、クイックソートを4行程度で書くことができるということを知って・・・

```haskell
qs [] = []
qs (p:xs) = qs l ++ [p] ++ qs h
 where l = [x | x <- xs, x < p ]
       h = [x | x <- xs, x >= p]
```

遅いし厳密にいうとクイックソートではないけど、読みやすくて理解しやすい。こういうコードをC++で書けないものかと挑戦したかった。

### コード解説
Haskell知らない人のためにコードを解説しておく。
#### `qs (p:xs)`
`qs`という関数がリストを受け取ったとき、最初の要素を`q`、残りのリストを`xs`と定義する。
#### `l = [x | x <- xs, x < p ]`
`xs`の`p`より小さい要素のリストを`l`と定義。
#### `h = [x | x <- xs, x >= p]`
`xs`の`p`以上の要素のリストを`h`と定義。
#### `= qs l ++ [p] ++ qs h`
`qs l` : 関数`qs`に`l`を渡す。
`[p]` : 要素が`p`のみのリスト。
`qs h` : 関数`qs`に`h`を渡す。
`++` : リストの結合。
#### `qs (p:xs) = qs l ++ [p] ++ qs h`
`qs`の戻り値は`qs l ++ [p] ++ qs h`。
#### `qs [] = []`
`qs`の引数が空のリストだった場合、空のリストを返す。


## C++で書く
C++でこのコードを実現するにはいくつか問題点があります。
vectorをソートすることを前提とすると、
1. フィルタがない。`if_copy`使うと冗長でわかりにくい。
2. 結合が面倒。`insert`を使うと冗長でわかりにくい。
ということで、vectorクラスを拡張してみた。

```cpp
#include <vector>
#include <functional>

template<typename T>
class myVector :protected std::vector<T> {
public:
	using std::vector<T>::vector;
	using std::vector<T>::push_back;
	using std::vector<T>::begin;
	using std::vector<T>::end;
	using std::vector<T>::insert;
	using std::vector<T>::size;
	using std::vector<T>::pop_back;
	using std::vector<T>::operator[];

	T pop_back() {
		T ret=(*this)[this->size() - 1];
		std::vector<T>::pop_back();
		return ret;
	}

	myVector<T> operator[] (std::function<bool(T&)> f) {
		myVector<T> v;
		for (T& x:*this) {
			if(f(x))v.push_back(x);
		}
		return v;
	}
	myVector<T>& operator<< (myVector<T>& v) {
		this->insert(this->end(),v.begin(),v.end());
		return *this;
	}
	myVector<T>& operator<< (T x) {
		this->push_back(x);
		return *this;
	}
};
```

演算子をオーバーロードしてフィルタと結合を簡単にできるようにしたかっただけなのに、こんなにコード書かなきゃいけないなんてどうかしてるよね。
次は拡張した機能を紹介していきます。

### filter機能
`[]`演算子をオーバーロードして実現しました。中に評価式を書くとfilterしてくれます。

```cpp
int main(void) {
	myVector<int> a = { 88,6,3,1,22,32,1,23,11 };
	auto isEven = [](int x) {return x % 2 == 0;};
	for (auto x : a[isEven])
		std::cout << x << ",";//88,6,22,
	return 0;
}
```
各要素を式に適用していって、Trueが返る要素のみが含まれたリストを生成します。
チェイン的なこともできます。

```cpp
int main(void) {
	myVector<int> a = { 88,6,3,1,22,32,1,23,11 };
	auto isEven = [](int x) {return x % 2 == 0;};
	auto over50 = [](int x) {return x > 50;};
	for (auto x : a[isEven][over50])
		std::cout << x << ",";//88,
	return 0;
}
```
一つ残念なことがあって、中にラムダ式を書くと括弧があふれて読みにくくなってしまいます。

```cpp
a[([](int x){return x%2==0;})];
```

### 結合
`<<`演算子をオーバーロードして実現しました。左のリストに右のリストを追加してくれます。

```cpp
int main(void) {
	myVector<int> a = { 88,6,3,1,22,32,1,23,11 };
	myVector<int> b = { 1,2,3,4,5,4 };
	a << b;
	for (auto x : a)
		std::cout << x << ",";//88,6,3,1,22,32,1,23,11,1,2,3,4,5,4,
	return 0;
}
```
要素を追加することもできます。

```cpp
int main(void) {
	myVector<int> a;
	a << 1 << 2 << 100 << 44 << 53;
	for (auto x : a)
		std::cout << x << ",";//1,2,100,44,53,
	return 0;
}
```

### クイックソート
こんな感じにvectorの機能を拡張していって完成したクイックソートがこのコードです。

```cpp
template<typename T>myVector<T> qs(myVector<T> v) {
	if (v.size() == 0) return v; //要素が空ならそのまま返す
	auto p=v.pop_back(); //要素を一つ取り出しピボットにする
	auto l = qs(v[([&](T& x) {return x < p;})]); //ピボットより小さい要素のリストをソートしてlに代入
	auto h = qs(v[([&](T& x) {return x >= p;})]); //ピボット以上の大きさの要素のリストをソートしてhに代入
	return l << p << h; //結合して返す
}
```
4行は難しかった。
どう考えても遅いけど、分かりやすいコードが書けたので満足です。


## コード
最後にコード全体を載せておきます。

```cpp
#include <iostream>
#include <vector>
#include <functional>

template<typename T>
class myVector :protected std::vector<T> {
public:
	using std::vector<T>::vector;
	using std::vector<T>::push_back;
	using std::vector<T>::begin;
	using std::vector<T>::end;
	using std::vector<T>::insert;
	using std::vector<T>::size;
	using std::vector<T>::pop_back;
	using std::vector<T>::operator[];

	T pop_back() {
		T ret=(*this)[this->size() - 1];
		std::vector<T>::pop_back();
		return ret;
	}

	myVector<T> operator[] (std::function<bool(T&)> f) {
		myVector<T> v;
		for (T& x:*this) {
			if(f(x))v.push_back(x);
		}
		return v;
	}
	myVector<T>& operator<< (myVector<T>& v) {
		this->insert(this->end(),v.begin(),v.end());
		return *this;
	}
	myVector<T>& operator<< (T x) {
		this->push_back(x);
		return *this;
	}
};

template<typename T>myVector<T> qs(myVector<T> v) {
	if (v.size() == 0) return v;
	auto p=v.pop_back();
	auto l = qs(v[([&](T& x) {return x < p;})]);
	auto h = qs(v[([&](T& x) {return x >= p;})]);
	return l << p << h;
}

int main(void) {
	myVector<int> a = {100,54,66,27,21,523,1,65,3,7,45,21,532,53,987,98,33};
	for (auto x : qs(a))
		std::cout << x << ",";
	return 0;
}
```
