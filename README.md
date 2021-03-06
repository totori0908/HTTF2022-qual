# HTTF2022-qual

## 考察
- skillの初期値を問題文通りのランダムで初期化するか、適当な値もしくは0で初期化するか

- AHC003の上位解法が部分的に使えそう？  
    - 1st eivourさん 
    https://qiita.com/contramundum/items/b945400b81536df42d1a
    - 10th ymatsuxさん  
    https://ymatsux-cp-ja.blogspot.com/2021/06/atcoder-heuristic-contest-003-10.html

- 強い人が重要なタスクをやるのがよい。
    - 他の束縛度が大きい、要求のL2ノルムが大きいなど
    - 他の人のを見ると、L2ノルムが大きいものを強い人にやらせているように見える
    - 最後の方は仕事の遅い人に任せない方が良い
- 逆に、弱い人にはどうでもいい仕事を与える方が良い
    - 特に、束縛度の小さいものや要求スキルが小さいもの
    - 最後の方はできる仕事が残っていてもお休みする方が良い
    
- 依存度 * L2ノルムを評価値としてタスクの重要度を決定  
    - 依存度は子孫全部の数にする
    - 依存度 * x + L2ノルムは？
        - xを入力のRの値に応じて変更したい  
    - スキルのL2ノルムで人の強さを決定  
    - これで貪欲だと、人の適正を考慮できない

- Rとスコアの相関を見たい
    - Rが低い場合(seed 0)で2000を安定して出せるようにしたい
    - Rが高い場合はモデルを分ける、みたいなことが必要になるかもしれない

- 現状、スキルのL2ノルムが大きいか小さいかで判断しているだけで、得意不得意を判断できていない
    - 最悪遅くてもいいので、ここをどうにかすべき？
        - (何かしらを焼きなますなどのヒューリスティックより強い初期解が大事という方針)
- スキルの更新、タスクのベクトル方向に更新をするほうが精度良くなりそう？


- seed 132みたいにRが大きいと、ターン数がめっちゃかかってスコアが低くなる

- K個の特徴量と今までにやったタスク数をサンプル数Nとして、targetがそのタスクにかかった日数とする機械学習モデルを立てられると良い。
    - taskをjとして、i番目のskillパラメータをs_iで表すと, 
        - predicted Day = max(1, sum( max(0, d_i - s_i) ) + r_j)
        - 最小化したい損失関数 :
            - Loss = sum_j( abs( target_j - max(1, sum_i( max(0, d_i - s_i) ) + r_j) ) )
        - task_i の 乱数 r_i を特定するのは難しそうなので、Lossを簡易化すると、
            - Loss = sum_i( abs( target_i - sum_j( max(0, d_j - s_j) ) ) )
        - これ、absがあるとLossの計算が難しい？ので、二乗誤差にしたりするのか、なんかそれはそうだな
            - Loss = sum_i ( (target_i - sum_j( max(0, d_j - s_j) ) ) ^ 2 )
        - こうなると単純なLinear Regressionだよね
    - 新しいデータが追加されるたびに学習モデルを更新したい気持ちになる
    - NN もしくは古典的なモデルで、高速なもの

- パラメータ復元
    - スキル: 20人 * K = 400個
    - s の生成に用いる式の分子 randdouble(20, 60)  
    - 各タスクにおいて、かかる日数にノイズが入り、それが -3 ~ 3 日 (N個)
    - しかし、タスクにかかる日数のノイズ成分を予測したいとは思わない。
    -> 401個？
    - 分子のranddoubleを復元せず、400個でいいか？


- 標準正規分布の絶対値から生成される  
    -> https://qiita.com/toku_san/items/22bdce9b9efb92798c97  
    -> 平均 sqrt(2/pi), 分散 1 - sqrt(2/pi)  
    -> mean: 0.80, var: 0.20 (ほんまか？)


- もし正確に全員のスキルが分かっているとき、どのようにタスクを割り振るのが最適か
    - dousiyo~
    - なるべく全員が同時に動けるように. 序盤、中盤で手の空いている人がいるのは無駄