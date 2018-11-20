# TensorflowObjectAPI_Colab

gistでよかったんだけど、バージョン管理のしやすさ的な問題でこっちにする

## 機能
[Tensorflow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection)
をColaboratory上で学習の実行やモニタリングなどをできるようにした

データはローカルで[LabelImage](https://github.com/tzutalin/labelImg)を使って、作成したものと、[LabelBox](https://app.labelbox.com)を使って作成したものに現状では対応している

学習の結果はGoogleドライブに出力することが可能

## Appendix

せっかくだからColaboratory上で使えるTipsもちょこっと書いておく

### Tensorboardを起動
Colaboratory上でバックグランドジョブを簡単に走らせる方法がわからなかった...のでとりあえずこれで
```
from subprocess import Popen

cmd = "tensorboard --logdir [path/to/logdir] --port 6006"
proc = Popen( cmd,shell=True )
```
### Tensorboardを見る
ngrokというサービスを使うhttps://ngrok.com/  
localのポートをトンネリングして外部から見えるようにする(機密度が高いものには当然使っちゃだめ)
```
!mkdir -p /content/ngrok
%cd /content/ngrok
!wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
!unzip ngrok-stable-linux-amd64.zip

cmd = "./ngrok http 6006"
proc = Popen( cmd , shell=True)
```
ここまでで、トンネリングはできた。あとは実際に見るのに必要なURLを取得する  
json解析してもよかったんだけどもタグが変更させるたびに更新するのが嫌なので、正規表現で直接取得することにした  
おまけでQRコードも表示してスマホとかからチェックしやすくした
```
import requests
import qrcode
from PIL import Image
import matplotlib.pyplot as plt
import re

res = requests.get('http://localhost:4040')
m = re.search(r"(https:\/\/.*?ngrok.io)", res.text)
if m:
    print("Watch runing status bellow")
    url = m.group(1)
    print(url)
    img = qrcode.make(url)
    plt.axis('off')
    plt.imshow(img)
    plt.show()
```

### Googleドライブにデータをバックグラウンド同期
Colaboratoryサーバー常にマウントしたGoogleドライブはアクセススピードが非常に遅いので、学習中などにそこにアクセスさせたりすると実行時間が遅くなるのがよくない  
なので、簡単に定期的にバックアップをとる仕組みを作る  
googleドライブがどこかしらにマウントされている前提
```
from subprocess import Popen
src_path = "/path/on/colaboratory/server"
dst_path = "/path/to/google/drive/directory"
cmd = "/bin/bash -c 'while sleep 30; do rsync -a \"{0}\" \"{1}\"; done'".format(src_path, dst_path)
proc = Popen( cmd,shell=True )
```

### Colaboratoryセル内にボタンとかを作る
気が向いたらそのうち書く  
List Available Modelsのセルの中身を見ればなんとなくわかるかも
