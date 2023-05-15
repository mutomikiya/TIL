```
//スプレッドシートのメニューバーから起動できるようにする
function onOpen() {
  SpreadsheetApp
    .getActiveSpreadsheet()
    .addMenu('Slack送信', [
      {name: '送信する', functionName: 'startFunction'},
      {name: '送信テストする', functionName: 'startFunctionTest'}
    ]);
};

//本番用（「集計」シートに記載してあるメンバーにSlackを送信する）
function startFunction(){
  let msg = Browser.msgBox('確認','支払明細をSlackで送信しますか？', Browser.Buttons.OK_CANCEL);
  if (msg == 'ok'){
    var totalSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('集計');
    sendingSlack(totalSheet)
  };
};

//テスト用（「テスト送信用」シートに記載してあるメンバーにSlackを送信する）
function startFunctionTest(){
  let msg = Browser.msgBox('確認（テスト）','支払明細をSlackで送信テストしますか？', Browser.Buttons.OK_CANCEL);
  if (msg == 'ok'){
    var totalSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('テスト送信用');
    sendingSlack(totalSheet);
  };
};

//以下は本番用、テスト用ともに共通
function sendingSlack(totalSheet){
  var detailSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('明細');
  var paymonth = totalSheet.getRange(1,2).getValue();
  var members = getMembers(totalSheet);
  var paymonthdetails = getPayMonthDetails(detailSheet,paymonth);
  sendingMessage(paymonth,members,paymonthdetails)
};

function getMembers(totalSheet) {
  var last_row = totalSheet.getLastRow();
  var members = [];
  for(let i=4; i<=last_row; i++){
    var name = totalSheet.getRange(i,2).getValue();
    var sum = totalSheet.getRange(i,3).getValue();
    var slackID = totalSheet.getRange(i,4).getValue();
    if(name !== '' && slackID !== ''){
        var member = {"name":name,"slackID":slackID,"sum":sum};
        members.push(member);
    };
  };
  return members;
};

function getPayMonthDetails(detailSheet,paymonth){
  var payMonthDetails = [];
  var last_row = detailSheet.getLastRow();
  for (var i = 1; i<=last_row;i++){
    if(detailSheet.getRange(i,10).getValue() == paymonth){
      var name = detailSheet.getRange(i,2).getValue();
      var media = detailSheet.getRange(i,3).getValue();
      var type = detailSheet.getRange(i,4).getValue();
      var detail = detailSheet.getRange(i,6).getValue();
      var money = detailSheet.getRange(i,7).getValue();
      var tax = detailSheet.getRange(i,8).getValue();
      var paymoney = detailSheet.getRange(i,9).getValue();

      var payDetail = {
        '名前':name,'媒体':media,'名目': type,'詳細':detail ,'金額':money, '源泉徴収':tax, '支払金額':paymoney
        };
      payMonthDetails.push(payDetail);
    };
  };
  return payMonthDetails;
};


function getMembersDetails(paymonthdetails,name){
  var membersDetails = [];
  var last_objevct = paymonthdetails.length;
  for (var i = 0; i<last_objevct;i++){
    if(paymonthdetails[i]['名前'] == name){
      membersDetail = {
        '媒体':paymonthdetails[i]['媒体'],'名目': paymonthdetails[i]['名目'],'詳細':paymonthdetails[i]['詳細'],'金額':paymonthdetails[i]['金額'], '源泉徴収':paymonthdetails[i]['源泉徴収'], '支払金額':paymonthdetails[i]['支払金額']
      }
      membersDetails.push(membersDetail);
    };
  };
  return membersDetails;
};

function sendingMessage(paymonth,members,paymonthdetails){
  members.forEach(function(member){
    var name = member['name'];
    var slackID = member['slackID'];
    var sum = member['sum'];
    details = getMembersDetails(paymonthdetails,name);
    var text = '```【原稿料・交通費等支払明細（'+name+', '+paymonth+'）】\n\n';

    details.forEach(function(detail){
      text += '媒体：'+detail['媒体']+' 名目：'+detail['名目']+'\n詳細：' +detail['詳細']+'\n金額：'+detail['金額']+'円'+' 源泉徴収：'+detail['源泉徴収']+'円'+'\n- 支払金額：'+detail['支払金額']+'円'+'\n\n';
    });

    var payload = {
      'username': '支払明細',
      'text': text+'合計： '+sum+'円```',
      'channel': '@'+slackID
    };
    var options = {
      'method': 'post',
      'contentType': 'application/json',
      'payload': JSON.stringify(payload)
    };

    var url = 'webhook url';
    UrlFetchApp.fetch(url, options);
  });
  Browser.msgBox('支払明細のSlack送信が完了しました。');
};

```
