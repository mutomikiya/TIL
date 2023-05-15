# やりたいこと
- スプレッドシート上にボタンを設置しそれを押すと交通費経費等集計シートから各人の給与明細を作成しDMに送信する機能を実装したい
# 設計
- 「集計」タブで給与が発生する人の名前、SlackID、支払い月情報を取得する
  - 二次元配列にして[[name,slackID],[name,slackID]]のような形で取得したい
  - 支払い月情報を取得しておく
- 作成した配列一人一人に対して「明細」タブから給与情報を取得してSlackに送信する
  - 配列１つ１つに対して「明細」タブで名前と支払い月が一致する給与発生項目を探し出し、その項目と金額・源泉徴収額・支払額を新たな配列に入れる
  - SlackIDを元にDMに送信する
  - これを給与が発生する全員に対して行う
# 必要な関数
- 集計タブのシート情報を取得
  ```
  function getMembers() {
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    var totalSheet = ss.getSheetByName('集計');
    var last_row = ss.getLastRow();
    var Members = [];
    for(let i=4; i<=last_row; i++){
      var name = totalSheet.getRange(i,2).getValue();
      var sum = totalSheet.getRange(i,3).getValue();
      var slackID = totalSheet.getRange(i,4).getValue();
      if(slackID !== ''){
        if(sum > 5000){
          var Member = {"name":name,"slackID":slackID,"sum":sum};
          Members.push(Member);
        }
        else{
          break;
        };
      };
    };
    return Members;
  };
  ```
 - 名前と支払い月を受け取ってそれをもとに支払項目を取得する
   ```
   function getDetail(name,paymonth){
      var Details = [];
      var detailSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('明細');
      var last_row = detailSheet.getLastRow();
      for (var i = 1; i<last_row;i++){
        if(detailSheet.getRange(i,2).getValue() == name && detailSheet.getRange(i,10).getValue() == paymonth){
          var media = detailSheet.getRange(i,3).getValue();
          var type = detailSheet.getRange(i,4).getValue();
          var detail = detailSheet.getRange(i,6).getValue();
          var money = detailSheet.getRange(i,7).getValue();
          var tax = detailSheet.getRange(i,8).getValue();
          var paymoney = detailSheet.getRange(i,9).getValue();

          var payDetail = {'媒体':media, '名目': type,'詳細':detail ,'金額':money, '源泉徴収':tax, '支払金額':paymoney};
          Details.push(payDetail);
        };
      };
      return Details;
   };
 - 上記の二つの関数を使って、支払いが発生するメンバーそれぞれに対して支払項目を取得しSlackに送信する
   ```
   function notifySlack(){
      var paymonth = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('集計').getRange(1,2).getValue();
      var members = getMembers();
      members.forEach(function(member){
        var name = member['name'];
        var slackID = member['slackID'];
        var sum = member['sum'];
        details = getDetail(name,paymonth);

        var text = '【原稿料・交通費等支払明細（'+name+', '+paymonth+'）】\n';
        details.forEach(function(detail){
          text += '媒体：'+detail['媒体']+' 名目：'+detail['名目']+'\n 詳細：' +detail['詳細']+'\n 金額：'+detail['金額']+'円'+' 源泉徴収：'+detail['源泉徴収']+'円'+'\n - 支払金額：'+detail['支払金額']+'円'+'\n\n';
        });
        var payload = {
          'username': '支払明細',
          'text': text+'合計： '+sum+'円',
          'channel': '@'+slackID
        };
        var options = {
          'method': 'post',
          'contentType': 'application/json',
          'payload': JSON.stringify(payload)
        };

        var url = '（incomming webhookのURL）';
        UrlFetchApp.fetch(url, options);
      });
      Browser.msgBox('支払明細のSlack送信が完了しました。');
   };

 - スプレッドシート上にボタンを配置し、それを押すと確認が出て、OKを押すとSlackに送信する関数が実行されるようにする
    ```
    function startFunction(){
      let msg = Browser.msgBox('確認','支払明細をSlackで送信しますか？', Browser.Buttons.OK_CANCEL);
      if (msg == 'ok'){
        notifySlack();
      };
    }
    ```   
