---
marp: true
# dark theme
class: invert
---
<!-- headingDivider: 1 -->

# About this contents

GitHub Pages: [https://inoue-ryoki.github.io/rspec-study/](https://inoue-ryoki.github.io/rspec-study/)

Repository: [https://github.com/inoue-ryoki/rspec-study - GitHub](https://github.com/inoue-ryoki/rspec-study)

# Rspecの処理の順番

describe,context,before,it,let,subject等いろいろあるので、処理の順番を整理したい
よく使用する以下の辺りを調べる

①RSpec.configure do |config| の中

②RSpec.describe "Slips", type: :request doの中

③describe (説明)

④context (文脈)

⑤before

⑥subject (対象)

⑦it

⑧let

# まずはシンプルに

spec/rails_helper.rb

```rb
RSpec.configure do |config|
  p "configure"
```

spec/requests/slips_spec.rb

```rb
require "rails_helper"

RSpec.describe "Slips", type: :request do
  p "RSpec.describe"
  describe "GET /index" do
    p "describe"
  end
end
```
# 出力結果

```rb
"configure"
"RSpec.describe"
"describe"
```

の順番。上から順番で想定通り

# subject,it,contextを追加

```rb
require "rails_helper"

RSpec.describe "Slips", type: :request do
  p "RSpec.describe"
  describe "GET /index" do
    p "describe"
    before do
      p "before"
    end

    subject do
      p "subject"
    end

    it do
      subject
      p "it"
    end

    context do
      p "context"
    end
  end
end
```

# 出力結果

```
"configure"
"RSpec.describe"
"describe"
"context"
"before"
"subject"
"it"
```

describeの次はcontextが走る（contextを下に書いていても）

# letについて

letはRspecでインスタンス変数の代わりに使える便利なもの

```rb

@params = { name: '佐藤' }

↓

let(:params) { { name: '佐藤' } }
```

letはメリットは多いらしいが、エラーが出る時あるので今回よく確認してみる

# letはcontextで参照できない？

今までletが使えない時はその都度エラーみて修正していた。
今回そういう場合でletが使えないのかみて見る

```rb

  describe "GET /index" do
    let(:sample_let) { 1111 }
    before do
      p sample_let
    end

    subject do
      p sample_let
    end

    it do
      subject
      p sample_let
    end

    context do
      p sample_let
    end
  end

```

# 出力結果

```rb
Failure/Error: p sample_let
  `sample_let` is not available on an example group (e.g. a `describe` or `context` block). It is only available from within individual examples (e.g. `it` blocks) or from constructs that run in the scope of an example (e.g. `before`, `let`, etc).
```


contextの所でエラー。letはcontextでは使えない？
ちなみにcontextをコメントアウトした結果の出力
before,subject,it では正常にletが出力されている


# letはbefore内では定義できない？

以下のようにbeforeやsubjectでletを書くとエラーが出る。describeやcontextの中だけでletは使える。
インスタンス変数はbeforeでも使えるのに、この辺りが混乱しそう。。

```rb
  describe "GET /index" do
    before do
      let(:before_let) { "before_let!" }  # エラー
    end

    subject do
      let(:subject_let) { "subject_let!" } # エラー
    end

    context do
      let(:context_let) { "context_let!" } # 正常
      it do
        subject
        p before_let
        p subject_let
        p context_let
      end
    end
  end
```

# letは遅延評価

letは呼び出された時に実行されるので、エラー出る時がある。
遅延評価がテストの失敗の原因になっている場合は let! を使うとexample の実行前に定義した値が作られる。

参考
https://qiita.com/jnchito/items/42193d066bd61c740612

https://qiita.com/jnchito/items/cdd9eef2ed193267c651
