# Areal2 数据刷新时间修改

## 服务器逻辑

- 定时器刷新

  TaskService.init

  - BeginOfDayTask
    - PClearRoleDayData 结算所有在线玩家的数据
    - PClearSociatyDayData 结算所有工会数据
    - UnlockForbidUp 封印
  - EightClockTask
    - PArenaRankAward 竞技场头像框
    - PSociatyRankAward 工会排行榜头像框
    - PLevelRankAward 等级排行榜头像框
    - PPowerRankAward 战力排行榜头像框
    - PSendArenaDayAwards 竞技场排行榜每日奖励





服务器现有的逻辑



已经修改的逻辑

- 竞技场刷新
- 竞技场奖励
- 每日登录弹窗
- 切地图次数
- 购买金币
- 工会踢人和退出
- 订单刷新
- 初航试炼

额外配置了结算时间

- 宠物喂食