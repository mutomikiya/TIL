# git push origin masterについて
- orijinで送り先を指定して、masterでどのブランチにpushするか指定する
- masterは本来（master:master）であり、これを省略して書いている
- 基本的に現在いるローカルブランチと同じ名前のブランチにプッシュされる
  - ただ送り先を指定することも可能
  - git push origin feature/aws:masterと　入力すればローカルのfeature/awsというブランチからリモートのmasterというブランチにpushすることもできる
  - ただ事故の原因にもなるし、普通はやらない
- git push origin feature/awsだけ打てば、ローカルのfeature/awsブランチからリモートのfeature/awsブランチにpushされる（リモートに同じ名前のブランチが
存在しない場合は勝手に生成される

- git push orijin masterのmasterの部分でどのブランチをプッシュするか指定する代わりに、他の方法で指定してこの部分を省略することもできるが、自分がどの
ブランチで作業しているのかを把握するという意味でもきちんと毎回打った方が良い
