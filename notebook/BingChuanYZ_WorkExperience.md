1. 物品配置表：Leechdom.csv

2. 活动配置表：HuodongSchedule.csv(经典版和远征2分开配置，远征2有_dream)

3. 活动时间配置表：HuodongTime.csv

4. 活动指引配置表Work.csv(经典版和远征2分开配置，远征2有_dream)

5. 邮件发送可以使用 

   ```cpp
   /**
   @name         : 系统发送邮件，邮件中包含多个物品给玩家,可以发送多封邮件
   @param pTopic : 邮件标题
   @param nAddresseeID : 接收邮件人的数据库ID
   @param szContext  : 邮件内容
   @param PrizeInfoList : 奖励物品List  1234;3;61#3423;2;61   奖励ID;个数;绑定标志#奖励ID;个数;绑定标志
   @param pValueItem : 价值串 需要和奖励项个数保持一致
   @return           : 
   */
   bool SendSystemMailSomePrizeWithBindFlag(const char* pTopic,int nAddresseeID,const char* pContext,const char* PrizeInfoList,const char* pValueItem);
   
   /**
   @name				: 系统发送邮件给玩家，可发指定绑定标识物品，注意：发非绑定接物品时要慎重
   @param pTopic		: 邮件标题
   @param nAddresseeID : 接收邮件人的数据库ID
   @param szContext	: 邮件内容
   @param nItemID		: 物品ID
   @param nNum			: 个数
   @param nBindFlags	: 绑定标识
   @param nItemValue	: 物品价值
   @return				: 
   */
   bool SendSystemMailWithBindFlag(const char* pTopic,int nAddresseeID,const char* pContext,int nItemID,int nNum, int nBindFlags, int nItemValue = 0);
   
   /**
   @name				: 系统发送邮件给玩家，可发送金币
   @param pTopic		: 邮件标题
   @param nAddresseeID : PDBID
   @param pAddresseeName : 接收邮件人的名字
   @param szContext	: 邮件内容
   @param nGold		: 金币数
   @param dwGoodsID	; 邮件物品ID
   @param dwGoodsCount	: 邮件物品数量
   @param nType		: 邮件类型
   @param nItemValue	: 物品价值
   @return				: 
   */
   bool SendSystemMailAddGold(const char* pTopic, DWORD nAddresseeID, const char* pAddresseeName, const char* pContext, int nGold, DWORD dwGoodsID, DWORD dwGoodsCount = 1, int nType = emMailType_MakeProfit, int nItemValue = 0);
   
   ```

6. 在下发物品到包裹时要提前判断玩家包裹种是否还有空间：

   ```cpp
   /**
   @name  统计玩家包裹中空格数目
   @brief
   @param bTaskSkep	: true，只统计任务物品，false为不统计任务物品
   @param nSkepType	: 物品栏类型，默认不指定
   */
   int CountEmptyGrid(int nActor, bool bTask, int nSkepType = emSkepTypeUnknown);
   ```

7. 下发物品到包裹：

   ```cpp
   /**
   @name          : 给玩家添加绑定物品,只用于药品
   @brief         : 
   @param nActor  : 玩家ID
   @param nItem   : 物品ID
   @param nNum    : 添加数量,只有药品才能设数量
   @param szReason: 添加原因
   @param bBindFlag:邦定标识
   @param bNewSerial: 记录日志时是否采用新序号
   @param nOssResAddType	: 加物品时可以显式的指定途径类型,便于后台统计
   @param nSkepType	: 物品栏类型，默认不指定
   @return        : 返回放置在包裹中的位置,添加失败返回-1
   */
   int AddBindItem(int nActor,int nItemID,int nNum,const char * szReason,int nBindFlag = tGoods_BindFlag_Hold, bool bNewSerial=true,int nOssResAddType=OssResAdd_TaskCreate, int nSkepType = emSkepTypeUnknown, int nItemValue = 0, int nPassTime =0);
   ```

8. 从玩家包裹扣除道具：

   ```cpp
   /**
   @name          : 从玩家包裹中只移除绑定物品或只移除不绑定物品
   @brief         : 
   @param nActor  : 玩家ID
   @param nPrize  : 物品ID
   @param nNum    : 移除数量
   @param reason  : 移除原因
   @param bNewSerial: 记录日志时是否采用新序号
   @param bSameGroup	: 原因是有了成品装备(物品ID不同，但物品是同一种，只是可改造标志不同而已)以后的后遗症，比如任务需要收集“玄铁盔”,
   　					 现在“玄铁盔”有好几种，每种的ID都不同，那么玩家得到某个“玄铁盔”以后，无法完成任务。
   					 如果为true，表示与nGoodsID的lGoodsGroupID相同即可
   @param nSkepType	: 物品栏类型，默认不指定
   */
   bool RemoveBindItem(int nActor,int nItemID,int nNum,const char * reason , bool bBind, bool bNewSerial=true, bool bSameGroup = false, int nSkepType = emSkepTypeUnknown, int nItemValue = 0);
   ```

9. 家园服连接配置

   ![HomeConfig](D:\HonkerForce.github.io\notebook\BingChuanYZ_WorkExperience\imageset\HomeConfig.png)
