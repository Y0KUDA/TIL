import from qiita  
https://qiita.com/Y0KUDA/items/d77275435fdf20b9a6ac

---
title: C / C++ 最初の一度だけ実行する関数
tags: C C++ マルチスレッド 黒魔術 組み込み
author: Y0KUDA
slide: false
---
### 概要
この関数何回か呼ばれるけど、この部分一回だけ実行したいなぁ・・・
ってことがまれによくあるので、解決方法を変態度が低い方から紹介していく。

### 1.グローバル変数フラグ
一番原始的な方法。
野蛮とも言う。
むしろ変態度が高いかもしれない。
名前空間を汚染するからやめたほうがいい。
というかやめろ。

```cpp
#include <iostream>
void f(){ // 一度だけ実行したい関数
  std::cout << "hello" << std::endl;
}

bool IsFirst = true;
main(){
  for(int i;i < 10;i++){
    if( IsFirst ){
      f();
      IsFirst = false;
    }
  }
}
```

### 2.static変数フラグ
一番一般的かもしれない。

```cpp
#include <iostream>
void f(){ // 一度だけ実行したい関数
  std::cout << "hello" << std::endl;
}

main(){
  static bool IsFirst = true; 
  for(int i;i < 10;i++){
    if( IsFirst ){
      f();
      IsFirst = false;
    }
  }
}
```

### 3.`std::call_once`を利用( c++のみ )
マルチスレッド対応。
お行儀がよい。

```cpp
#include <iostream>
#include <mutex>
void f(){ // 一度だけ実行したい関数
  std::cout << "hello" << std::endl;
}

main(){
  for(int i;i < 10;i++){
    static std::once_flag flag;
    std::call_once( flag, f );
  }
}
```

### 4.static変数の初期化を利用
マニアック過ぎ。
でも、一番スマート（だと思う）。

```cpp
#include <iostream>
void f(){ // 一度だけ実行したい関数
  std::cout << "hello" << std::endl;
}

main(){
  for(int i;i < 10;i++){
    // static変数の初期化は最初の一度しか実行されない。
    static bool CallOnce = [](){ f();return true; }(); // 右辺はラムダ式の実行
  }
}
```


### 5.マクロで定義
4で紹介した方法の応用。
`__LINE__`でユニークな名前を生成してるのがミソ。

```cpp
#include <iostream>
void f(){
  std::cout << "f" << std::endl;
}
void g(){
  std::cout << "g" << std::endl;
}
void h(){
  std::cout << "h" << std::endl;
}

#define CAT_(a,b) a##b
#define CAT(a,b) CAT_(a,b)
#define do_once(f) static bool CAT(DO_ONCE_,__LINE__)=[](){f();return true;}();

main(){
  for(int i;i < 10;i++){
    do_once( f ); // f,g,h がそれぞれ一回ずつ呼ばれる
    do_once( g );
    do_once( h );
  }
}
```
任意の引数の関数を呼ぶには、ラムダ式に包んでやるといい。

```cpp
do_once([](){ function( 1,2,3,"abc"); });
```


### 6.おまけ N回実行
5で紹介した方法の応用。

```cpp

#include <iostream>
void f(){
  std::cout << "f" << std::endl;
}

#define CAT_(a,b) a##b
#define CAT(a,b) CAT_(a,b)
#define do_times(f,n) static int CAT(DO_ONCE_CTR,__LINE__)=n;\
                       if(CAT(DO_ONCE_CTR,__LINE__)>0){\
                         f();\
                         CAT(DO_ONCE_CTR,__LINE__) --;\
                       }\

main(){
  for(int i;i < 10;i++){
    do_times( f, 5 );// f が5回実行される
  }
}
```