# notify

pomelo中有四种消息类型：request, response, notify, push

一般的web框架，例如express，短连接 两种消息类型request, response，有求必应。
按理说长连接是不需要等待服务器端响应，也就是说客户端client可以使用notify, 不需要注册callback

## lordOfPomelo notify使用

```
//client
$leaveTeam.on('click', function() {
    console.log('click leaveTeam ...');
    pomelo.notify("area.teamHandler.leaveTeam", {
        playerId: pomelo.playerId,
        teamId: pomelo.teamId
    });
    console.log('leaveTeam ~ pomelo.teamId = ', pomelo.teamId);
    $teamMenu.hide();
});


//server
Handler.prototype.leaveTeam = function(msg, session, next) {
  var area = session.area;
  var playerId = session.get('playerId');
  var player = area.getPlayer(playerId);

  if(!player) {
    logger.warn('The request(leaveTeam) is illegal, the player is null: msg = %j.', msg);
    next();
    return;
  }

  utils.myPrint('playerId, IsInTeamInstance = ', playerId, player.isInTeamInstance);
  if (player.isInTeamInstance) {
    next();
    return;
  }

  var result = consts.TEAM.FAILED;

  utils.myPrint("player.teamId = ", player.teamId);
  utils.myPrint("typeof player.teamId = ", typeof player.teamId);

  utils.myPrint("msg.teamId = ", msg.teamId);
  utils.myPrint("typeof msg.teamId = ", typeof msg.teamId);

  if(player.teamId <= consts.TEAM.TEAM_ID_NONE || player.teamId !== msg.teamId) {
    logger.warn('The request(leaveTeam) is illegal, the teamId is wrong: msg = %j.', msg);
    next();
    return;
  }

  var args = {playerId: playerId, teamId: player.teamId};
  this.app.rpc.manager.teamRemote.leaveTeamById(session, args,
    function(err, ret) {
      result = ret.result;
      utils.myPrint("1 ~ result = ", result);
      if(result === consts.TEAM.OK && !player.leaveTeam()) {
        result = consts.TEAM.FAILED;
      }
      if (result === consts.TEAM.OK) {
        var route = 'onTeamMemberStatusChange';
        if(player.isCaptain) {
          route = 'onTeamCaptainStatusChange';
          player.isCaptain = consts.TEAM.NO;
        }
        var ignoreList = {};
        messageService.pushMessageByAOI(area,
          {
            route: route,
            playerId: playerId,
            teamId: player.teamId,
            isCaptain: player.isCaptain,
            teamName: consts.TEAM.DEFAULT_NAME
          },
          {x: player.x, y: player.y}, ignoreList);
      }

      utils.myPrint("teamId = ", player.teamId);
    });

  next();
};

```

需要注意的是，虽然说不需要等待服务器端的响应， 但是服务器一定要next() 只不过不需要返回任何信息给client.

