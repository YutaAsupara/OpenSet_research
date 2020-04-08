# 「識別モデルの構造も，結果をできるだけ変えずに」OpenSetRecogntionに対応するモデルを実現したい

# OpenSetRecognition

　Scheirerらの[1]では
> —To date, almost all experimental evaluations of machine learning based recognition algorithms in computer vision have taken the form of “closed set” recognition, whereby all testing classes are known at training time. A more realistic scenario for vision applications is “open set” recognition, in which incomplete knowledge of the world is present at training time, and unknown classes can be submitted to an algorithm during testing.

　と述べられており，現実世界に適用できる認識モデルを作ることは難しい課題となっている．実際に，画像を用いて犬と猫を識別するモデルを作成した場合，そこにタヌキの画像が入力されても，犬と猫のどちらのクラスに属しているかを出力する．また，工業製品の完成品を画像として入力し，作られた製品が良品か不良品であるかを判別するモデルを作る際にも，あらかじめ不良品のパターンを想定し，学習データを作成することになる．しかし，全ての不良を想定するのは難しく，運用時にうまくいかない要因となる．

　この時，モデルに未知を検出する構造が導入されていれば，未知のクラスのデータが入ってきた場合に誤って既知のクラスに分類することを避けることができる．未知を未知として扱える構造は，誤りを減らすだけでなく，未知として正しく判別できれば，未知と判別されたデータを収集して，新たなクラスを定義することも考えられる．

![Object 138](https://user-images.githubusercontent.com/17122464/78804975-ccec1e80-79fb-11ea-85b2-a144a7a88f25.jpg)
図1. オープンワールド認識のための考え方，モデルが未知を未知として検出できれば，新しいクラスとしてデータを作り，認識器を機能を増やすことにも繋がる．（画像はBendaleらの[2]より引用 ）

# OpenMax

　Deep Learningの分野でOpen Set Recognitionを達成するための方法論の一つとして，Bandaleらの[2]は多くの研究で用いられている．

　OpenMaxを活用したDeep Neural Networkでは，
学習時にはSoftMaxを使用して通常の分類問題を解き，推論時にSoftMaxをOpenMaxに置き換えることで，未知の未知に対して対処することを期待する．OpenMaxでは出力される層のベクトルをActivation Vectorとし，その平均との距離をワイブル分布でフィッティングする．推論時に入力されるデータは，先ほどの平均との距離を計算して，フィッティングした分布に従うか否かで未知のクラスのデータであるか，既知のクラスのデータであるかを判別する仕組みとなっている．

　OpenMaxを使うことで，[2]では未知のデータの検出についての可能性を論じているが，Softmaxをそのまま使う状態のモデルに比べて，学習していたタスクの精度自体は低下していることが懸念されている．

# Extreme Value Machine
　Extreme Value Machine[3]では，Extreme Value Theoryを用いて，特徴マップから得るmargin distributionを基に閾値処理を行なって，未知を検出する仕組みを用いる．
![Object 143](https://user-images.githubusercontent.com/17122464/78805072-ed1bdd80-79fb-11ea-981d-9a6712cc50fd.jpg)
![Object 141](https://user-images.githubusercontent.com/17122464/78805085-f1e09180-79fb-11ea-9caf-d97b0d46504e.jpg)
![Object 142](https://user-images.githubusercontent.com/17122464/78805077-eee5a100-79fb-11ea-95cb-339571084fed.jpg)

このモデルはデータ数が増える意味でのIncremental な状況にも対応していて，Open Set RecognitionでありながらOpen World Recognitionに対応する考え方になっている．

　しかしながら，データ数の増えていった時のデータの代表値を残していく方法にもっと工夫が必要とも考えられる．

# 認識モデルに影響を与えないモデルの構築を目指して

　紹介した例以外にも，Open Set Recognitionへの挑戦は増えており，ついにSurvey論文も出るようになったため[4]，この問題に対する関心は上がってきていると考えられるが，[4]で述べられる方法論のほとんどは，元々の識別モデルの出力層をOpenMaxに改造したり，EVMでは，識別モデルの特徴量（DeCAF[5]など）から別の識別器に渡すような仕組みで計算を行なっていた．言いかえれば，推論の結果が，Open Set Recognitionに対応する前よりも，対応後は既存のタスクの結果が悪くなることがあるという事になる．そのため，例えばOpen Set Recognition「識別モデルの結果を変えずに」実現することはできないだろうかと考えており，これから行う研究ではその実現を目指したい．

### 参考文献
[1]Scheirer, W. J. Rocha, A. Sapkota, A. and Boult, T. E. Towards open set recognition. IEEE T-PAMI vol. 36, 2013. 

[2]A. Bendale and T. E. Boult, “Towards open set deep networks,”Proceedings of the IEEE conference on computer vision and pattern recognition, pp. 1563–1572, 2016.

[3]E. M. Rudd, L. P. Jain, W. J. Scheirer and T. E. Boult, “The extreme value machine,” IEEE Transactions on Pattern Analysis and Machine Intelligence, vol. 40, no. 3, pp. 762–768, 2018.

[4]C Geng, S Huang, and Songcan Chen, "Recent Advances in Open Set Recognition: A Survey" arXiv:1811.08581

[5]J. Donahue, Y. Jia, O. Vinyals, J. Hoffman, N. Zhang, E. Tzeng, and T. Darrell. Decaf: A deep convolutional acti- vation feature for generic visual recognition. arXiv preprint arXiv:1310.1531, 2013.
