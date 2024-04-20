---
title: "【class】objc_util からrubicon-objc へ移行【宣言】"
emoji: "📲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python", "iOS", "objectivec", "Pythonista3", "rubiconobjc", ]
published: true
---

## objc_util からrubicon-objc へ乗り換える

Pythonista3が、3.4になったタイミングでobjc_utilの[`ObjcBlock`](https://omz-software.com/pythonista/docs-3.4/py3/ios/objc_util.html#objc_util.ObjCBlock) の処理が落ちる。
blockを使わずに実装する。といっても限度があるし、Python側で処理。というよりも内部の問題ぽい（深くは調べて（られ）ない）ので、rubicon-objcへ移行することにした。

[objc_util — Utilities for bridging Objective-C APIs — Pythonista Documentation](https://omz-software.com/pythonista/docs-3.4/py3/ios/objc_util.html)

@[card](https://omz-software.com/pythonista/docs-3.4/py3/ios/objc_util.html)
[beeware/rubicon-objc: A bridge interface between Python and Objective-C.](https://github.com/beeware/rubicon-objc)
@[card](https://github.com/beeware/rubicon-objc)

よく使いつつ、毎回過去のコードを見直す内容をメモ的に書いていく。

今回はClass宣言:

## 書き換え

### objc_util

[`objc_util.create_objc_class` | objc_util — Utilities for bridging Objective-C APIs — Pythonista Documentation](https://omz-software.com/pythonista/docs-3.4/py3/ios/objc_util.html#objc_util.create_objc_class)

```python
sub_class = objc_util.create_objc_class(name, superclass=NSObject, methods=[], classmethods=[], protocols=[], debug=True)
```

PythonのClass継承のように書けず、宣言っぽさも希薄な印象。
独自の書き方として、`Class` 宣言をラップするイメージで書いていた:

```python: 雑な例.py
class SubClass:
    def __init__(self, *args, **kwargs):
        self.インスタンスにさせたいオブジェクト: '型'   # 何を呼び出したかったか忘れないため

    def override_hoge(self):
        # メソッド内に、objc で定義したいメソッドを書く
        def hoge_fuga_(_self, _cmd, _fuga):
            # （大体）ポインタで来るので、ObjCInstance でラップする
            # `self だとClass 内で衝突してしまうので、`this`
            this = ObjCInstance(_self)

        def foo_bar_baz_(_self, _cmd, _bar, baz):
            # objc のメソッドとして、存在するものもオーバーライドできる
            this = ObjCInstance(_self)

        _methods = [
            hoge_fuga_,
            foo_bar_baz,
        ]
        create_kwargs = {
            'name': 'onamae',    # 変数名と揃える
            'superclass': 使いたいClass,    # 事前に、`ObjCClass('使いたいClass')` しておく
            'methods': _methods,
        }
        onamae = create_objc_class(**create_kwargs)
        self.インスタンスにさせたいオブジェクト = onamae

    def _init(self):
        self.override_hoge()
        return self.インスタンスにさせたいオブジェクト

    @classmethod
    def new(cls, *args, **kwargs):
        _cls = cls(*args, **kwargs)
        return _cls._init()

```

意味ない部分も多いかも、コードの視認性として、Classのブロックとしてまとめたかった。
docsにもあるが、無理にsubClass化する必要はない。

### rubicon-objc

[Tutorial 2 - Writing your own class - Rubicon 0.4.7](https://rubicon-objc.readthedocs.io/en/stable/tutorial/tutorial-2.html)
@[card](https://rubicon-objc.readthedocs.io/en/stable/tutorial/tutorial-2.html)

なんと、宣言できる。

```python:Tutorial2.py
from rubicon.objc import NSObject, objc_method


class Handler(NSObject):
    @objc_method
    def initWithValue_(self, v: int):
        self.value = v
        return self

    @objc_method
    def pokeWithValue_andName_(self, v: int, name) -> float:
        print("My name is", name)
        return v / 2.0
```

デコレーション`@objc_method` をつける。initializeのmethodはreturnで`self` を返す。引数には型をつける。など、rubicon上でのルール（エラーでしっかり警告が出る）はあるが、PythonのClass宣言です。

サンプルコード`NSObject` の部分を、継承させたいClassにすれば継承される。`ObjCClass('呼びたいclass')` で呼び出しておくことは必要。

[ObjCClass | rubicon.objc.api — The high-level Rubicon API - Rubicon 0.4.7](https://rubicon-objc.readthedocs.io/en/stable/reference/rubicon-objc-api.html#rubicon.objc.api.ObjCClass)
@[card](https://rubicon-objc.readthedocs.io/en/stable/reference/rubicon-objc-api.html#rubicon.objc.api.ObjCClass)

## 所感

rubiconのPythonicなClass宣言は、objc_utilでのsubClass生成よりもハードルは低い印象。
しかし、rubiconでは`__init__` が隠蔽されていたり、デコレータをつけたり。仕様の理解度合いで、細かい実装の可能性が変わっていきそうな予感がしている。
一方objc_utilは、素朴で無骨な「なんとかPythonで操作できる状態のオブジェクト」を提供している印象があり、状態の管理をどっち（モジュールか自分自身）が持つかの認識判断が必要だと感じた。

rubiconは現在もアクティブに更新され続けています。objc_utilのソースとの差異（過去のrubiconをベースにobjc_utilが作られている）を眺めつつ、過去から今までの変遷を追っかけることで、理解が深まりますね。

## delegate

### objc_util

### rubicon

## main Thread

## print 関係

:::message
追記予定（分割するかも）。
:::
