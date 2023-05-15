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
  var triggers = ScriptApp.getProjectTriggers();
  for (let i = 0; i < triggers.length; i++) {
    var trigger = triggers[i];
    if(trigger.getHandlerFunction() == "notifySlack"){
      Browser.msgBox("リマインダーは起動中です");
      return;
    }
  }
  setTrigger();
  Browser.msgBox("リマインダーを起動しました");
}

//次の日の8:30にnotifySlack関数を実行する
function setTrigger(){
  
  //複数のトリガーが設定されてnotifySlackが２回以上呼び出されるのを防ぐために一度notifySlackを削除する
  var triggers = ScriptApp.getProjectTriggers();
  for (let i = 0; i < triggers.length; i++) {
    var trigger = triggers[i];
    if(trigger.getHandlerFunction() == "notifySlack"){
      ScriptApp.deleteTrigger(trigger);
    }
  }
  
  const time = new Date();
  time.setDate(time.getDate()+1);
  time.setHours(8);
  time.setMinutes(30);
  ScriptApp.newTrigger('notifySlack').timeBased().at(time).create();
};

//Slack送信
function notifySlack() {
  
  //次の日の8:30に再びnotifySlackが実行されるようにセット
  setTrigger();
  
  //入稿スケジュール、編集長タスクを取得 
  var info = getInformation();
  
  var submit_schedule = info['submit_schedule'];
  var edit_tasks = info['edit_tasks'];
  
  var text = '';

  //入稿前スケジュールのテキスト
  if(submit_schedule['student'] || submit_schedule['parent']){
    text += '【入稿前スケジュール】\n';
      
    if(submit_schedule['student'] == submit_schedule['parent']){
      text += submit_schedule['student']+'\n';
    }else{
      if(submit_schedule['student']){
        text += '（高校生版）\n'+submit_schedule['student']+'\n';
      };
      if(submit_schedule['parent']){
        text += '（保護者版）\n'+submit_schedule['parent']+'\n';
      }; 
    };
  };
    
  //編集長タスクのテキスト
  if(edit_tasks.length !== 0){
      text += '【編集長タスク】\n';
      edit_tasks.forEach(function(edit_task){
        text += '・'+edit_task+'\n';
      });
  };
  
  //送信
  if(text.length != 0){
    var issue = SpreadsheetApp.getActiveSpreadsheet().getName().replace('フリーペーパー管理シート','');
    
    var payload ={
      'username': '**',
      'text': text
    };
    var options = {
      'method': 'post',
      'contentType': 'application/json',
      'payload': JSON.stringify(payload)
    };
    
    var url = 'webhook url';
    
    UrlFetchApp.fetch(url, options);
  };
};


//タスクを取得する
function getInformation(){
  
  var date = new Date();
  var day = date.getDate();
  var month = date.getMonth();
  var year = date.getFullYear();
  
  var ss = SpreadsheetApp.getActiveSpreadsheet();

  var submit_schedule = getSchedule(day,month,year,ss);
  var edit_tasks = getTasks(day,month,year,ss);
  
  var info = {'submit_schedule':submit_schedule,'edit_tasks':edit_tasks};
  
  return info;
};

//入稿前スケジュールを取得
function getSchedule(day,month,year,ss){

  var sh_submit = ss.getSheetByName('入稿前スケジュール');
  var last_row_sub = sh_submit.getLastRow();
  
  for(var i = 2; i <= last_row_sub; i++){
    var daterow1 = sh_submit.getRange(i,1).getValue();    
    if(Object.prototype.toString.call(daterow1) == '[object Date]'){         
      if(daterow1 != false && daterow1.getDate()==day && daterow1.getMonth()==month && daterow1.getFullYear()==year){
        var schedule_student = sh_submit.getRange(i,4).getValue();
        break;
      }; 
    };    
  };
  
  for(var j = 2; j <= last_row_sub; j++){
    var daterow2 = sh_submit.getRange(j,6).getValue();
    if(Object.prototype.toString.call(daterow2) == '[object Date]'){    
      if(daterow2 != false && daterow2.getDate()==day && daterow2.getMonth()==month && daterow2.getFullYear()==year){
      var schedule_parent = sh_submit.getRange(j,9).getValue();
      break;
      }; 
    };
  };
    
  var schedule = {'student':schedule_student, 'parent':schedule_parent};
  return schedule;
}; 

//編集長タスクを取得
function getTasks(day,month,year,ss){
  
  var sh_edit = ss.getSheetByName('編集長タスク');
  var last_row_edit = sh_edit.getLastRow();
  var last_column_edit = sh_edit.getLastColumn();
  
  var tasks_edit = [];
  
  for(var i = 3; i<= last_column_edit; i++){
    var datecolumn = sh_edit.getRange(2,i).getValue();
    if(Object.prototype.toString.call(datecolumn) == '[object Date]'){
      if(datecolumn.getDate() ==day && datecolumn.getMonth()==month && datecolumn.getFullYear()==year){
        for(var j = 4; j <= last_row_edit; j++){
          if(sh_edit.getRange(j,i).isBlank() == false){
            tasks_edit.push(sh_edit.getRange(j,i).getValue());
          }
        }
      break;
      }
    }
  }; 
  return tasks_edit;
};
