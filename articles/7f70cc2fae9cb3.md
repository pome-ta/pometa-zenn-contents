---
title: "objc_util からrubicon-objc への書き換え（Class宣言）"
emoji: "📲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python", "iOS", "objectivec", "Pythonista3", "rubicon"]
published: true
---

# objc_util からrubicon-objc へ乗り換える

Pythonista3 が、3.4 になったタイミングでobjc_util の[`ObjcBlock`](https://omz-software.com/pythonista/docs-3.4/py3/ios/objc_util.html#objc_util.ObjCBlock) の処理が落ちる。
block を使わずに実装する。といっても限度があるし、Python 側で処理。というよりも内部の問題ぽい（深くは調べて（られ）ない）ので、rubicon-objc へ移行することにした。

[objc_util — Utilities for bridging Objective-C APIs — Pythonista Documentation](https://omz-software.com/pythonista/docs-3.4/py3/ios/objc_util.html)
[beeware/rubicon-objc: A bridge interface between Python and Objective-C.](https://github.com/beeware/rubicon-objc)

よく使いつつ、毎回過去のコードを見直す内容をメモ的に書いていく。

## Class

### objc_util

[`objc_util.create_objc_class` | objc_util — Utilities for bridging Objective-C APIs — Pythonista Documentation](https://omz-software.com/pythonista/docs-3.4/py3/ios/objc_util.html#objc_util.create_objc_class)

```python
sub_class = objc_util.create_objc_class(name, superclass=NSObject, methods=[], classmethods=[], protocols=[], debug=True)
```

Python のClass 継承のように書けず、宣言っぽさも希薄な印象。
独自の書き方として、`Class` 宣言でラップするイメージで書いていた:

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

意味ない部分も多いかも、コードの視認性として、Class のブロックとしてまとめたかった。
docs にもあるが、無理にsubClass 化する必要はない。

### rubicon-objc

[Tutorial 2 - Writing your own class - Rubicon 0.4.7](https://rubicon-objc.readthedocs.io/en/stable/tutorial/tutorial-2.html)

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

デコレーション`@objc_method` をつける。initialize のmethod はreturn で`self` を返す。引数には型をつける。などなど、rubicon 上でのルール（エラーでしっかり警告が出る）はあるが、Python のClass 宣言である。

サンプルコード`NSObject` の部分を、継承させたいClass にすれば継承される。`ObjCClass('呼びたいclass')` で呼び出しておくことは必要。

[ObjCClass | rubicon.objc.api — The high-level Rubicon API - Rubicon 0.4.7](https://rubicon-objc.readthedocs.io/en/stable/reference/rubicon-objc-api.html#rubicon.objc.api.ObjCClass)

## delegate

### objc_util

### rubicon

## main Thread

## print 関係

:::message
追記予定
:::
