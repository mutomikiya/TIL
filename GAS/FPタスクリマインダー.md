```
//スプレッドシートのメニューバーからリマインダーの確認/起動/停止ができるようにする
function onOpen() {
  SpreadsheetApp.getActiveSpreadsheet().addMenu('リマインダー',[
    {name: 'リマインダーの状態を確認する', functionName: 'confirmTrigger'},
    {name: 'リマインダーを起動する', functionName: 'startSetTrigger'},
    {name: 'リマインダーを停止する', functionName: 'deleteAllTriggers'}
    ])
}
    
//次の日に実行するnotifySlackがセットされているか確認する
function confirmTrigger(){
  var exist = false;
  var triggers = ScriptApp.getProjectTriggers();
  for (var i = 0; i<triggers.length; i++){
    var trigger = triggers[i];
    if(trigger.getHandlerFunction() == "notifySlack"){
       Browser.msgBox("リマインダーは起動中です");
       var exist = true;
       break;
    }
  }
  if(!exist){
  Browser.msgBox("リマインダーは停止しています");
  }
}
      
//全てのトリガーを削除する
function deleteAllTriggers(){
   var triggers = ScriptApp.getProjectTriggers();
   for (let i = 0; i < triggers.length; i++) {
     var trigger = triggers[i];
     ScriptApp.deleteTrigger(trigger);
   }
  Browser.msgBox("リマインダーを停止しました");
}

//setTriggerを起動する
function startSetTrigger(){
  setTrigger();
  Browser.msgBox("リマインダーを起動しました");
}

//次の日の8:30にnotifySlack関数を実行する
function setTrigger(){
  const time = new Date();
  time.setDate(time.getDate()+1);
  time.setHours(8);
  time.setMinutes(30);
  ScriptApp.newTrigger('notifySlack').timeBased().at(time).create();
};

//Slackに送る
function notifySlack() {
  
  //次の日の8:30に再びnotifySlackが実行されるようにセット
  setTrigger();
  
  //getTasks関数を呼び出してタスクを取得する
  var tasks = getTasks();
  
  if(tasks.length !== 0){
    var text = '【入稿前スケジュール】\n'
    
    //保護者版タスクが存在し、かつそれが高校生版タスクと異なる場合は分けて書く
    if(tasks['task_parent'] && tasks['task_student'] !== tasks['task_parent']){
      Logger.log("true")
      text += '（高校生版）\n'+tasks['task_student']+'\n'+'（保護者版）\n'+tasks['task_parent']
    }
    //それ以外の場合は高校生版タスクのみ書く
    else{
      text += tasks['task_student']
    };
    
    //以下はSlack送信のための情報
    var payload ={
      'username': '**',
      'text': text,
      'channel': '**'
    };
    var options = {
      'method': 'post',
      'contentType': 'application/json',
      'payload': JSON.stringify(payload)
    };
    
    var url = '**';
    
    //送信
    UrlFetchApp.fetch(url, options);
  };
};

//タスクを取得する
function getTasks(){
  
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sh_submit = ss.getSheetByName('入稿前スケジュール');
  
  
  //実行日の日付を取得
  var date = new Date();
  var day = 3;//date.getDate();
  var month = 11;date.getMonth();
  
  var last_row = sh_submit.getLastRow()
  var task = {};
  
  //タスクの日付が書いてあるA列を順番に調べる
  for(var i = 2; i <= last_row; i++){
    var daterow = sh_submit.getRange(i,1).getValue();
    
    //もし関数の実行日とタスクの日付が一致した場合、D、E列の内容を取得する
    if(daterow.getDate()==day && daterow.getMonth()==month){
      
      var task_student = sh_submit.getRange(i,4).getValue();
      var task_parent = sh_submit.getRange(i,5).getValue(); 
      
      task["task_student"] = task_student;
      task["task_parent"] = task_parent;
      break;
      
    } 
  }; 
  return task;
};

